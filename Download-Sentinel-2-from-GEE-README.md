# Download-Sentinel-2-from-GEE
# Sentinel-2 Image Collection for Custom Region in 2024

## Overview

This repository contains a script designed to process Sentinel-2 L2A imagery for a custom rectangular region in 2024. The script filters Sentinel-2 data based on date and cloud cover, creates a grid of tiles, and exports each tile's image to Google Drive.

The script is intended for use in the Google Earth Engine (GEE) platform and processes images in batches to avoid exceeding the Google Drive limits.

## Features

- Custom region defined by a rectangular bounding box.
- Filters Sentinel-2 L2A data for the year 2024 with cloud coverage less than 10%.
- Creates a grid of 1-degree tiles for the region.
- Exports each tile's median image to Google Drive as separate files.
- Image export includes a name based on the tile's bounding coordinates.

## Requirements

- Google Earth Engine account: [Sign up here](https://signup.earthengine.google.com/)
- Google Drive for export
- Internet connection for processing and exporting data

## Instructions

1. **Open Google Earth Engine Code Editor**: Visit [Google Earth Engine Code Editor](https://code.earthengine.google.com/).
2. **Load the Script**: Copy the script from this repository and paste it into the code editor.
3. **Run the Script**: Adjust the `startIndex` and `batchSize` parameters to process your desired number of tiles. Then, run the script.
4. **Check Google Drive**: Once the export completes, the tiles will appear in the `Sentinel2_CustomRegion` folder in your Google Drive.

## Export Parameters

- **Tile Size**: 1 degree by 1 degree grid.
- **Resolution**: 10 meters (for Sentinel-2).
- **Time Range**: January 1, 2024, to December 31, 2024.
- **Cloud Cover Filter**: Images with less than 10% cloud cover are selected.

## Acknowledgements

- [Google Earth Engine](https://earthengine.google.com/)
- [Sentinel-2 Data](https://sentinel.esa.int/web/sentinel/home)
