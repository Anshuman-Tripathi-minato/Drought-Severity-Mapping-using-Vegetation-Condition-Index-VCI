# Drought Severity Mapping Using the Vegetation Condition Index (VCI)

This project assesses vegetation condition and drought severity across the
Bundelkhand region of Uttar Pradesh and Madhya Pradesh, India. It combines
MODIS NDVI composites generated in Google Earth Engine with Python-based
processing, VCI calculation, drought classification, spatial analysis, and
CHIRPS rainfall validation.

## Study area and period

- **Study area:** Bundelkhand, represented by 13 districts across Uttar
  Pradesh and Madhya Pradesh
- **Analysis period:** 2015–2023
- **NDVI source:** MODIS/061/MOD13Q1
- **NDVI resolution:** 250 m
- **Administrative boundaries:** FAO GAUL 2015, level 2

## Workflow

1. Build and dissolve the Bundelkhand area of interest (AOI).
2. Load MODIS MOD13Q1 NDVI imagery in Google Earth Engine.
3. Apply the MODIS quality mask and remove invalid fill values.
4. Scale NDVI values and create annual median composites for 2015–2023.
5. Export annual NDVI GeoTIFFs and summary statistics.
6. Stack the annual NDVI rasters in Python and calculate pixel-wise NDVI
   minimum and maximum values.
7. Calculate the Vegetation Condition Index:

   ```text
   VCI = (NDVI - NDVI_min) / (NDVI_max - NDVI_min)
   ```

8. Classify drought severity, calculate area statistics, create maps, and
   compare VCI results with CHIRPS rainfall.

## Drought classification

| VCI range | Class |
| --- | --- |
| `< 0.20` | Extreme drought |
| `0.20–0.40` | Moderate drought |
| `0.40–0.60` | Mild drought |
| `> 0.60` | Healthy vegetation |

## Repository contents

```text
.
├── data/
│   ├── Google_Earth_Engine_code.md    # GEE AOI and MODIS NDVI workflow
│   ├── Bundelkhand_Annual_NDVI_Statistics.csv
│   ├── Bundelkhand_CHIRPS/             # CHIRPS rainfall rasters and statistics
│   ├── NVDI/                           # Annual NDVI rasters
│   └── bundelkhand shape file/         # AOI boundary data
├── notebook/
│   └── Bundelkhand Drought.ipynb       # Python analysis workflow
├── results/
│   ├── Graphs/                         # Trends and VCI–rainfall correlation
│   ├── Maps/                           # NDVI, VCI, and drought maps
│   ├── Raster/                         # GeoTIFF and NumPy raster products
│   ├── Tables/                         # Statistics, validation, and reports
│   └── Vector/                         # Drought zone vector outputs
├── Anshuman_Tripathi_P1.pdf
└── README.md
```

## Running the analysis

1. Open `data/Google_Earth_Engine_code.md` in the
   [Google Earth Engine Code Editor](https://code.earthengine.google.com/).
2. Configure the AOI export destination if an Earth Engine asset export is
   required, then run the script to generate annual NDVI products.
3. Place the exported NDVI rasters in `data/NVDI/`.
4. Open `notebook/Bundelkhand Drought.ipynb` from the repository root and run
   the cells in order. The notebook writes generated products to `results/`.

The notebook uses common scientific Python and geospatial packages, including
NumPy, pandas, Matplotlib, SciPy, scikit-learn, rasterio, GeoPandas, Shapely,
Rasterstats, rioxarray, xarray, pyproj, Fiona, and tqdm. Uncomment the
installation cells at the beginning of the notebook only when these packages
are not already available in the active environment.

## Outputs

The repository includes:

- Annual NDVI, VCI, and drought-classification GeoTIFFs
- NumPy stacks for NDVI, VCI, and drought classes
- Publication-ready NDVI, VCI, and drought maps
- Drought area and pixel-count statistics
- Polygon area statistics
- Trend-analysis data and graphs
- VCI–CHIRPS correlation and raster validation reports

## Limitations and future work

VCI is a vegetation-condition indicator and should be interpreted together
with rainfall, soil, and other field or climate observations. Possible
extensions include SPI–VCI combined analysis, monthly monitoring, improved
time-series change detection, and an interactive web map.

## Author

**Anshuman Tripathi**  
B.Tech Computer Science  
Google Earth Engine · Remote Sensing · GIS · Python
