# PFAS Source-Pressure Screening for Mississippi Sound Watersheds

An interactive web map of a screening-level PFAS source-pressure index for the
coastal-connected HUC-12 watersheds that drain to the Mississippi Sound, together with the
terrain, hydrology, and source-inventory layers it was built from.

PFAS = per- and polyfluoroalkyl substances.

**[▶ Open the interactive map](https://nishatshermin.github.io/pfas-mississippi-sound-map/)**

---

## What this is, and what it is not

This is a relative, dimensionless ranking. Source pressure is a 0–1 index produced by a
weighted overlay of six proximity-and-density variables, then routed downstream along a
flow network derived from a digital elevation model. It addresses one question: given where
known PFAS-related facilities sit, and how water moves across this landscape, which
watersheds and stream reaches are the more plausible places to sample first?

It is not the following things:

- **Not a predicted concentration.** The index has no units. A value of 0.6 does not
  correspond to 0.6 of any measurable quantity, and values cannot be converted to ng/L.
- **Not a fate-and-transport model.** It contains no discharge, advection, dilution,
  degradation, or partitioning, and no time dimension. Hydrodynamic modelling is planned
  separately (EFDC for the Sound, SWMM for watershed loading time series). This screening is
  the step that precedes it.
- **Not validated.** No measured PFAS concentration was used to fit, calibrate, or test the
  index. Nothing here has been checked against observations.
- **Not a statement about any individual facility.** A high value near a site reflects the
  severity weight assigned to that site's *class*, such as military/AFFF or industrial, and
  not any finding about the site itself.

The index is intended to prioritise where to sample and where to focus later modelling. It
should not be used to make claims about contamination at a location.

---

## How the index is built

Six variables are computed on a 30 m grid, each normalised to 0–1, then combined by weighted
sum and rescaled to 0–1:

| | Variable | Built from |
|---|---|---|
| V1 | Nearest PFAS-related source, severity-weighted | EWG PFAS sites, EPA FRS industrial facilities |
| V2 | Density of weighted sources, 10 km moving window | EWG PFAS sites, EPA FRS industrial facilities |
| V3 | Nearest WWTP or NPDES discharge | EPA ECHO |
| V4 | Nearest airport or military fire-training site | OurAirports, EWG military sites |
| V5 | Nearest stormwater outfall, permit proxy | EPA ECHO stormwater permits |
| V6 | Nearest river mouth or bay inlet | USGS NHD, intersected with the model-grid boundary |

```
SourcePressure = 0.30·V1 + 0.20·V2 + 0.15·V3 + 0.15·V4 + 0.10·V5 + 0.10·V6
```

Distance decay is exponential, with an 8 km scale (10 km for V6). Facility severity weights
inside V1 and V4 form a second, separate layer of weighting: 1.00 for military/AFFF sites and
sites confirmed above the limit, 0.75 for industrial facilities and large airports, 0.50 for
WWTPs and sites confirmed below the limit, and 0.25 to 0.40 for stormwater permits and small
airports. These were adapted from published PFAS sampling-prioritisation schemes. The
combination weights in the equation and the severity weights are two different things, and
they are easy to confuse when reading the map.

The surface is then routed downstream using D8 flow direction over a priority-flood-filled
DEM, which gives each stream reach a flow-weighted upstream mean (`SP_MEANUP`).

The study-area scope is deliberately narrow. It includes only watersheds that contain a PFAS
source, plus every watershed downstream of them toward the coast. Upstream contributions are
excluded at this stage to keep the problem tractable, which is a simplification and a known
limitation.

---

## Map layers

### Model inputs and outputs

| Layer | Count | Popup attributes |
|---|---|---|
| Source pressure, 30 m raster | | *(see the per-watershed and per-reach layers)* |
| Watersheds (HUC-12) | 115 | HUC-12 code, name, type, role, states, area, mean and max source pressure, cell count |
| Streams with source pressure | 1,478 | Reach ID, Strahler order, drainage area, flow-weighted SP, accumulated SP, local SP mean and max, length, slope, upstream and downstream elevation, HUC-12 |
| PFAS occurrence points (EWG) | 141 | Type (military/AFFF, above limit, below limit), source ID |
| NPDES discharges | 2,917 | Name, permit ID, class. Clustered; 18,348 were downloaded statewide and clipped to the study area |
| Airports | 124 | Name, type, identifier |
| Industrial facilities | 238 | Facility name |

### Terrain and hydrology derived from the DEM

Hillshade, elevation, slope, aspect, flow accumulation, and source pressure averaged over
each cell's upstream contributing area.

### Reference context, not model inputs

| Layer | Count | Note |
|---|---|---|
| EPA UCMR5 systems with PFAS detected | 55 | Measured drinking-water results |
| EPA UCMR5 systems with no detection | 414 | |

UCMR5 did not contribute to the index. It is displayed alongside the index as independent
context, and it is the dataset a future validation step would use. Two caveats apply:

1. **Positions are approximate.** UCMR5 reports by public water system ID and does not
   publish intake coordinates. Each system is plotted at the mean centroid of the ZIP codes
   it serves, using the UCMR5 ZIP-code crosswalk and Census ZCTA centroids. A marker
   indicates which system reported a result, not where water was withdrawn. Of 507 sampled
   systems in Mississippi and Louisiana, 469 could be located; 38 had no ZIP match and are
   not shown.
2. **It measures treated drinking water**, not surface water in the modelled network, so it
   is not a direct observation of the quantity the index ranks.

### Other

The EFDC model domain layer shows the footprint of the planned hydrodynamic model grid.

---

## Key numbers

| | |
|---|---|
| Study area | 115 HUC-12 watersheds, 11,691 km² |
| Watershed roles | 20 source, 95 downstream |
| Screened to reach that set | 3,271 HUC-12 units across Mississippi and Louisiana |
| Stream network | 1,478 reaches, 4,921 km, maximum Strahler order 6, maximum drainage area 23,353 km² |
| Grid | 30 m, EPSG:26916 (NAD83 / UTM zone 16N), 12,989,810 valid cells |
| Source pressure per 30 m cell | range 0.058 to 1.000, mean 0.293, median 0.258, P90 0.485 |
| Source pressure per watershed mean | range 0.085 to 0.694, mean 0.303, median 0.278 |
| UCMR5 | 469 systems mapped, 55 with at least one PFAS detection, maximum 72.2 ng/L |

The two source-pressure rows describe different populations, individual grid cells versus
the 115 watershed averages, and are not interchangeable. Averaging over a watershed pulls in
the extremes, which is why the watershed range is narrower.

Three open-water HUC-12 units (Chandeleur Sound, Biloxi Bay-Mississippi Sound, and Lake
Maurepas) were removed from an earlier 118-watershed version, because they are water bodies
rather than land watersheds. Any figure citing 118 watersheds or roughly 16,470 km² predates
that correction.

---

## Data sources

Every layer comes from a public federal dataset or an openly licensed compilation. The table
gives the dataset, the page it is documented on, and the exact endpoint the data was
retrieved from.

| Layer | Dataset | Documentation | Retrieved from |
|---|---|---|---|
| PFAS occurrence points | EWG PFAS Contamination Map | [ewg.org](https://www.ewg.org/interactive-maps/pfas_contamination/map/) | GeoJSON layers behind the map: `Military_2024JUL16.geojson`, `sdwis_12FEB2026_above.geojson`, `sdwis_12FEB2026_below.geojson` |
| Drinking-water PFAS results | EPA UCMR5, Fifth Unregulated Contaminant Monitoring Rule (2023–2025) | [epa.gov/dwucmr](https://www.epa.gov/dwucmr/occurrence-data-unregulated-contaminant-monitoring-rule) | [ucmr5-occurrence-data-by-state.zip](https://www.epa.gov/system/files/other-files/2023-08/ucmr5-occurrence-data-by-state.zip) |
| UCMR5 ZIP-code crosswalk | `UCMR5_ZIPCodes.txt` | as above | included in the same UCMR5 archive |
| Permitted discharges (NPDES) | EPA ECHO, Clean Water Act services | [echo.epa.gov/tools/web-services](https://echo.epa.gov/tools/web-services) | `https://echodata.epa.gov/echo/cwa_rest_services.get_facilities` then `.get_download` |
| Public water system attributes | EPA ECHO, Safe Drinking Water Act services | [echo.epa.gov/tools/web-services](https://echo.epa.gov/tools/web-services) | `https://echodata.epa.gov/echo/sdw_rest_services.get_systems` then `.get_download` |
| Industrial facilities | EPA Facility Registry Service, state combined files, filtered to facilities holding a TRIS (Toxics Release Inventory System) program registration | [epa.gov/frs](https://www.epa.gov/frs/epa-state-combined-csv-download-files) | [state_combined_ms.zip](https://ordsext.epa.gov/FLA/www3/state_files/state_combined_ms.zip), [state_combined_la.zip](https://ordsext.epa.gov/FLA/www3/state_files/state_combined_la.zip) |
| Airports | OurAirports, public domain | [ourairports.com/data](https://ourairports.com/data/) | [airports.csv](https://davidmegginson.github.io/ourairports-data/airports.csv) |
| Stream network | USGS National Hydrography Dataset, high-resolution state GeoPackages, `NHDFlowline` layer | [usgs.gov/national-hydrography](https://www.usgs.gov/national-hydrography) | [NHD_H_Mississippi_State_GPKG.zip](https://prd-tnm.s3.amazonaws.com/StagedProducts/Hydrography/NHD/State/GPKG/NHD_H_Mississippi_State_GPKG.zip), [NHD_H_Louisiana_State_GPKG.zip](https://prd-tnm.s3.amazonaws.com/StagedProducts/Hydrography/NHD/State/GPKG/NHD_H_Louisiana_State_GPKG.zip) |
| Watershed boundaries | USGS Watershed Boundary Dataset, `WBDHU12` layer, distributed inside the same NHD state GeoPackages | [usgs.gov/national-hydrography/watershed-boundary-dataset](https://www.usgs.gov/national-hydrography/watershed-boundary-dataset) | the two NHD GeoPackages above |
| Elevation | USGS 3DEP, 1 arc-second (about 30 m) | [usgs.gov/3d-elevation-program](https://www.usgs.gov/3d-elevation-program) | `https://prd-tnm.s3.amazonaws.com/StagedProducts/Elevation/1/TIFF/current/{tile}/USGS_1_{tile}.tif` |
| ZIP code centroids | Census Bureau 2024 Gazetteer, ZCTA | [census.gov gazetteer](https://www.census.gov/geographies/reference-files/time-series/geo/gazetteer-files.html) | [2024_Gaz_zcta_national.zip](https://www2.census.gov/geo/docs/maps-data/data/gazetteer/2024_Gazetteer/2024_Gaz_zcta_national.zip) |
| County and state boundaries | Census Bureau TIGER/Line 2024 | [census.gov TIGER](https://www.census.gov/geographies/mapping-files/time-series/geo/tiger-line-file.html) | [tl_2024_us_county.zip](https://www2.census.gov/geo/tiger/TIGER2024/COUNTY/tl_2024_us_county.zip), [tl_2024_us_state.zip](https://www2.census.gov/geo/tiger/TIGER2024/STATE/tl_2024_us_state.zip) |
| Basemap tiles | Esri Light Gray Canvas, Light Gray Reference, World Topographic, World Imagery | [Esri basemaps](https://server.arcgisonline.com/ArcGIS/rest/services) | `https://server.arcgisonline.com/ArcGIS/rest/services/{service}/MapServer/tile/{z}/{y}/{x}` |

USGS and EWG pages return HTTP 403 to command-line tools but open normally in a browser.
Basemap tiles are served by Esri under its free basemap terms, and attribution appears in
the bottom-right corner of the map.

Two attributions in earlier versions of this project were imprecise, and are corrected here.
The industrial layer was described as "EPA TRI 2024". It is drawn from the EPA Facility
Registry Service state combined files, selecting facilities that carry a TRIS program
registration, so it identifies TRI-registered facilities rather than a specific TRI reporting
year. Watershed boundaries were listed as a standalone WBD download; the `WBDHU12` layer was
in fact read from the NHD state GeoPackages, which distribute it.

---

## Limitations

- The six variables and their weights come from literature-informed judgement, not from a
  fitted model. Other defensible weights would reorder some watersheds. No sensitivity
  analysis has been run.
- Distances are Euclidean, so the index does not yet account for a source that sits across a
  drainage divide and is hydrologically distant while spatially near. Hydrologic distance is
  a planned refinement.
- Landfills are absent. They score highly in the published prioritisation literature, but no
  landfill point layer has been added.
- Stormwater is represented by permit locations rather than physical outfalls, which are not
  publicly available statewide.
- Streams are derived from the DEM rather than surveyed. Reaches that ran straight across
  flat open water were flagged and removed, so the network is conservative and some real
  channels in the flattest coastal terrain are missing.
- Source inventories are incomplete by nature. The absence of a point means the absence of a
  public record, not the absence of a source.
- The EWG "above limit" and "below limit" classes are taken from that compilation and were
  not independently re-verified against the underlying state records.

---

## Technical notes

Raster overlays are warped to EPSG:3857 (Web Mercator) before being embedded. Leaflet's
`ImageOverlay` stretches an image linearly in Web Mercator screen space, so embedding a
WGS84 image instead produces a north-south drift of roughly a kilometre at this latitude.
Rasters are colour-mapped and palette-quantised to 128 colours at about 1,400 px, and vector
geometry is simplified with coordinates rounded. Roughly 1 GB of source layers reduce to a
single self-contained HTML file of about 8.5 MB, with no build step.

Leaflet and the basemap tiles load from the internet. Offline, every project layer including
the hillshade still draws, but the basemap will be blank.

Built with Python: GeoPandas, rasterio, Folium/Leaflet, NumPy, Numba, and Matplotlib.

---

## Repository contents

```
index.html    the interactive map, self-contained, about 8.5 MB
README.md     this file
```

The full project, including DEM tiles, intermediate hydrology rasters, and source-pressure
grids, runs to tens of gigabytes and is not version-controlled here. The map embeds
everything needed to view the results.

---

## Citation

> Shermin, N. (2026). *PFAS source-pressure screening for coastal-connected watersheds of
> the Mississippi Sound* [Interactive map]. Mississippi State University.

This work is part of a Gulf Futures Challenge PFAS project at Mississippi State University.
The screening supports, and precedes, hydrodynamic modelling of PFAS transport into the
Mississippi Sound.

## Licence

The underlying data are US federal public-domain products or openly licensed compilations,
as listed above. A licence for the derived map and code has not yet been chosen. CC BY 4.0
for the map and data products, paired with MIT for the code, is a common and appropriate
combination, but confirm it against your advisor's and MSU's requirements first, then add a
`LICENSE` file and record the choice here.

---

*Prepared for research use. Verify against the original sources before any regulatory,
planning, or public-health application.*
