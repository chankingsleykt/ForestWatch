# May 2, 2026
* Selected XGBoost as it did best in model_comparison.ipynb
* Evaluated performance on testing data, realized model had high precision but low recall on Amazon and Taiga 2024
* Ran K-S test, confirmed shift in distribution
* Decision: users can manually adjust the probability threshold to account for effects in years
* Realized data leakage in delta calculation methods, where the delta is with the current year rather than current year - 1
    * Fixed data_collection.ipynb, generated new datasets, re-ran model_comparison.ipynb
    * All models now do disastrously worse
* TODO: build region-specific models, hopefully those do well enough, if they still fail, deep learning may be necessary.

# May 23, 2026
* Built the region-specific Amazon model. Still did badly, though there was improvement
* Upon more research, a 30.91 x 30.91 region is about as big as an Olympic-sized swimming pool. My assumption that looking at past spectral indices could "catch deforestation in the act" is wrong because deforesters could clear that region in minutes.
* Decision: pivot to classifying deforestation in real time. While this is essentially recreating the Hansen loss label, the Hansen dataset is static only up to 2025, while this model can be used for future years and dynamically called for specific regions. Essentially a GFW-lite
* TODO: retrain models on unlagged data