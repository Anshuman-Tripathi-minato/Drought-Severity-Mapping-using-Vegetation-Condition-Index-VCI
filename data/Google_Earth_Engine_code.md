/***************************************************************
 Project      : Multi-temporal Drought Assessment
 Study Area   : Bundelkhand Region, India
 Dataset      : FAO GAUL 2015 Level-2 Administrative Boundaries
 Purpose      : Create Bundelkhand AOI by merging 13 districts
 Author       : Anshuman Tripathi
***************************************************************/


//==============================================================
// STEP 1 : LOAD FAO GAUL LEVEL-2 DISTRICT BOUNDARIES
//==============================================================

var indiaDistricts = ee.FeatureCollection('FAO/GAUL/2015/level2')
    .filter(ee.Filter.eq('ADM0_NAME', 'India'));

print('India Districts', indiaDistricts.limit(5));


//==============================================================
// STEP 2 : FILTER BUNDELKHAND DISTRICTS
//==============================================================

var upDistricts = indiaDistricts.filter(
    ee.Filter.and(
        ee.Filter.eq('ADM1_NAME', 'Uttar Pradesh'),
        ee.Filter.inList('ADM2_NAME', [
            'Jhansi',
            'Lalitpur',
            'Jalaun',
            'Hamirpur',
            'Mahoba',
            'Banda',
            'Chitrakoot'
        ])
    )
);

var mpDistricts = indiaDistricts.filter(
    ee.Filter.and(
        ee.Filter.eq('ADM1_NAME', 'Madhya Pradesh'),
        ee.Filter.inList('ADM2_NAME', [
            'Datia',
            'Tikamgarh',
            'Chhatarpur',
            'Panna',
            'Damoh',
            'Sagar'
        ])
    )
);


//==============================================================
// STEP 3 : MERGE ALL DISTRICTS
//==============================================================

var bundelkhand = upDistricts.merge(mpDistricts);

print('Bundelkhand Districts', bundelkhand);
print('Number of Districts', bundelkhand.size());


//==============================================================
// STEP 4 : DISSOLVE INTO SINGLE AOI
//==============================================================

var bundelkhandAOI = bundelkhand.union().geometry();

// FIX: Dissolving/unioning 13 district polygons can leave tiny
// topology artifacts (micro-slivers, near-duplicate vertices) along
// the merged boundaries. These are invisible on the map but cause
// "Unable to transform edge ... from SR-ORG:6974 PLANAR ... to
// EPSG:4326" errors later, because MODIS is natively stored in the
// Sinusoidal projection and Earth Engine struggles to reproject a
// noisy boundary into it during .clip(). Simplifying with a small
// error margin (100 m) removes the artifacts without visibly
// changing the district boundaries.
bundelkhandAOI = bundelkhandAOI.simplify({maxError: 100});


//==============================================================
// STEP 5 : VISUALIZATION
//==============================================================

Map.centerObject(bundelkhandAOI, 8);

Map.addLayer(
    bundelkhand.style({
        color: 'blue',
        fillColor: '00000000',
        width: 2
    }),
    {},
    'Bundelkhand Districts'
);

Map.addLayer(
    bundelkhandAOI,
    {
        color: 'red'
    },
    'Bundelkhand AOI'
);


//==============================================================
// STEP 6 : FINAL AOI
//==============================================================

var aoi = bundelkhandAOI;


//==============================================================
// STEP 7 : EXPORT AOI AS ASSET (OPTIONAL - EDIT PATH BEFORE RUNNING)
//==============================================================
//
// FIX: Removed the duplicate export from the original script.
// Only ONE asset export remains. Update 'YOUR_PROJECT_ID' below to
// your actual Cloud Project ID (Cloud-based GEE accounts require
// 'projects/<project-id>/assets/<asset-name>' - the old
// 'users/<username>/...' path only works for legacy accounts).
//
// This step is OPTIONAL for the drought-assessment workflow since
// only the NDVI GeoTIFFs + CSV (Steps 15-16) are needed in Python.
// Uncomment only if you specifically want the AOI saved as an asset.

/*
Export.table.toAsset({
  collection: ee.FeatureCollection([
    ee.Feature(aoi, {
      name: 'Bundelkhand',
      districts: 13,
      source: 'FAO GAUL 2015 Level-2'
    })
  ]),
  description: 'Bundelkhand_AOI',
  assetId: 'projects/YOUR_PROJECT_ID/assets/Bundelkhand_AOI'
});
*/

// Optional: also export AOI to Drive as a shapefile for use in Python/QGIS
Export.table.toDrive({
  collection: bundelkhand,
  description: 'Bundelkhand_AOI_Districts',
  folder: 'Bundelkhand_NDVI',
  fileFormat: 'SHP'
});

//-------------------------------------------------------------------

/***************************************************************
 Project      : Multi-temporal Drought Assessment using MODIS NDVI
 Study Area   : Bundelkhand Region, India
 Dataset      : MODIS/061/MOD13Q1
 Period       : 2015 - 2023
 Resolution   : 250 m
 Purpose      : Generate Annual NDVI Composites for Python Analysis
 Author       : Anshuman Tripathi
***************************************************************/


//=====================================================================
// STEP 8 : LOAD MODIS MOD13Q1 NDVI
//=====================================================================

var modis = ee.ImageCollection("MODIS/061/MOD13Q1")
    .filterBounds(aoi)
    .filterDate('2015-01-01', '2023-12-31');

print('Total MODIS Scenes', modis.size());


//=====================================================================
// STEP 9 : QUALITY ASSURANCE (QA MASK) + FILL-VALUE MASK
//=====================================================================
//
// SummaryQA Values
// 0 = Good
// 1 = Marginal
// 2 = Snow/Ice
// 3 = Cloud
//
// Keep only Good and Marginal pixels.
//
// FIX: Original script only masked on SummaryQA. MOD13Q1 NDVI also
// contains a fill value of -3000 for pixels with no valid data. This
// additional mask prevents fill values from contaminating the
// NDVI min/mean/stdDev statistics later on.
//

function maskQA(image) {

    var qa = image.select('SummaryQA');
    var qaMask = qa.lte(1);

    var ndviRaw = image.select('NDVI');
    var validRangeMask = ndviRaw.gte(-2000).and(ndviRaw.lte(10000));

    return image.updateMask(qaMask).updateMask(validRangeMask);

}

var qaCollection = modis.map(maskQA);


//=====================================================================
// STEP 10 : SCALE NDVI
//=====================================================================
//
// MODIS Scale Factor = 0.0001
//

function scaleNDVI(image) {

    // FIX (round 2): Previously this clipped EVERY raw MODIS scene
    // (207 clips total) to the AOI. MODIS's native Sinusoidal
    // projection (SR-ORG:6974) occasionally fails to transform the
    // AOI edge into a specific scene's tile grid, throwing
    // "Unable to transform edge ... from SR-ORG:6974 PLANAR ... to
    // EPSG:4326" - and which scene fails is unpredictable (it moved
    // from 2017_05_25 to 2019_04_23 between runs). Clipping 207
    // times just multiplies the odds of hitting this.
    //
    // Fix: don't clip here at all. Only clip the 9 ANNUAL COMPOSITES
    // later, after they've been reprojected to EPSG:4326 - so there's
    // no cross-CRS edge transform left to fail on.

    return ee.Image(
        image
            .select('NDVI')
            .multiply(0.0001)
            .rename('NDVI')
            .copyProperties(image, ['system:time_start'])
    );

}

var ndviCollection = qaCollection.map(scaleNDVI);


//=====================================================================
// STEP 11 : CREATE ANNUAL NDVI COMPOSITES
//=====================================================================

var years = ee.List.sequence(2015, 2023);

var annualNDVI = ee.ImageCollection.fromImages(

    years.map(function(year) {

        year = ee.Number(year);

        var start = ee.Date.fromYMD(year,1,1);
        var end = start.advance(1,'year');

        // FIX: reproject to EPSG:4326 BEFORE clipping. Reprojecting
        // first means the composite and the AOI polygon are already
        // in the same CRS when .clip() runs, so there's no risky
        // Sinusoidal -> EPSG:4326 edge transform happening inside
        // clip() itself. This also guarantees all 9 annual GeoTIFFs
        // share an identical, aligned pixel grid - useful later when
        // stacking rasters in Python for NDVI min/max/VCI.
        var image = ndviCollection
            .filterDate(start,end)
            .median()
            .reproject({crs: 'EPSG:4326', scale: 250})
            .clip(aoi)
            .set('Year', year)
            .set('system:time_start', start.millis());

        return image;

    })

);

print('Annual NDVI Collection', annualNDVI);


//=====================================================================
// STEP 12 : NDVI VISUALIZATION
//=====================================================================

var ndviVis = {
    min:0,
    max:1,
    palette:[
        '#8c510a',
        '#d8b365',
        '#f6e8c3',
        '#c7eae5',
        '#5ab4ac',
        '#01665e'
    ]
};

years.getInfo().forEach(function(year){

    var image = annualNDVI
        .filter(ee.Filter.eq('Year',year))
        .first();

    Map.addLayer(
        image,
        ndviVis,
        'NDVI ' + year,
        false
    );

});


//=====================================================================
// STEP 13 : CALCULATE ANNUAL NDVI STATISTICS
//=====================================================================

var reducer = ee.Reducer.mean()
    .combine({
        reducer2: ee.Reducer.min(),
        sharedInputs:true
    })
    .combine({
        reducer2: ee.Reducer.max(),
        sharedInputs:true
    })
    .combine({
        reducer2: ee.Reducer.stdDev(),
        sharedInputs:true
    });

var annualStats = ee.FeatureCollection(

    years.map(function(year){

        year = ee.Number(year);

        var image = annualNDVI
            .filter(ee.Filter.eq('Year',year))
            .first();

        var imageCount = ndviCollection
            .filterDate(
                ee.Date.fromYMD(year,1,1),
                ee.Date.fromYMD(year.add(1),1,1)
            )
            .size();

        var stats = image.reduceRegion({
            reducer: reducer,
            geometry: aoi,
            scale:250,
            maxPixels:1e13
        });

        return ee.Feature(null,{
            Year:year,
            Image_Count:imageCount,
            Mean_NDVI:stats.get('NDVI_mean'),
            Min_NDVI:stats.get('NDVI_min'),
            Max_NDVI:stats.get('NDVI_max'),
            StdDev_NDVI:stats.get('NDVI_stdDev')
        });

    })

);

print('Annual NDVI Statistics', annualStats);


//=====================================================================
// STEP 14 : EXPORT ANNUAL NDVI IMAGES
//=====================================================================

years.getInfo().forEach(function(year){

    var image = annualNDVI
        .filter(ee.Filter.eq('Year',year))
        .first();

    Export.image.toDrive({
        image:image,
        description:'NDVI_' + year,
        folder:'Bundelkhand_NDVI',
        fileNamePrefix:'NDVI_' + year,
        region:aoi,
        scale:250,
        crs:'EPSG:4326',
        maxPixels:1e13
    });

});


//=====================================================================
// STEP 15 : EXPORT ANNUAL STATISTICS
//=====================================================================

Export.table.toDrive({
    collection:annualStats,
    description:'Bundelkhand_Annual_NDVI_Statistics',
    folder:'Bundelkhand_NDVI',
    fileFormat:'CSV'
});


//=====================================================================
// STEP 16 : PROCESS COMPLETED
//=====================================================================

print('------------------------------------------------------------');
print(' MODIS NDVI PROCESSING COMPLETED ');
print(' Study Area : Bundelkhand');
print(' Period     : 2015-2023');
print(' Dataset    : MODIS/061/MOD13Q1');
print('------------------------------------------------------------');


/***************************************************************
 CHANGE LOG (vs. original script)
 1. Removed duplicate/legacy Export.table.toAsset call (old script
    had it twice - once commented, once active with a hardcoded
    legacy 'users/...' asset path). Kept a single, clearly-labeled
    OPTIONAL asset export with a placeholder for your actual
    Cloud Project ID, plus an added Drive/SHP export as a more
    portable alternative.
 2. Added a fill-value mask (-3000 to -2000/10000 valid range) in
    the QA masking step, in addition to the existing SummaryQA
    mask, so invalid pixels can't skew NDVI min/mean/stdDev stats.
 3. Removed the redundant duplicate 'Number of Districts' print
    statement.
 4. Renumbered steps sequentially after removing the duplicate
    export block.
 5. Fixed a runtime error in scaleNDVI(): copyProperties() returns a
    generic Object, so calling .clip() after it threw
    "image....clip is not a function". Reordered to clip() before
    copyProperties() and wrapped the result in ee.Image(...) to
    restore the correct type.
 6. Added aoi = aoi.simplify({maxError: 100}) right after the
    union/dissolve step. Without this, clipping MODIS images
    (native Sinusoidal projection) to the dissolved AOI threw
    "Unable to transform edge ... from SR-ORG:6974 PLANAR ... to
    EPSG:4326" due to micro-topology artifacts left over from the
    district union operation.
 7. That clip error persisted intermittently (a different scene
    date failed on each run) because scaleNDVI() clipped EVERY raw
    MODIS scene (207 clips). Moved .clip(aoi) out of scaleNDVI()
    entirely; now only the 9 ANNUAL COMPOSITES are clipped, and
    only after being explicitly .reproject()-ed to EPSG:4326 first,
    so clip() never has to transform an edge across CRSs. Also
    guarantees all 9 annual GeoTIFFs share an identical pixel grid.
***************************************************************/