# EUDR Deforestation Risk Screening — Kodagu, Karnataka

A satellite-based deforestation risk assessment for a coffee-growing area in Kodagu District, Karnataka, India, built entirely with free and open data.

**Deliverable:** [Coorg_EUDR_Risk_Assessment.pdf](./Coorg_EUDR_Risk_Assessment.pdf)

---

## Why this matters

The EU Deforestation Regulation (EUDR) requires operators placing coffee, cocoa, rubber, palm oil, soy, wood, and cattle products on the EU market to prove those goods did not originate from land deforested after 31 December 2020. Compliance requires plot-level geolocation and evidence of deforestation-free status.

Kodagu produces a large share of India's coffee and supplies exporters serving the EU market. The region's supply chain is heavily smallholder-driven, which makes plot-level traceability and screening a practical challenge.

This project demonstrates a reproducible screening workflow using only free imagery and open-source software.

---

## Area and data

| | |
|---|---|
| **Location** | Kodagu District, Karnataka, India |
| **AOI bounds** | 75.65–75.83 E, 12.35–12.49 N (WGS 84) |
| **Area** | ~19 × 15 km |
| **Sensor** | Sentinel-2 L2A (surface reflectance) |
| **Resolution** | 10 m |
| **Dates** | 20 Feb 2021 and 19 Feb 2025 |

Both dates fall in the dry season, four calendar days apart. This was deliberate: the Western Ghats are cloud-covered for much of the year, and comparing images from different phenological stages introduces change that has nothing to do with land use.

---

## Method

**1. Vegetation index**

NDVI was computed from raw bands for both dates:

```
NDVI = (B08 - B04) / (B08 + B04)
```

Healthy vegetation strongly reflects near-infrared (B08) and absorbs red (B04), so the ratio separates vegetation from bare ground, built-up surfaces, and water.

**2. Water masking**

Water bodies produce large NDVI swings between dates purely from level changes, so pixels that were water in either year were excluded:

```
land_mask = (B08_2021 > 0.15) AND (B08_2025 > 0.15)
```

**3. Change detection**

```
change = NDVI_2025 - NDVI_2021
masked_change = change × land_mask
```

**4. Interpretation thresholds**

| ΔNDVI | Interpretation |
|---|---|
| > −0.05 | No significant change (within seasonal range) |
| −0.05 to −0.15 | Possible degradation — investigate |
| −0.15 to −0.30 | Significant vegetation loss |
| < −0.30 | Likely clearing / land use conversion |

For calibration: dense forest in this region reads 0.70–0.85 NDVI, while cleared ground reads 0.15–0.20. Genuine clearing therefore produces a drop of roughly 0.50 or more.

---

## Result

![NDVI change detection, Kodagu 2021-2025](change_map.png)

**No evidence of large-scale forest clearing was detected.** Vegetation cover remained stable across the majority of the assessed area.

### One signal investigated and excluded

A high-magnitude change of **−0.35** appeared near a water body. Rather than reporting it as vegetation loss, the underlying band values were examined:

| | B04 (Red) | B08 (NIR) | NDVI |
|---|---|---|---|
| **Feb 2021** | 0.085 | 0.197 | 0.398 |
| **Feb 2025** | 0.279 | 0.310 | 0.052 |

Both bands **increased**, with red reflectance more than tripling. Vegetation removal produces the opposite pattern — near-infrared reflectance falls when canopy is lost. The observed signature is consistent with a reservoir water-level change exposing bright sediment, which visual inspection of high-resolution imagery confirmed.

**This signal was excluded from the findings.** A fixed NIR threshold does not reliably mask shallow, turbid water over bright substrate.

---

## Limitations

These are stated plainly because they affect how the output should be used.

- **Terrain illumination.** Kodagu sits in the Western Ghats on steep slopes. Differing solar geometry between acquisition dates changes apparent brightness on slopes independently of land cover. No topographic correction was applied.
- **Cryptic deforestation.** Coffee in this region is grown under native shade trees. Conversion of forest to shade-grown plantation can leave canopy cover largely intact, making it difficult to detect with NDVI alone. Canopy structure metrics (SAR or LiDAR-derived) would be required.
- **Water masking is imperfect.** As shown above, a single-threshold NIR mask does not capture all water-related change.
- **This is a risk screening, not a legal certification.** Outputs indicate where further investigation is warranted; they do not constitute a due diligence statement under EUDR.

---

## Tools

- **QGIS 3.44** — raster calculation, styling, zonal statistics, print layout
- **Copernicus Browser** — Sentinel-2 data access
- No paid software or commercial imagery was used.

---

## Reproducing this

1. Open Copernicus Browser and navigate to the AOI, zoomed so the scale bar reads ~2 km
2. Filter: Sentinel-2 L2A, cloud cover < 10%, dry season dates
3. Verify data exists on the chosen date before downloading (switch to the NDVI visualisation — if the map does not change, there is no acquisition)
4. Download via the Analytical tab: TIFF 32-bit float, HIGH resolution, raw bands B04 and B08 only
5. Repeat for the second date **without moving the map**, so both rasters share an extent
6. Compute NDVI, mask, and difference in the QGIS Raster Calculator using the expressions above

Keeping the AOI small matters: Copernicus Browser caps exports at 2500 px, so a large area is silently downsampled. A 19 km AOI returns true 10 m pixels; a 100 km AOI returns roughly 55 m pixels, which is too coarse for plot-level work.

---

## Data attribution

Contains modified Copernicus Sentinel data (2021, 2025), processed by Rohan Kumar.

---

## Contact

**Rohan Kumar**
rohanthekumar@gmail.com

Available for geospatial analysis work: deforestation screening, land use change detection, vegetation monitoring, and EUDR supply chain risk assessment.
