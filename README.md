# Eco-Predict — Cambodia Personal Carbon Emission Tracker

## Link: https://carbon-emissions-prediction.streamlit.app

**Mini Project | Team Work**

| | Member |
|---|---|
| 1 | TAN Vouchleang |
| 2 | SOUN Chey Nutt |
| 3 | SEK Bonditkolyaney |
| 4 | THOL Thida |
| 5 | VEI Saata |

---

A Streamlit web application that predicts daily personal carbon emissions based on electricity usage, transportation, and plastic consumption. Built using a Multiple Linear Regression model trained on IoT sensor data and validated against primary survey data collected from Cambodian respondents.

---

## Files Required

| File | Description |
|---|---|
| `app.py` | Main Streamlit application |
| `carbon_predictor_model.pkl` | IoT trained linear regression model |
| `carbon_scaler.pkl` | StandardScaler fitted on IoT training data |
| `carbon_calibrator.pkl` | Calibration layer fitted on primary survey data |
| `primary_data_CO2.csv` | Primary survey data (used for comparison chart) |

All files must be in the same folder.

---

## Installation

```bash
pip install streamlit pandas numpy scikit-learn joblib matplotlib
```

## Running the App

```bash
streamlit run app.py
```

---

## How It Works

The prediction uses a three-step pipeline that mirrors the Local Validation notebook exactly:

```
User inputs → IoT Scaler → IoT Model → Calibrator → Prediction
```

**Step 1: IoT Scaler** (`carbon_scaler.pkl`)
Normalizes the three raw input features (kWh, km, kg) using StandardScaler fitted on the IoT training dataset. This ensures inputs are in the same range the model was trained on.

**Step 2: IoT Model** (`carbon_predictor_model.pkl`)
A Multiple Linear Regression model trained on 10,000 IoT sensor records. Learned emission factors (unscaled):
- Energy: **0.4969 kgCO₂/kWh**
- Transport: **0.1997 kgCO₂/km**
- Plastic: **3.5413 kgCO₂/kg**
- Intercept: **0.3160 kgCO₂**

**Step 3: Calibrator** (`carbon_calibrator.pkl`)
A secondary linear correction that aligns IoT model outputs with primary survey data:
- Scale factor: **0.3995**
- Intercept: **-0.0584**

The final prediction is: `CO₂ = 0.3995 × (IoT prediction) - 0.0584`

---

## Model Performance

| Metric | IoT Test Set | Primary Survey Data (after calibration) |
|---|---|---|
| R² Score | 0.8537 | 0.9368 |
| MAE | 2.96 kg CO₂ | 0.43 kg CO₂ |

---

## Input Features

| Feature | Unit | How it is collected |
|---|---|---|
| Energy Usage | kWh/day | Converted from monthly electricity bill (Riel) using EDC rate of 610 Riel/kWh, then divided by 30 days |
| Transportation Distance | km/day | Direct user input |
| Plastic Usage | kg/day | Number of single-use plastic items × 0.141 kg/item |

### Why 0.141 kg per plastic item?
This value was derived from the primary survey data, where all plastic values are exact multiples of 0.141 — confirming it was the conversion factor used during data collection.

---

## Emission Constants (Display Only)

These constants are used only for the breakdown cards in the app. The final prediction always comes from the ML pipeline above.

| Constant | Value | Source |
|---|---|---|
| Grid emission factor | 0.18708 kgCO₂/kWh | Primary survey data |
| Motorcycle (petrol) | 0.11367 kgCO₂/km | Primary survey data |
| Plastic lifecycle | 1.0 kgCO₂/kg | Primary survey data |
| EDC tariff | 610 Riel/kWh | Electricité Du Cambodge |

---

## Zero-Input Behaviour

If all inputs are zero, the app returns **0.50 kg CO₂/day** as a human metabolic baseline. This is not from the model it represents the minimum unavoidable daily footprint from metabolism and indirect consumption. Human breathing CO₂ is carbon-neutral (part of the natural cycle) so this baseline does not represent harmful emissions.

---

## Breakdown Card Logic

The three breakdown cards (Electricity, Transport, Plastic) show CO₂ values that are proportionally split from the ML prediction:

```
energy_share    = (energy_co2 / total_co2) × prediction
transport_share = (transport_co2 / total_co2) × prediction
plastic_share   = (plastic_co2 / total_co2) × prediction
```

This ensures the three cards always sum exactly to the prediction displayed at the top.

---

## Project Context

Built in response to **SDG Goal 13.3** to raise awareness of personal carbon emissions among Cambodian communities. The model bridges the gap between international IoT datasets and local Cambodian lifestyle data through a two-stage training and calibration approach.
