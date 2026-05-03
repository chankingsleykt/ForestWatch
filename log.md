# May 2, 2026
* Selected XGBoost as it did best in model_comparison.ipynb
* Evaluated performance on testing data, realized model had high precision but low recall on Amazon and Taiga 2024
* Ran K-S test, confirmed shift in distribution
* Decision: users can manually adjust the probability threshold to account for effects in years
* Realized data leakage in delta calculation methods, where the delta is with the current year rather than current year - 1
    * Fixed data_collection.ipynb, generated new datasets, re-ran model_comparison.ipynb
    * All models now do disastrously worse
* TODO: build region-specific models, hopefully those do well enough, if they still fail, deep learning may be necessary.