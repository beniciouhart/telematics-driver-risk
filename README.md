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

## Results

### Segmentation

K-Means with k=3 produced the best silhouette score and is used as the final segmentation model.
The three risk tiers are well-separated in PCA space with distinct behavioural profiles:

- **Risk 1 (Low):** consistently low harsh-event rates, moderate speeds, low-risk driving patterns
- **Risk 2 (Medium):** intermediate profile across all telematics features
- **Risk 3 (High):** materially elevated per-km harsh braking, acceleration, and cornering rates; 95th-percentile speeds significantly higher than other tiers

### Loss Cost Model

A Gamma GLM (log link, IRLS) was fit on 420 policy observations with driver risk tier as a
categorical factor alongside policy and vehicle covariates. Pseudo R-squared (Cox-Snell) of 0.374.

Key findings (Recall that the impact over the baseline is measured as exp(coef)):

- **Risk tier is statistically significant.** Risk_2 and Risk_3 drivers carry 12.7% and 16.9%
  higher predicted claim amounts than Risk_1 (coef = 0.120, p < 0.001 and coef = 0.157,
  p = 0.019 respectively).
- **Driver age** is positively associated with claim amount (~ 1.1%) (coef = 0.011, p < 0.001)
- **Pay-as-you-drive policies** associated with 26.4% higher predicted losses
  (coef = 0.235, p = 0.020)
- **Retired usage** associated with 18.5% lower predicted losses vs. baseline
  (coef = -0.205, p = 0.010)
- **Annual payment frequency** associated with lower predicted losses (~ 9.6%) vs. monthly
  (coef = -0.101, p = 0.017)
- **Policy duration** negatively associated with claim amount (coef = -0.005, p = 0.012),
  suggesting longer-tenured policyholders are lower risk
- Vehicle value and weight are both positively associated with claim amounts

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