# Agro AI: Smart Crop Advisory Web App

Agro AI is a Flask-based web application that helps farmers and agronomists make practical decisions in three areas:

1. Crop recommendation from soil and weather conditions.
2. Fertilizer guidance from soil nutrients and intended crop.
3. Plant disease detection from leaf images, with treatment/prevention advice.

The project combines classical machine learning (for crop recommendation), deep learning (for image disease classification), and rule/data-driven recommendations (for fertilizer suggestions).

## Table of Contents

1. Overview
2. Key Features
3. Project Architecture
4. Tech Stack
5. Folder and Component Breakdown
6. Request Flow
7. Setup and Installation
8. Running the Application
9. API/Route Reference
10. Model and Data Artifacts
11. Deployment Notes
12. Known Limitations
13. Troubleshooting
14. Future Improvements

## Overview

The app serves HTML pages for user input and returns model-driven recommendations:

- Crop module accepts N, P, K, pH, rainfall, and city.
- Weather (temperature and humidity) is fetched from OpenWeatherMap for the selected city.
- Features are fed to a pre-trained Random Forest model to predict the best crop label.
- Fertilizer module compares user N-P-K values with crop-specific target values from a CSV and returns nutrient correction advice.
- Disease module classifies uploaded plant images using a ResNet9 model and returns disease details and treatment guidance.

## Key Features

- Multi-service agricultural advisory from one interface.
- Real-time weather integration for crop recommendation.
- 38-class crop disease recognition via PyTorch model.
- Human-readable treatment/prevention recommendations for diseases.
- Data-backed fertilizer guidance based on crop nutrient standards.
- Separate result pages for each advisory module.

## Project Architecture

```
Browser (HTML/CSS/JS forms)
				|
				v
Flask Routes in app.py
	|        |         |
	|        |         +--> Disease Pipeline (PyTorch ResNet9)
	|        +------------> Fertilizer Rule Engine (CSV lookup + dictionary)
	+---------------------> Crop Recommendation (RandomForest pickle)
				|
				v
Rendered Jinja2 Templates
```

## Tech Stack

- Backend: Flask
- ML/DL: scikit-learn, PyTorch, torchvision
- Data processing: pandas, numpy
- Frontend: Jinja2 templates, Bootstrap, custom CSS/JS
- External API: OpenWeatherMap current weather API
- Deployment target (configured): Gunicorn

## Folder and Component Breakdown

### Core Python files

- `app.py`
  - Main Flask application.
  - Loads trained models at startup.
  - Defines routes for pages and predictions.
  - Contains weather fetch function and image preprocessing/prediction function.

- `model.py`
  - Defines the custom `ResNet9` architecture used by disease classification.

- `config.py`
  - Stores `weather_api_key` for OpenWeatherMap requests.

- `disease_dic.py`
  - Dictionary mapping model class labels to HTML-formatted disease information and suggested actions.

- `fertilizer_dic.py`
  - Dictionary mapping nutrient-status keys (`NHigh`, `Nlow`, etc.) to recommendation text.

### Data and model artifacts

- `Data/Final_Data_recomm.csv`
  - Crop recommendation dataset (features include N, P, K, temperature, humidity, pH, rainfall, label).

- `Data/FertilizerData.csv`
  - Crop nutrient baseline table used to compute nutrient deficits/excesses.

- `Trained_Model/RandomForest.pkl`
  - Pickled crop recommendation model loaded by the Flask app.

- `Trained_Model/plant_disease_model.pth`
  - Trained PyTorch weights for disease classifier.

- `Trained_Model/DecisionTree.pkl`, `Trained_Model/NBClassifier.pkl`, `Trained_Model/XGBoost.pkl`
  - Additional model artifacts present in repository but not currently used by routes in `app.py`.

### Frontend

- `templates/`
  - `layout.html`: shared layout with navbar/footer.
  - `index.html`: landing page and service navigation.
  - `crop.html`, `crop-result.html`: crop input/result pages.
  - `fertilizer.html`, `fertilizer-result.html`: fertilizer input/result pages.
  - `disease.html`, `disease-result.html`: image upload/result pages.
  - `try_again.html`: fallback page when weather lookup fails.
  - `login.html`, `signup.html`: basic UI pages (no active auth backend).
  - `test_components.html`: UI snippet/test template.

- `static/css/`
  - Bootstrap and project styles.

- `static/scripts/cities.js`
  - Populates state/city dropdowns used by crop input form.

- `static/images/`
  - UI images, logo, and background assets.

### Deployment/config files

- `requirements.txt`: Python dependencies.
- `Procfile`: Gunicorn startup command.
- `Runtime.txt`: Python runtime hint (`python-3.6.12`).
- `.gitignore`: excludes cache/editor artifacts.

## Request Flow

### Crop recommendation

1. User submits N, P, K, pH, rainfall, state, city.
2. App calls OpenWeatherMap with city name.
3. Temperature/humidity are extracted and combined with user soil inputs.
4. `RandomForest.pkl` predicts crop label.
5. Result rendered in `crop-result.html`.

### Fertilizer recommendation

1. User submits N, P, K and selects target crop.
2. App loads target crop nutrient values from `Data/FertilizerData.csv`.
3. App computes nutrient difference and dominant imbalance (N/P/K high/low).
4. Matching text from `fertilizer_dic.py` is rendered in `fertilizer-result.html`.

### Disease prediction

1. User uploads image in `disease.html`.
2. App transforms image (resize + tensor conversion).
3. `ResNet9` predicts one of the defined disease classes.
4. Prediction key is mapped to advisory text in `disease_dic.py`.
5. Output rendered in `disease-result.html`.

## Setup and Installation

### 1) Clone and move into project

```bash
git clone <your-repo-url>
cd agro.ai-main
```

### 2) Create and activate virtual environment

Windows (PowerShell):

```powershell
python -m venv .venv
.\.venv\Scripts\Activate.ps1
```

macOS/Linux:

```bash
python3 -m venv .venv
source .venv/bin/activate
```

### 3) Install dependencies

```bash
pip install -r requirements.txt
```

### 4) Configure weather API key

Open `config.py` and set:

```python
weather_api_key = "YOUR_OPENWEATHERMAP_API_KEY"
```

You can get a key from OpenWeatherMap.

## Running the Application

### Development

```bash
python app.py
```

App runs with Flask debug server and is typically available at:

`http://127.0.0.1:5000`

### Production-style (Gunicorn)

```bash
gunicorn app:app --log-level debug
```

Note: Gunicorn is typically run on Linux-based hosts.

## API/Route Reference

- `GET /`
  - Home page.

- `GET /crop-recommend`
  - Crop input form.

- `POST /crop-predict`
  - Predicts crop based on soil + weather.

- `GET /fertilizer`
  - Fertilizer input form.

- `POST /fertilizer-predict`
  - Returns fertilizer recommendation text.

- `GET /disease`
  - Disease upload form.

- `GET|POST /disease-predict`
  - On POST, predicts plant disease from uploaded image.

- `GET /signup`
  - Signup UI page.

- `GET /login`
  - Login UI page.

- `GET /test`
  - Simple JSON response used as a test endpoint.

## Model and Data Artifacts

### Disease model

- Architecture: custom `ResNet9` from `model.py`.
- Classes: 38 crop/disease labels (including healthy classes).
- Weights: `Trained_Model/plant_disease_model.pth`.
- Inference device: CPU (`map_location=torch.device('cpu')`).

### Crop model

- Model: Random Forest classifier loaded from pickle.
- Expected features order:
  - `[N, P, K, temperature, humidity, ph, rainfall]`
- Weights: `Trained_Model/RandomForest.pkl`.

### Fertilizer logic

- Not an ML model during runtime.
- Uses nutrient target values from `Data/FertilizerData.csv`.
- Computes difference between ideal and supplied N/P/K, then maps to recommendation dictionary.

## Deployment Notes

- `Procfile` already defines web process command.
- `Runtime.txt` pins an older Python runtime (`3.6.12`).
- For modern deployment, validate dependency compatibility on Python 3.10+ and consider upgrading Flask/PyTorch stack.

## Known Limitations

- Weather key is hardcoded in `config.py`; this is not ideal for security.
- Several templates have commented-out `{% extends 'layout.html' %}`, which can lead to inconsistent page layout behavior.
- `login` and `signup` pages are UI-only and do not implement authentication/storage.
- Minimal server-side validation for form numeric ranges and file type checks.
- Requirement list is very broad and includes duplicate/legacy entries (for example `gunicorn` appears twice).

## Troubleshooting

- `FileNotFoundError` for models:
  - Ensure all files in `Trained_Model/` exist, especially `RandomForest.pkl` and `plant_disease_model.pth`.

- Weather lookup fails and shows retry page:
  - Verify API key in `config.py`.
  - Check city string validity and network access.

- Torch/torchvision install issues:
  - Install version-compatible binaries for your OS/Python version from official PyTorch instructions.

- App fails on package conflicts:
  - Create a fresh virtual environment and reinstall dependencies.

## Future Improvements

1. Move API keys/secrets to environment variables.
2. Add robust input validation and file upload constraints.
3. Add automated tests for route responses and model pipeline behavior.
4. Add model versioning and explicit metadata (training data/version/metrics).
5. Introduce user authentication and persistence for advisory history.
6. Clean and modernize dependency pinning.

---

If you want, I can also generate a second version of this README with badges, screenshots section, and a developer-focused contribution guide.
