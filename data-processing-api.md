API Reference: Data Normalization EndpointThe /normalize endpoint accepts a raw dataset (JSON) and returns a clean, normalized feature set suitable for machine learning models.Post /v1/data/normalizeUse this endpoint to remove null values, scale numerical features between 0 and 1, and encode categorical variables.Request ParametersParameterTypeRequiredDescriptiondataset_idstringYesThe unique UUID of the uploaded dataset.methodstringNoThe scaling method. Options: minmax (default) or standard.drop_nullsbooleanNoIf true, rows containing NaN values are removed. Default: true.Example Request (Python)Pythonimport requests
url = "https://api.analytics-engine.com/v1/data/normalize"payload = {"dataset_id": "a1b2-c3d4-e5f6","method": "minmax","drop_nulls": True}headers = {"Authorization": "Bearer YOUR_API_KEY"}response = requests.post(url, json=payload, headers=headers)print(response.json())
### Success Response (200 OK)
JSON{
  "status": "success",
  "processed_rows": 1500,
  "download_url": "https://cdn.analytics-engine.com/processed/clean_data.csv"
}
