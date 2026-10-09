# AI Rental House Decision Maker

Answers one question: **"Is this house actually suitable for me?"**

Given rent, salary, distance to work/college, electricity cost, water
availability, number of people, area safety rating, and transport
availability, it outputs:

```
Suitability: 87%
Monthly financial burden: LOW
Travel burden: MEDIUM
Decision: Suitable ✅
```

## How it works

Real labeled data ("was this house actually suitable, yes/no") doesn't exist
publicly, so this project builds its own:

1. **`data_generator.py`** — generates 8,000 realistic synthetic households
   (rent correlated with salary and people-count, distance following a
   realistic gamma distribution, etc.) and labels each one using a
   domain-knowledge rule engine (rent-to-income ratio thresholds, commute
   distance adjusted for transport quality, safety overrides) **plus random
   noise**, so the labels resemble messy real human judgment rather than a
   clean formula. Output: `data/rental_dataset.csv`.

2. **`train_models.py`** — trains 4 Random Forest models on that dataset:
   - `decision_classifier` — Suitable / Not Suitable (92% test accuracy)
   - `suitability_regressor` — the 0–100% score (R² = 0.89, ~4.3 pt avg error)
   - `financial_burden_classifier` — LOW / MEDIUM / HIGH
   - `travel_burden_classifier` — LOW / MEDIUM / HIGH

   Models are saved to `models/*.joblib`. Metrics are in `models/metrics.json`.

3. **`predict.py`** — the interface you actually use. Takes the 8 raw inputs
   and returns the formatted report.

## Quick start

```bash
pip install pandas numpy scikit-learn joblib

# 1. (optional — already done, dataset is included) regenerate the data
python3 data_generator.py

# 2. (optional — already done, models are included) retrain
python3 train_models.py

# 3. Use it
python3 predict.py
```

Or import it directly:

```python
from predict import evaluate_house, print_report

result = evaluate_house(
    rent=15000,
    salary=60000,
    distance_km=8.5,
    electricity_cost=1200,
    water_availability="24x7",       # or 0/1/2
    num_people=2,
    safety_rating=7,                 # 1 (unsafe) - 10 (very safe)
    transport_availability="good"    # or 0-4 (none/poor/average/good/excellent)
)
print_report(result)
```

## Replacing synthetic data with real data

If you (or a group of friends/classmates) actually collect real answers —
e.g. a Google Form asking these same 8 questions plus "Were you happy with
this house? Yes/No" — you can drop that CSV in as `data/rental_dataset.csv`
(matching column names) and rerun `train_models.py`. The model quality will
only improve with real labels; the synthetic data is a solid stand-in until
then.

## Tuning the rules

All the domain assumptions (30%/45% rent-to-income thresholds, 6km/15km
commute thresholds, safety overrides, feature weights) live in
`data_generator.py`'s `label_dataset()` function in one place — adjust them
to match your local cost of living / commute norms and regenerate.
