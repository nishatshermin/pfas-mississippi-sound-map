# PFAS Source-Pressure Screening — Mississippi Sound Watersheds

An interactive web map of a **screening-level PFAS source-pressure index** for the
coastal-connected HUC-12 watersheds draining to the Mississippi Sound, together with the
terrain, hydrology and source-inventory layers it was built from.

PFAS = per- and polyfluoroalkyl substances.

**[▶ Open the interactive map](https://nishatshermin.github.io/pfas-mississippi-sound-map/)**

---

## What this is — and what it is not

**It is** a relative, dimensionless ranking. Source pressure is a 0–1 index produced by a
weighted overlay of six proximity-and-density variables, then routed downstream along a
DEM-derived flow network. It answers one question: *given where known PFAS-related
facilities sit and how water moves across this landscape, which watersheds and stream
reaches are the more plausible places to look first?*

**It is not:**

- **Not a predicted concentration.** The index has no units. A value of 0.6 does not mean
  0.6 of anything measurable, and values cannot be converted to ng/L.
- **Not a fate-and-transport model.** There is no discharge, no advection, no dilution, no
  degradation, no partitioning and no time dimension. Hydrodynamic modelling (EFDC for the
  Sound, SWMM for watershed loading time series) is planned separately; this screening is
  the step that comes before it.
- **Not validated.** No measured PFAS concentration was used to fit, calibrate or test the
  index. Nothing here has been checked against observations.
- **Not a statement about any specific facility.** A high value near a site reflects that
  site's *class* (e.g. military/AFFF, industrial), not any finding about that site.

Use it to prioritise where to sample and where to focus later modelling. Do not use it to
make claims about contamination at a location.

---

## How the index is built

Six variables, each computed on a 30 m grid, each normalised to 0–1, combined by weighted
sum and rescaled to 0–1:

| | Variable | Built from |
|---|---|---|
| V1 | Nearest PFAS-related source, severity-weighted | EWG PFAS sites + EPA TRI |
| V2 | Density of weighted sources (10 km moving window) | EWG PFAS sites + EPA TRI |
| V3 | Nearest WWTP / NPDES discharge | EPA ECHO |
| V4 | Nearest airport / military fire-training site | OurAirports + EWG military sites |
| V5 | Nearest stormwater outfall (permit proxy) | EPA ECHO stormwater permits |
| V6 | Nearest river mouth / bay inlet | USGS NHD × model-grid boundary |

```
SourcePressure = 0.30·V1 + 0.20·V2 + 0.15·V3 + 0.15·V4 + 0.10·V5 + 0.10·V6
```

Distance decay is exponential (8 km scale; 10 km for V6). Facility severity weights within
V1 and V4 are a separate layer of weighting — 1.00 military/AFFF and confirmed-above-limit
sites, 0.75 industrial and large airports, 0.50 WWTP and confirmed-below-limit sites,
0.25–0.40 stormwater permits and small airports — adapted from published PFAS
sampling-prioritisation schemes. **The combination weights above and the severity weights
are two different things; they are easy to confuse when reading the map.**

The surface is then routed downstream with D8 flow direction over a priority-flood-filled
DEM, giving each stream reach a flow-weighted upstream mean (`SP_MEANUP`).

**Study-area scope is deliberately narrow.** Only watersheds that contain a PFAS source,
plus every watershed downstream of them toward the coast, are included. Upstream
contributions are excluded at this stage to keep the problem tractable — a simplification,
and a known limitation.

---

## Map layers

### Model inputs and outputs

| Layer | Count | In the popup |
|---|---|---|
| Source pressure (30 m raster) | — | *(raster; see per-watershed and per-reach layers)* |
| Watersheds (HUC-12) | 115 | HUC-12 code, name, type, role, states, area, mean/max source pressure, cell count |
| Streams (source pressure) | 1,478 | Reach ID, Strahler order, drainage area, flow-weighted SP, accumulated SP, local SP mean/max, length, slope, up/downstream elevation, HUC-12 |
| PFAS occurrence points (EWG) | 141 | Type (military/AFFF, above limit, below limit), source ID |
| NPDES discharges | 2,917 | Name, permit ID, class *(clustered; 18,348 downloaded statewide, clipped to the study area)* |
| Airports | 124 | Name, type, identifier |
| Industrial facilities (TRI 2024) | 238 | Facility name |

### Terrain and hydrology (derived from the DEM)

Hillshade · Elevation · Slope · Aspect · Flow accumulation · Source pressure averaged over
each cell's upstream contributing area.

### Reference context — not model inputs

| Layer | Count | Note |
|---|---|---|
| EPA UCMR5 systems — PFAS detected | 55 | Measured drinking-water results |
| EPA UCMR5 systems — no detection | 414 | |

**UCMR5 did not contribute to the index.** It is displayed alongside it as independent
context, and it is the dataset that would be used in a future validation step. Two caveats
matter when reading it:

1. **Positions are approximate.** UCMR5 reports by public water system ID and does not
   publish intake coordinates. Each system is plotted at the mean centroid of the ZIP codes
   it serves (via the UCMR5 ZIP-code crosswalk and Census ZCTA centroids). A marker shows
   *which system reported a result*, not where water was withdrawn. 469 of 507 sampled
   MS + LA systems could be located; 38 had no ZIP match and are not shown.
2. **It measures treated drinking water**, not surface water in the modelled network. It is
   not a direct observation of the quantity the index ranks.

### Other

EFDC model domain — the footprint of the planned hydrodynamic model grid.

---

## Key numbers

| | |
|---|---|
| Study area | 115 HUC-12 watersheds, 11,691 km² |
| Watershed roles | 20 source · 95 downstream |
| Screened to get there | 3,271 HUC-12 units across MS + LA |
| Stream network | 1,478 reaches, 4,921 km, max Strahler order 6, max drainage area 23,353 km² |
| Grid | 30 m, EPSG:26916 (NAD83 / UTM zone 16N), 12,989,810 valid cells |
| Source pressure, **per 30 m cell** | range 0.058–1.000, mean 0.293, median 0.258, P90 0.485 |
| Source pressure, **per watershed mean** | range 0.085–0.694, mean 0.303, median 0.278 |
| UCMR5 | 469 systems mapped, 55 with ≥1 PFAS detection, max 72.2 ng/L |

The two source-pressure rows describe different things — individual grid cells versus the
115 watershed averages — and are not interchangeable. Averaging over a watershed pulls in
the extremes, which is why its range is narrower.

Open-water HUC-12 units (Chandeleur Sound, Biloxi Bay–Mississippi Sound, Lake Maurepas)
were removed from an earlier 118-watershed version, since they are water bodies rather than
land watersheds. Any figure citing 118 watersheds or ~16,470 km² predates that correction.

---

## Data sources

All inputs are public domain or openly licensed. No proprietary or restricted data.

| Data | Source |
|---|---|
| PFAS occurrence sites | Environmental Working Group (EWG) PFAS Contamination Map |
| Drinking-water PFAS results | US EPA **UCMR5** — Fifth Unregulated Contaminant Monitoring Rule (2023–2025) |
| Permitted discharges | US EPA **ECHO** / NPDES |
| Industrial facilities | US EPA **TRI** — Toxics Release Inventory, 2024 |
| Airports | OurAirports (public domain) |
| Watershed boundaries | USGS **WBD** — Watershed Boundary Dataset, HUC-12 |
| Elevation | USGS **3DEP**, 1 arc-second (~30 m) |
| Reference stream network | USGS **NHD** — National Hydrography Dataset |
| ZIP code centroids | US Census Bureau 2024 Gazetteer (ZCTA) |
| County boundaries | US Census Bureau TIGER/Line 2024 |
| Basemap tiles | Esri — Light Gray Canvas, World Topographic, World Imagery |

Basemap tiles are served by Esri under its free basemap terms; attribution is shown in the
map's bottom-right corner.

---

## Limitations

- The six variables and their weights are **literature-informed judgement, not a fitted
  model**. Different defensible weights would reorder some watersheds. No sensitivity
  analysis has been run yet.
- **Distances are Euclidean**, so the index does not yet know that a source across a
  drainage divide is hydrologically far away even when it is spatially near. Hydrologic
  distance is a planned refinement.
- **Landfills are missing.** They score highly in the published prioritisation literature,
  but no landfill point layer has been added yet.
- **Stormwater is a permit proxy**, not physical outfall locations, which are not publicly
  available statewide.
- **Streams are DEM-derived, not surveyed.** Reaches that ran straight across flat open
  water were flagged and removed, so the network is conservative and some real channels in
  the flattest coastal terrain are absent.
- **Source inventories are incomplete by nature.** Absence of a point means absence of a
  public record, not absence of a source.
- The EWG "above limit" / "below limit" classes come from that compilation and were not
  independently re-verified against the underlying state records.

---

## Technical notes

Raster overlays are warped to **EPSG:3857 (Web Mercator)** before being embedded, because
Leaflet's `ImageOverlay` stretches an image linearly in Web Mercator screen space —
embedding a WGS84 image instead produces a north–south drift of roughly a kilometre at this
latitude. Rasters are colour-mapped and palette-quantised to 128 colours at ~1,400 px;
vector geometry is simplified and coordinates rounded. About 1 GB of source layers reduce to
a single self-contained ~8.5 MB HTML file with no build step and no external dependencies
beyond Leaflet and the basemap tiles.

Leaflet and the basemap tiles load from the internet. Offline, every project layer —
including the hillshade — still draws, but the basemap will be blank.

Built with Python (GeoPandas, rasterio, Folium/Leaflet, NumPy, Numba, Matplotlib).

---

## Repository contents

```
index.html    the interactive map (self-contained, ~8.5 MB)
README.md     this file
```

The full project — DEM tiles, intermediate hydrology rasters, source-pressure grids — runs
to tens of gigabytes and is not version-controlled here. The map embeds everything needed
to view the results.

---

## Citation

> Shermin, N. (2026). *PFAS source-pressure screening for coastal-connected watersheds of
> the Mississippi Sound* [Interactive map]. Mississippi State University.

Part of a Gulf Futures Challenge PFAS project at Mississippi State University. The screening
supports, and precedes, hydrodynamic modelling of PFAS transport into the Mississippi Sound.

## Licence

Underlying data are US federal public-domain products or openly licensed (see sources
above). **Choose a licence for the derived map and code before publishing** — CC BY 4.0 for
the map and data products with MIT for the code is a common and appropriate pairing, but
confirm it against your advisor's and MSU's requirements first, then add a `LICENSE` file
and state the choice here.

---

*Prepared for research use. Verify against the original sources before any regulatory,
planning, or public-health application.*
