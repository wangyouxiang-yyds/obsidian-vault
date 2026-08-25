# 決策：Prithvi + DeepLabV3(ASPP) head decoder 消融 arm

- 日期：2026-07-15
- 決策者：使用者拍板；Claude×Codex 協商兩輪收斂（R1 REVISE 8 blocking → R2 ACCEPT, BLOCKING=[]）
- 背景：Prithvi-300M+UPerNet+Tversky 已 3-fold 雙閘門全過（pooled mean 0.4498 vs tversky 0.3915，355-scene Wilcoxon p=3.9e-10），但該比較同時換了 encoder+decoder。本 arm 把 head 換回 torchvision DeepLabV3(ASPP)，拆開 encoder/decoder 貢獻，並支持計畫報告「DeepLabV3 架構」延續性主張。

## 定稿設計（預註冊，不得看結果後改）
1. encoder 建構/C6 嚴格載入與 UPerNet arm 共用同一 helper（`deeplab_adapter._prithvi_build_encoder()`，純搬移重構，UPerNet 回歸測試 10/10 PASS）
2. head=整包 torchvision DeepLabHead（ASPP+projection+dropout 0.5+classifier），只參數化 in_channels 與 rates，不自組
3. ASPP rates=(6,12,18)（DeepLab 論文 OS=16 標準）；rate 18 在 16×16 grid 已知幾何退化，仍照論文標準預註冊；(3,6,9) 等 grid 適配版不做，除非另立 sensitivity arm
4. tap=[23]＝第 24 block 經 encoder terminal norm 的最終特徵（程式 assert；測試驗證與 UPerNet 版最後層特徵逐位一致）
5. 無 aux classifier/aux loss
6. logits 上採樣 bilinear align_corners=False、明確 size=(256,256)
7. LR 分組：encoder（含 terminal norm）=0.1×lr，DeepLabHead 全部=1.0×lr，exhaustive/disjoint assertion
8. drop_last 維持 False＋既有 collate 守門（min_physical_batch_size=2）——Codex 原提 drop_last=True，經澄清「與 comparator 協定凍結」後接受 Claude 方案；skipped-microbatch 計數已有 log（觀測性要求）

## Pilot 預註冊 gate
- fold1 only；主閘門 comparator=tversky fold1 pooled_oil_iou 0.4376；GO=pooled≥0.4676（Δ≥+0.030）且 per-scene one-sided Wilcoxon(>) p<0.05 且無 protocol 異常
- 次要參考=Prithvi+UPerNet fold1 0.5352（消融解讀用，不當 gate）
- 結論措辭限定「Prithvi 條件下的 decoder-family contrast + encoder bridge contrast」；無 ResNet50+UPerNet 的完整 2×2，不得做 encoder/decoder 貢獻百分比分解

## 落地
- 新檔：`main/prithvi_deeplab.py`、`main/test_prithvi_dlv3_shapes.py`（8/8 PASS）、`main/experiments_CV_358clean_gt_expand_prithvi300m_dlv3_fold1_pilot.yaml`
- 修改：`main/deeplab_adapter.py`（dispatch 加 `model_family: prithvi_eo_v2_dlv3`、encoder 建構抽共用 helper、min-batch 守門與 LR 分組擴充）
- GPU preflight：batch16 AMP 峰值 8.08/31.3 GB
- 訓練啟動：2026-07-15 11:56 UTC，log=`train_log/nohup_prithvi300m_dlv3_pilot_20260715-115605.log`
