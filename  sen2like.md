# File Tree: sen2like

**Generated:** 5/4/2026, 10:15:54 AM
**Root Path:** `/home/alanyh/oil_dataset/sen2like`

```
├── .claude
│   ├── settings.json
│   └── settings.local.json
├── .github
│   └── workflows
│       ├── lint.yml
│       └── test.yml
├── prisma4sen2like
│   ├── prisma
│   │   ├── product_template
│   │   │   ├── DATASTRIP
│   │   │   │   └── DS_ID
│   │   │   │       └── MTD_DS.xml
│   │   │   ├── GRANULE
│   │   │   │   └── TL_ID
│   │   │   │       └── MTD_TL.xml
│   │   │   ├── rep_info
│   │   │   │   └── S2_User_Product_Level-1C_Metadata.xsd
│   │   │   └── MTD_MSIL1C.xml
│   │   ├── sen2like
│   │   │   ├── __init__.py
│   │   │   ├── grids.py
│   │   │   ├── image_file.py
│   │   │   └── s2tiles.db
│   │   ├── adapter.py
│   │   ├── geometry.py
│   │   ├── log.py
│   │   ├── main.py
│   │   ├── mgrs_util.py
│   │   ├── prisma_product.py
│   │   ├── prisma_s2_spectral_aggregation.py
│   │   ├── product_builder.py
│   │   ├── sen2like_product.py
│   │   ├── spectral_aggregation_functions.py
│   │   ├── utils.py
│   │   └── version.py
│   ├── .gitignore
│   ├── .pre-commit-config.yaml
│   ├── .pylintrc
│   ├── LICENSE.txt
│   ├── README.md
│   ├── apache-2.tmpl
│   ├── environment.yml
│   └── requirements_dev.txt
├── sen2cor3
│   └── README.md
├── sen2like
│   ├── aux_data
│   │   └── dem
│   │       └── dem_downloader.py
│   ├── conf
│   │   ├── Sen2Like_GIPP.xsd
│   │   ├── config.ini
│   │   ├── config.xml
│   │   └── tile_mgrs_31TFJ.json
│   ├── docs
│   │   ├── resources
│   │   │   └── miniconda_version.png
│   │   └── source
│   │       ├── OMPC.TPZ.S2L.PFS.001 - i1r3 Level 2HF Product Format Specification.pdf
│   │       ├── S2-SEN2LIKE-UM-V1.10.pdf
│   │       ├── changelog.rst
│   │       ├── conf.py
│   │       ├── index.rst
│   │       └── readme.rst
│   ├── sen2like
│   │   ├── atmcor
│   │   │   ├── smac
│   │   │   │   ├── COEFS
│   │   │   │   │   ├── Coef_LANDSAT8_1370_1.dat
│   │   │   │   │   ├── Coef_LANDSAT8_1630_1.dat
│   │   │   │   │   ├── Coef_LANDSAT8_2250_1.dat
│   │   │   │   │   ├── Coef_LANDSAT8_440_1.dat
│   │   │   │   │   ├── Coef_LANDSAT8_490_1.dat
│   │   │   │   │   ├── Coef_LANDSAT8_560_1.dat
│   │   │   │   │   ├── Coef_LANDSAT8_660_1.dat
│   │   │   │   │   ├── Coef_LANDSAT8_860_1.dat
│   │   │   │   │   ├── Coef_LANDSAT8_PAN_1.dat
│   │   │   │   │   ├── Coef_S2A_CONT_B1.dat
│   │   │   │   │   ├── Coef_S2A_CONT_B10.dat
│   │   │   │   │   ├── Coef_S2A_CONT_B11.dat
│   │   │   │   │   ├── Coef_S2A_CONT_B12.dat
│   │   │   │   │   ├── Coef_S2A_CONT_B2.dat
│   │   │   │   │   ├── Coef_S2A_CONT_B3.dat
│   │   │   │   │   ├── Coef_S2A_CONT_B4.dat
│   │   │   │   │   ├── Coef_S2A_CONT_B5.dat
│   │   │   │   │   ├── Coef_S2A_CONT_B6.dat
│   │   │   │   │   ├── Coef_S2A_CONT_B7.dat
│   │   │   │   │   ├── Coef_S2A_CONT_B8.dat
│   │   │   │   │   ├── Coef_S2A_CONT_B8a.dat
│   │   │   │   │   ├── Coef_S2A_CONT_B9.dat
│   │   │   │   │   ├── Coef_S2B_CONT_B1.dat
│   │   │   │   │   ├── Coef_S2B_CONT_B10.dat
│   │   │   │   │   ├── Coef_S2B_CONT_B11.dat
│   │   │   │   │   ├── Coef_S2B_CONT_B12.dat
│   │   │   │   │   ├── Coef_S2B_CONT_B2.dat
│   │   │   │   │   ├── Coef_S2B_CONT_B3.dat
│   │   │   │   │   ├── Coef_S2B_CONT_B4.dat
│   │   │   │   │   ├── Coef_S2B_CONT_B5.dat
│   │   │   │   │   ├── Coef_S2B_CONT_B6.dat
│   │   │   │   │   ├── Coef_S2B_CONT_B7.dat
│   │   │   │   │   ├── Coef_S2B_CONT_B8.dat
│   │   │   │   │   ├── Coef_S2B_CONT_B8a.dat
│   │   │   │   │   ├── Coef_S2B_CONT_B9.dat
│   │   │   │   │   └── Coef_S2B_allbands.dat
│   │   │   │   ├── __init__.py
│   │   │   │   ├── copyright.txt
│   │   │   │   ├── readme.md
│   │   │   │   └── smac.py
│   │   │   ├── __init__.py
│   │   │   ├── atmospheric_parameters.py
│   │   │   ├── cams_data_reader.py
│   │   │   └── get_s2_angles.py
│   │   ├── core
│   │   │   ├── QI_MTD
│   │   │   │   ├── xml_backbones
│   │   │   │   │   ├── L2F_QUALITY_backbone.xml
│   │   │   │   │   ├── L2H_QUALITY_backbone.xml
│   │   │   │   │   ├── MTD_MSIL2F_S2.xml
│   │   │   │   │   ├── MTD_MSIL2H_S2.xml
│   │   │   │   │   ├── MTD_OLIL2F_L8.xml
│   │   │   │   │   ├── MTD_OLIL2H_L8.xml
│   │   │   │   │   ├── MTD_TL_L2F_L8.xml
│   │   │   │   │   ├── MTD_TL_L2F_S2.xml
│   │   │   │   │   ├── MTD_TL_L2H_L8.xml
│   │   │   │   │   ├── MTD_TL_L2H_S2.xml
│   │   │   │   │   └── S2_folder_backbone.xml
│   │   │   │   ├── xsd_files
│   │   │   │   │   ├── L2F_QUALITY.xsd
│   │   │   │   │   └── L2H_QUALITY.xsd
│   │   │   │   ├── QIreport.py
│   │   │   │   ├── S2_structure.py
│   │   │   │   ├── __init__.py
│   │   │   │   ├── generic_writer.py
│   │   │   │   ├── mtd.py
│   │   │   │   ├── mtd_writers.py
│   │   │   │   └── stac_interface.py
│   │   │   ├── file_extractor
│   │   │   │   ├── __init__.py
│   │   │   │   ├── file_extractor.py
│   │   │   │   └── landsat_utils.py
│   │   │   ├── product_archive
│   │   │   │   ├── data
│   │   │   │   │   ├── l8tiles.db
│   │   │   │   │   └── s2tiles.db
│   │   │   │   ├── __init__.py
│   │   │   │   ├── product_archive.py
│   │   │   │   ├── product_selector.py
│   │   │   │   └── tile_db.py
│   │   │   ├── products
│   │   │   │   ├── landsat_8
│   │   │   │   │   ├── __init__.py
│   │   │   │   │   ├── bands.csv
│   │   │   │   │   └── landsat8.py
│   │   │   │   ├── landsat_8_maja
│   │   │   │   │   ├── __init__.py
│   │   │   │   │   ├── bands.csv
│   │   │   │   │   └── landsat8_maja.py
│   │   │   │   ├── sentinel_2
│   │   │   │   │   ├── __init__.py
│   │   │   │   │   ├── bands.csv
│   │   │   │   │   └── sentinel2.py
│   │   │   │   ├── sentinel_2_maja
│   │   │   │   │   ├── __init__.py
│   │   │   │   │   ├── bands.csv
│   │   │   │   │   └── sentinel2_maja.py
│   │   │   │   ├── __init__.py
│   │   │   │   ├── hls_product.py
│   │   │   │   └── product.py
│   │   │   ├── readers
│   │   │   │   ├── data
│   │   │   │   │   └── l8_descending
│   │   │   │   │       ├── WRS2_descending.cpg
│   │   │   │   │       ├── WRS2_descending.dbf
│   │   │   │   │       ├── WRS2_descending.prj
│   │   │   │   │       ├── WRS2_descending.sbn
│   │   │   │   │       ├── WRS2_descending.sbx
│   │   │   │   │       ├── WRS2_descending.shp
│   │   │   │   │       ├── WRS2_descending.shx
│   │   │   │   │       └── WRS2_descending.xml
│   │   │   │   ├── __init__.py
│   │   │   │   ├── landsat.py
│   │   │   │   ├── landsat_maja.py
│   │   │   │   ├── maja_reader.py
│   │   │   │   ├── reader.py
│   │   │   │   ├── sentinel2.py
│   │   │   │   └── sentinel2_maja.py
│   │   │   ├── sen2cor_client
│   │   │   │   ├── L2A_GIPP_ROI_Landsat_template.xml
│   │   │   │   ├── __init__.py
│   │   │   │   └── sen2cor_client.py
│   │   │   ├── S2L_config.py
│   │   │   ├── S2L_tools.py
│   │   │   ├── __init__.py
│   │   │   ├── argparser.py
│   │   │   ├── dem.py
│   │   │   ├── image_file.py
│   │   │   ├── log.py
│   │   │   ├── mask_util.py
│   │   │   ├── metadata_extraction.py
│   │   │   ├── module_loader.py
│   │   │   ├── product_preparation.py
│   │   │   ├── product_process.py
│   │   │   ├── reference_image.py
│   │   │   └── toa_reflectance.py
│   │   ├── grids
│   │   │   ├── Reame.md
│   │   │   ├── __init__.py
│   │   │   ├── grids.py
│   │   │   ├── kml2s2tiles.py
│   │   │   ├── mgrs_framing.py
│   │   │   ├── utm_zone_wrs2.csv
│   │   │   ├── utm_zone_wrs2.lst
│   │   │   └── wrskml2l8tiles.py
│   │   ├── klt
│   │   │   ├── __init__.py
│   │   │   └── klt.py
│   │   ├── s2l_processes
│   │   │   ├── S2L_Atmcor.py
│   │   │   ├── S2L_Fusion.py
│   │   │   ├── S2L_Geometry.py
│   │   │   ├── S2L_GeometryCheck.py
│   │   │   ├── S2L_InterCalibration.py
│   │   │   ├── S2L_Nbar.py
│   │   │   ├── S2L_PackagerL2F.py
│   │   │   ├── S2L_PackagerL2H.py
│   │   │   ├── S2L_Process.py
│   │   │   ├── S2L_Product_Packager.py
│   │   │   ├── S2L_Sbaf.py
│   │   │   ├── S2L_Stitching.py
│   │   │   ├── S2L_Toa.py
│   │   │   ├── S2L_TopographicCorrection.py
│   │   │   └── __init__.py
│   │   ├── __init__.py
│   │   ├── generate_stac_files.py
│   │   ├── sen2like.py
│   │   └── version.py
│   ├── tests
│   │   ├── aux_data
│   │   │   └── _dem_downloader.py
│   │   ├── core
│   │   │   ├── downloader
│   │   │   │   ├── __init__.py
│   │   │   │   ├── config.ini
│   │   │   │   └── test_product_downloader.py
│   │   │   ├── file_extractor
│   │   │   │   ├── ref_data
│   │   │   │   │   ├── test_landsat
│   │   │   │   │   │   ├── nodata_pixel_mask.tif
│   │   │   │   │   │   └── test_landsat.tif
│   │   │   │   │   ├── test_landsat_maja
│   │   │   │   │   │   ├── nodata_pixel_mask.tif
│   │   │   │   │   │   └── test_landsat_maja.tif
│   │   │   │   │   ├── test_s2_l1c
│   │   │   │   │   │   ├── nodata_pixel_mask_B01.tif
│   │   │   │   │   │   └── test_s2_l1c.tif
│   │   │   │   │   ├── test_s2_l2a
│   │   │   │   │   │   ├── nodata_pixel_mask.tif
│   │   │   │   │   │   └── test_s2_l2a.tif
│   │   │   │   │   └── test_sentinel2_maja
│   │   │   │   │       ├── nodata_pixel_mask.tif
│   │   │   │   │       └── test_sentinel2_maja.tif
│   │   │   │   ├── Arles-communes-13-bouches-du-rhone.geojson
│   │   │   │   ├── Avignon-communes-84-vaucluse.geojson
│   │   │   │   ├── abstract_extractor_test.py
│   │   │   │   ├── config.ini
│   │   │   │   ├── test.geojson
│   │   │   │   ├── test_landsat.py
│   │   │   │   ├── test_landsat_maja.py
│   │   │   │   ├── test_sentinel2.py
│   │   │   │   └── test_sentinel2_maja.py
│   │   │   ├── product_archive
│   │   │   │   ├── Avignon-communes-84-vaucluse.geojson
│   │   │   │   ├── on_rome.geojson
│   │   │   │   └── test_tile_db.py
│   │   │   ├── products
│   │   │   │   ├── landsat_8
│   │   │   │   │   ├── __init__.py
│   │   │   │   │   └── test_landsat8.py
│   │   │   │   ├── __init__.py
│   │   │   │   ├── config.ini
│   │   │   │   └── test_processing_context.py
│   │   │   ├── __init__.py
│   │   │   ├── config.ini
│   │   │   ├── sen2cor_L2A_GIPP_DEM_LS.xml
│   │   │   ├── sen2cor_L2A_GIPP_DEM_OFF.xml
│   │   │   ├── sen2cor_L2A_GIPP_DEM_ON.xml
│   │   │   └── test_sen2cor_client.py
│   │   ├── s2l_processes
│   │   │   ├── nbar_aux_data
│   │   │   │   ├── S2L_UV_AUX_HABA_101_20221027T114424_V20190105T103429_20191231T103339_T31TFJ_MLSS2_MO.nc
│   │   │   │   ├── S2__USER_AUX_HABA___UV___20221027T000101_V20170105T103429_20171231T103339_T31TFJ_MLSS2_MO.nc
│   │   │   │   ├── S2__USER_AUX_HABA___UV___20221027T000101_V20180105T103429_20181231T103339_T31TFJ_MLSS2_MO.nc
│   │   │   │   └── S2__USER_AUX_HABA___UV___20221028T000101_V20170105T103429_20171231T103339_T31TFJ_MLSS2_MO.nc
│   │   │   ├── config.ini
│   │   │   ├── test_S2L_InterCalibration.py
│   │   │   ├── test_S2L_Nbar.py
│   │   │   └── test_s2l_processes.py
│   │   ├── __init__.py
│   │   ├── run_tests.py
│   │   └── test_sen2like.py
│   ├── .dockerignore
│   ├── .gitignore
│   ├── Dockerfile
│   ├── Dockerfile-base
│   ├── Dockerfile-tests
│   ├── LICENSE.txt
│   ├── MANIFEST.in
│   ├── README.md
│   ├── environment.yml
│   ├── generate-doc.sh
│   ├── release-notes.md
│   ├── requirements_dev.txt
│   ├── requirements_tests.txt
│   ├── setup.py
│   ├── stac.md
│   ├── stac_requests.py
│   └── window_conf.txt
├── Dockerfile.gemini
├── Dockerfile.github
├── GEMINI.md
├── README.md
├── apt-packages.txt
├── batch_process.py
├── compare_spectral_diff.py
├── create_comparison_rgb.py
├── create_full_montage.py
├── create_side_by_side.py
├── docker_run.sh
├── howto.md
├── organize_and_clean_s2.py
├── pip-packages.txt
├── run_all_landsat.sh
└── run_pipeline.py
```

---
*Generated by FileTree Pro Extension*