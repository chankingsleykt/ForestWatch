# ForestWatch: V1.0 Product & Architecture Outline

## 1. Core Identity

ForestWatch is a lightweight, API-first spatial inference engine that provides on-demand, user-calibrated deforestation detection for custom geographic regions.

Unlike traditional platforms that display pre-computed static maps, ForestWatch actively reaches out to satellite catalogs, extracts spectral data, and runs real-time machine learning inference whenever a user defines a new area of interest.

## 2. Architectural Divergence: ForestWatch vs. Global Forest Watch (GFW)

While both applications track deforestation, they solve the problem using fundamentally different software architectures. This project specifically highlights backend efficiency, dynamic inference, and user-driven calibration.

| System Feature | Global Forest Watch (WRI / UMD) | ForestWatch (This Project) | Technical Skill Demonstrated |
| --- | --- | --- | --- |
| **Inference Timing** | **Pre-computed.** Supercomputers process the globe in batches and serve static tiles to the frontend. | **Live / On-Demand.** ML inference is executed in RAM via FastAPI only when a user requests it. | Asynchronous backend orchestration, holding ML models in active memory (`lifespan`), and managing latency. |
| **Temporal Data** | **Static/Periodic.** Relies on the annual Hansen dataset or 8-day GLAD alerts. | **Dynamic Rolling Window.** Calculates the median spectral indices of the exact 365 days prior to the user's request. | Dynamic query construction in Google Earth Engine and temporal smoothing. |
| **Model Calibration** | **Black-Box.** Universal static thresholds dictate what is and isn't flagged as deforestation. | **User-in-the-Loop.** The frontend UI features a "Sensitivity Slider" that dynamically filters the raw `.predict_proba()` output. | Mastery of the Precision-Recall trade-off, Covariate Shift mitigation, and decoupling inference from visualization. |
| **Geographic Scope** | **Global.** Uses massive tiled infrastructure to render the entire Earth simultaneously. | **Targeted.** Bounded by a server-tested maximum polygon area to ensure low-latency API response times. | Load testing, scaling limits, and API payload management (GeoJSON). |

## 3. User Experience (UX) Flow

1. **Define Area:** User draws a custom polygon on an interactive web map (e.g., Leaflet or Mapbox).
2. **Set Parameters:** User adjusts the "Detection Sensitivity" slider (Strict, Balanced, Eager).
3. **Live Inference:** A loading state triggers while the backend orchestrates the Earth Engine data pull and XGBoost prediction.
4. **Visualize Live Data:** The map renders a red spatial overlay highlighting exact pixels predicted as deforested within the rolling 365-day window.
5. **Contextualize (Historical):** A side panel displays summary metrics. To save compute, historical stats (e.g., "Percentage deforested in 2022") are queried instantly from the existing Hansen dataset, contrasting the historical trend with the live ForestWatch prediction.

## 4. Backend Architecture (FastAPI MLOps)

* **API Layer:** FastAPI handles incoming GeoJSON requests asynchronously to prevent server blocking during Google Earth Engine API calls.
* **Data Pipeline (`ee.Image.getRegion`):** Extracts sub-annual spectral bands and indices using a rolling 365-day median composite. The `scale` parameter is downsampled (e.g., 30m or 50m) to keep the payload lightweight.
* **ML Execution:** A pre-trained XGBoost model (`.json`) is loaded into active memory via FastAPI lifespan events. It executes `.predict_proba()` on the tabular Pandas DataFrame generated from the Earth Engine payload.
* **Response Assembly:** The backend reconstructs the Pandas output into a localized GeoJSON FeatureCollection, appending the probability scores for the frontend to render dynamically.

## 5. Development Roadmap & Constraints

* **Phase 1 (Load Testing):** Experiment with Earth Engine's spatial scale and FastAPI's memory limits to determine the hard cap on user polygon size.
* **Phase 2 (Pipeline Integration):** Build the async Earth Engine wrapper to pull the 365-day median and historical Hansen data simultaneously.
* **Phase 3 (Model Loading):** Implement the FastAPI lifespan logic to hold the XGBoost model in RAM.
* **Phase 4 (Frontend Wiring):** Build the map UI, connect the GeoJSON response, and implement the JavaScript logic for the interactive Sensitivity Slider.