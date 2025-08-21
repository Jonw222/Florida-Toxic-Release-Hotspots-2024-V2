# Florida-Toxic-Release-Hotspots-2024-V2
Updated Interactive map of Florida Toxic Release Hotspots (TR)I 2024 facility releases using proportional circles, chemical filters, clustering, and a Space Coast zoom. It is built with Leaflet.

# Data Source and Sample
### Primary Dataset
 EPA Toxics Release Inventory (TRI) 2024 Preliminary, Basic Data Files, Florida subset is the primary dataset (downloaded data is in the assignment folder): https://www.epa.gov/toxics-release-inventory-tri-program/2024-tri-preliminary-dataset-basic-data-files
 These state-level CSV includes facility location, chemical identifiers, and quantities for on site releases and off site transfers. EPA confirms the 2024 preliminary files include forms processed as of July 9, 2025, and that “Basic” files contain the 100 most-used TRI fields in CSV format. According to the EPA’s data dictionary for the Basic files, the CSVs include ```YEAR, TRIFID (facility ID), FRS ID, FACILITY NAME, CITY, STATE, LATITUDE, LONGITUDE, PRIMARY NAICS, CHEMICAL```, and aggregated quantities such as ```TOTAL RELEASES and off site totals```. Coordinates come from EPA’s Facility Registry Service. 

### Geospatial Enrichment
Facility coordinates and crosswalks are governed by the Facility Registry Service (FRS). The FRS State Single File CSV is available for Florida and contains facility identifiers, geospatial fields, and program linkages. This is the authoritative reference used by TRI for location, which is why TRI LATITUDE and LONGITUDE are already reliable. 

### Cleaning Choices 
- Florida filter: kept rows where STATE == FL.
- Coordinates: kept rows where LATITUDE and LONGITUDE parsed to finite numbers within valid geographic ranges (lat −90 to 90, lon −180 to 180). Rows with missing, nonnumeric, or out-of-range coordinates were excluded.
- Numeric fields: Removed comma digit grouping from large values, for example 12,345 becomes 12345, then stored the results as numeric values. Applied to ```ON_SITE_RELEASE_TOTAL, OFF_SITE_RELEASE_TOTAL, and TOTAL_RELEASES```.
- Total logic: If TOTAL_RELEASES was missing or zero but either component had a value, set TOTAL_RELEASES to ON_SITE_RELEASE_TOTAL plus OFF_SITE_RELEASE_TOTAL.
- Column standardization: trimmed to these fields only to simplify the first map release: ```YEAR, TRIFID, FRS_ID, FACILITY_NAME, CITY, STATE, LATITUDE, LONGITUDE, PRIMARY_NAICS, CHEMICAL, ON_SITE_RELEASE_TOTAL, OFF_SITE_RELEASE_TOTAL, TOTAL_RELEASES```.
- Units: pounds for all quantities. TRI reports dioxin compounds in grams; those values remain as provided by EPA and will look comparatively small.

### Cleaned Dataset Summary 
- Saved as: ```data/tri_fl_2024_basic_clean.csv```
- Florida scope: 1,735 rows representing approximately 700 distinct facilities by TRIFID.
- Numbers: commas removed and converted to numeric types; totals calculated when missing.

# Methodology

### Data Acquisition & Normalization
The Florida subset of the EPA TRI 2024 Preliminary “Basic” files was ingested in-browser using Papa Parse. Fields were trimmed to ```YEAR, TRIFID, FRS_ID, FACILITY_NAME, CITY, STATE, LATITUDE, LONGITUDE, PRIMARY_NAICS, CHEMICAL, ON_SITE_RELEASE_TOTAL, OFF_SITE_RELEASE_TOTAL, TOTAL_RELEASES```. Coordinates were retained only when finite and plausibly in-range; numeric fields were de-formatted (e.g., “12,345” → ```12345```) and cast to numbers. When ```TOTAL_RELEASES``` was absent but its components were present, it was imputed as the sum of on-site and off-site totals. All units are pounds (TRI dioxins remain in grams per EPA reporting).

### Feature Engineering
- Deterministic color mapping: chemical name → integer hash → Chroma.js categorical scale; ensures stable colors across filtering and panning.
- Size transform: circle radius ```r = min(MAX_RADIUS, 0.20 * sqrt(TOTAL_RELEASES))```, with a small-radius floor for visibility.
- Heatmap weighting: per-point intensity ```max(1, log10(TOTAL_RELEASES))``` to mitigate heavy-tail dominance.
- Brevard focus: axis-aligned bounding box filter to support a repeatable “Space Coast” case study.

# Topic and Geographic Phenomena
- What:
 Facility reported toxic chemical releases and transfers relevant to environmental health and exposure potential.
- Where:
 Florida statewide, with attention to clusters near the Space Coast as a use case.
- When:
Reporting year: 2024.
- Title:
Florida Toxics Release Hotspots 2024
- Subtitle:
 Mapping facility reported releases and transfers that matter for environmental health

### Thematic Methods
 - Proportional circles sized by TOTAL_RELEASES using a square-root transform with an upper radius cap for legibility.
 - Per-chemical color encoding via a deterministic chroma scale hashed to chemical name to ensure stable, repeatable hues across interactions.
 - Heatmap density (optional) powered by Leaflet.heat; intensity is log-weighted (log10(TOTAL_RELEASES), floored at 1) to preserve contrast in the presence of heavy-tailed values.
 - Legend design combines a compact size scale (proportional dots with labeled pound values) and an in-view Top chemicals color key (swatches driven by the same color function).
 - Clustering at small scales with MarkerCluster to reduce occlusion without discarding outliers.
 - Sidebar summary chart: Top 6 chemicals bar chart (Chart.js) reflecting the current map filters/extent; x-axis labels are wrapped/abbreviated to prevent overlap while the chart height remains fixed.

### Anticipated UI
 - Chemical dropdown filter and Quick filter chips for common chemicals.
 - Minimum release slider (0–500,000 lbs) with dynamic “min/current/max” labels.
 - Brevard County only toggle (axis-aligned bbox) and Zoom to Space Coast control for focused exploration.
 - Heatmap density toggle to switch between proportional circles and density visualization.
 - Text search (facility or city) that flies the map to the first match.
 - Live counters for facilities shown and number of distinct chemicals in view.
 - Export current filter (CSV) generated client-side from the active subset.
 - Legend embedded on the map: size scale + top-chemicals color swatches; compact to avoid obscuring the map.
 - Basemap switcher (OSM Standard, OSM Humanitarian, Esri World Imagery) and metric scale bar.
 - About modal dialog and loading spinner for data fetch feedback.
 
# Map Objectives and User Needs
### Why this Map?
This TRI map establishes a Florida wide baseline of facility reported chemical releases that directly supports my dissertation on chemical contamination around spaceports. The map lets users see which facilities report which chemicals, where they are located, and visually inspect concentrations near the Space Coast focus area. Clear symbology, interactive filtering, and transparent units allow users to relate facility reports to potential pathways into water bodies and to compare these patterns with independent datasets on wastewater, microbial resistance, and aerospace activity.

### Intended Users and Needs
 - Residents and advocacy groups can use it to locate nearby facilities, see which chemicals are present, and judge amounts for health and advocacy decisions.
 - Journalists and local officials can use it to spot hotspots, verify sites, and move from statewide patterns to facility detail for reporting, planning, and enforcement.
 - To provide students and researchers with a documented and reproducible dataset for further analysis

# Technology Stack
- Data processing tools
 QGIS for quick inspection and field trimming when needed. The “clean” file is a flat CSV that preserves standardized quantities and coordinates described in US Enviromental Protection Agency (EPA) documentation. 
- Browser libraries
Leaflet (map + L.control.layers for basemaps), Leaflet.markercluster (decluttering), Leaflet.heat (density layer), Chart.js (top-chemicals bar chart), Papa Parse (CSV load and client-side export via unparse), and Chroma.js (deterministic categorical coloring), markercluster for legibility when zoomed out.

# Limitations and Data Caveats.
- TRI off-site transfers may represent quantities transported elsewhere, not necessarily released at the reporting coordinate.
- Facility coordinates denote site centroids, not discharge outfalls.
- 2024 files are preliminary (processed as of July 9, 2025); late submissions and revisions may change totals.
