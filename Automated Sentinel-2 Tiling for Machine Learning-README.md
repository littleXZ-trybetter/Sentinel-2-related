# Automated Sentinel-2 Tiling for Machine Learning

This project automates the generation of 5 km × 5 km tiles from Sentinel-2 imagery, exporting them to Google Earth Engine (GEE) assets and optionally Google Drive, for downstream machine-learning processing. Sentinel-2 is a high-resolution multispectral satellite mission (13 bands, 10–60 m resolution). The workflow runs in Google Colab using the Earth Engine Python API and Google Drive, clipping a large Sentinel-2 image into uniform tiles, exporting valid tiles as GeoTIFFs, and packaging them as NetCDF files.

## Environment Requirements

* **Google Account & Earth Engine:** A Google account with Earth Engine access. You must [register for Earth Engine](https://signup.earthengine.google.com/) and enable the Earth Engine API.
* **Google Colab:** The code runs in Colab (Python 3.x) with `earthengine-api` and other libraries installed (`geemap`, `xarray`, etc.).
* **Google Drive:** If you choose to download tiles, enable Google Drive for the Colab runtime. Mounting Drive allows saving files to your Drive space.
* **GEE Asset Folder:** A destination asset folder (e.g. `projects/<your-project>/assets/tiles/`) with write permissions for your Google Cloud project. You may create this via the [Earth Engine Asset Manager](https://code.earthengine.google.com/assets) or the `earthengine` CLI.

## Setup Instructions

1. **Authenticate with Earth Engine:** In Colab, install the Earth Engine API and authenticate. For example:

   ```python
   !pip install -q earthengine-api
   import ee
   ee.Authenticate()                 # Trigger Colab auth flow
   ee.Initialize(project='my-project')
   ```

   This follows the official EE guide (run `ee.Authenticate()` then `ee.Initialize()` with your GCP project).
2. **Mount Google Drive (optional):** To save downloads, mount Drive into Colab:

   ```python
   from google.colab import drive
   drive.mount('/content/drive')    # See SO example for Colab Drive mount:contentReference[oaicite:3]{index=3}
   ```

   This prompts for permission and mounts your Drive at `/content/drive/`.
3. **Set Parameters:** Define your region of interest (ROI) and parameters (tile size, scale, etc.). For example, use an `ee.Geometry.Polygon` or bounding box, and set tile size to 5000 m. Ensure your Earth Engine **assetId** folder exists and you have write access.

## Script Structure

The Colab notebook is organized into the following sections:

* **Initialization:** Import libraries (`ee`, optionally `geemap`), authenticate (as above), and import any utility functions.
* **ROI and Image Selection:** Define the ROI geometry and load the desired Sentinel-2 image (`ee.Image` or composite) over that ROI. For example, select a Sentinel-2 L2A image collection (`COPERNICUS/S2_SR`) and filter by date and bounds.
* **Tiling:** Build a grid of 5 km×5 km tiles covering the ROI. This can be done with `ee.Geometry.Polygon` (or `Geometry.coveringGrid`) at a 5000 m scale, producing a `FeatureCollection` of tile polygons. Filter or skip tiles that contain little to no data (e.g. cloud mask or empty).
* **Export Loop:** For each tile:

  * **To GEE Asset:** Use `ee.batch.Export.image.toAsset()` with the tile geometry and desired scale. Example in Python:

    ```python
    task = ee.batch.Export.image.toAsset(
        image=tile_image, 
        description='tile1', 
        assetId='projects/my-project/assets/tiles/tile1',
        region=tile_geom, 
        scale=10
    )
    task.start()
    ```

    This creates a batch task to export the tile to your Earth Engine assets. The example above follows the Earth Engine Export API.
  * **To Google Drive (optional):** Similarly, `ee.batch.Export.image.toDrive()` can send the GeoTIFF to your Drive:

    ```python
    task = ee.batch.Export.image.toDrive(
        image=tile_image, 
        description='tile1-drive', 
        folder='gee_tiles', 
        region=tile_geom, 
        scale=10
    )
    task.start()
    ```

    This uses the Colab Python API example for exporting to Drive.
* **Download Tiles:** If tiles are exported to Drive, they appear under `/content/drive/My Drive/gee_tiles/`. You can copy or move them as needed for local processing.

## How to Run

1. **Open the Colab Notebook:** Load the provided Colab notebook or code script.
2. **Run Initialization Cells:** Execute the cells that install packages, mount Drive, and authenticate Earth Engine. Ensure no errors.
3. **Configure Parameters:** Set variables for ROI coordinates, date range, asset folder, etc., at the top of the script.
4. **Run Processing Cells:** Execute cells in order:

   * Define ROI and load images.
   * Generate tile grid.
   * Iterate over tiles to launch export tasks.
   * Monitor progress. Each `task.start()` call enqueues a task in the Earth Engine **Tasks** panel (the code will print status).
5. **Check Exports:** In the Earth Engine [Tasks tab](https://code.earthengine.google.com/tasks), review and run any pending tasks if needed. Completed tiles will appear in your GEE Assets or Drive.
6. **Download (if using Drive):** Tiles exported to Drive can be accessed via the mounted path (e.g. `/content/drive/My Drive/gee_tiles/`). You may also use `ee.data.getAsset()` or `earthengine` CLI to retrieve assets.

## Optional Postprocessing (NetCDF)

After exporting GeoTIFF tiles, you can combine bands and save them as NetCDF for ML workflows. For example, using `xarray` in Python:

```python
import xarray as xr
# Open the multi-band TIFF (bands x, y, width, height)
data = xr.open_rasterio('/content/drive/My Drive/gee_tiles/tile1.tif')
# Optionally set meaningful names/coords, e.g. band names or wavelengths
# Save to NetCDF
data.to_netcdf('tile1.nc')
```

The `DataArray.to_netcdf()` method writes out the data and coordinates to a netCDF file. Ensure that each tile has consistent band ordering (e.g. Sentinel-2 bands B02, B03, …, B12) and include metadata as needed (CRS, geotransform, etc.) so downstream ML code can interpret the file.

## Troubleshooting Tips

* **Authentication Errors:** If `ee.Initialize()` fails, rerun `ee.Authenticate(force=True)` to refresh credentials. Ensure you selected the correct Google account during the OAuth flow.
* **Mount Issues:** If `drive.mount()` does nothing, check the Colab UI for a “Mount Drive” prompt (top-left file browser). Try restarting the runtime and running that cell again.
* **Asset Permissions:** A “permission denied” when exporting to an asset means your GCP project or asset folder is not writable. Verify that you initialized EE with the project that owns the asset and that your Earth Engine account has write access to that asset folder.
* **Export Failures (Max Pixels):** Large export tasks can hit the default 1e8 pixel limit and error out. To fix this, explicitly set `maxPixels` to a higher value in the export call (e.g. `maxPixels=1e12`). You may also reduce tile size or increase sampling resolution (by lowering `scale`) to stay within limits.
* **Slow Task Queue:** Earth Engine may throttle large numbers of tasks. If many exports are started simultaneously, some will pause in **Tasks**. Stagger task launches or monitor the EE Tasks panel to rerun stalled tasks.

## License and Sharing Notice

**Internal Use Only:** This code and data are for internal use within the organization. Do not redistribute or publish externally. The notebook and generated data are **not** licensed for public use.

**Acknowledgments:** Built on Google Earth Engine infrastructure. Refer to Google’s documentation for Earth Engine and Colab for detailed API usage.
