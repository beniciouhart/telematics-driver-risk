# Telematics Driver Risk Segmentation

Behavioural driver segmentation from raw OBD telematics data, built for the 2026 McMaster × Co-operators Case Competition.

Drivers are clustered into risk tiers using K-Medoids and K-Means, evaluated by silhouette score. The resulting classification is fed into a Gamma GLM to generate loss cost predictions on held-out policy data.

---

## Pipeline

1. **Cleaning** — resolve sampling irregularities, drop idle trips, nullify intra-trip gaps over 5s
2. **Feature engineering** — harsh braking, acceleration, and cornering rates normalized by distance; speed percentiles, RPM ratios, night driving, stop frequency
3. **Segmentation** — K-Means and K-Medoids evaluated for k=2–6; best method selected by silhouette score; clusters ranked by composite risk score
4. **Loss cost model** — Gamma GLM (log link) with driver risk tier as an additional categorical factor; predictions exported for submission

---

## Repo Structure

```
telematics-driver-risk/
├── telematics_driver_risk_segmentation.ipynb
├── requirements.txt
├── README.md
└── .gitignore
```

Data is not included. The notebook expects a `/data` directory at the repo root containing `Telematics.csv`, `LC_train.csv`, and `LC_test.csv`.

---

## Setup

```bash
git clone https://github.com/beniciouhart/telematics-driver-risk.git
cd telematics-driver-risk

python -m venv venv
source venv/bin/activate  # Windows: venv\Scripts\activate

pip install -r requirements.txt
jupyter lab
```

Then open `telematics_driver_risk_segmentation.ipynb` and run all cells.

---

## Dependencies

See `requirements.txt`. Key packages: `scikit-learn`, `scikit-learn-extra` (for K-Medoids), `statsmodels`, `pandas`, `seaborn`.