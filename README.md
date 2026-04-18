# 🛰️ Predictive Modeling of Li-ion Battery Health and Lifetime for Reliable CubeSat Power Systems

> A Data Mining project for predicting battery degradation in nanosatellites,
> Submitted by **Algerian Space Agency, Satellite Development Center**.

---

## 🔭 Motivation

Reliable power is mission-critical for CubeSats. Li-ion batteries degrade with every
charge-discharge cycle, and an unexpected failure mid-orbit can end a mission entirely.
This project builds predictive models that estimate **how healthy a battery is right now**
(State of Health) and **how many cycles it has left** (Remaining Useful Life) — directly
from onboard sensor data, without needing to measure capacity explicitly.

The goal: give mission operators early, accurate warnings so they can make informed
decisions before it's too late.

---

## 📦 Dataset

**NASA Ames Prognostics Battery Dataset** — multiple groups of Li-ion cells cycled under
various conditions (charge, discharge, impedance measurements). The dataset contains
batteries tested across different temperatures (4°C, 24°C, 43°C, 44°C) with varying
discharge profiles and End-of-Life criteria.

🔗 [Available on Kaggle](https://www.kaggle.com/datasets/patrickfleith/nasa-battery-dataset/data)

**The notebook downloads the dataset automatically** via the Kaggle API — no manual
download needed.

### Battery selection

Not all batteries in the dataset are suitable for modeling. The notebook runs a
data-driven selection pipeline that filters on:

- Sufficient number of discharge cycles
- Complete, non-missing capacity data
- Reaching or approaching End-of-Life (SOH ≤ 70%)
- Consistent measurement quality

Batteries that fail these criteria (e.g., incomplete experiments, software-crash
terminations, or unknown EOL status) are automatically excluded. The final selected
batteries are split as follows:

| Split           | Batteries    |
| --------------- | ------------ |
| Training        | B0005, B0006 |
| Validation      | B0007        |
| Test (held out) | B0018        |

Splitting at the battery level — not the cycle level — prevents temporal data leakage
and gives a realistic measure of how the models generalize to completely unseen batteries.

---

## ⚙️ Approach

The project follows the **CRISP-DM** methodology across six phases:

**1. Business Understanding** — Define SOH and RUL prediction targets with R² > 0.85
success criteria for mission-critical deployment.

**2. Data Understanding** — Explore all available batteries, analyze cycle counts,
capacity distributions, and identify data quality issues across battery groups.

**3. Data Preparation** — Extract 26 engineered features per cycle from raw time-series
voltage, current, and temperature measurements:

| Category               | Features                                                           |
| ---------------------- | ------------------------------------------------------------------ |
| Voltage statistics     | mean, std, min, max, drop, median, variance                        |
| Current statistics     | mean, std, max                                                     |
| Temperature statistics | mean, std, max, min, range                                         |
| Energy & time          | discharge time, integrated energy                                  |
| Degradation indicators | capacity fade rate, cumulative loss, voltage drift, discharge rate |
| Rolling statistics     | 5-cycle rolling means and standard deviations                      |

Raw capacity is deliberately excluded as a feature — the models are designed to work
from partial discharge cycles, making them deployable onboard without ground-truth
capacity measurements.

**4. Modeling** — Four regression algorithms trained and compared for both SOH and RUL:
Linear Regression · Random Forest · Gradient Boosting · XGBoost

**5. Evaluation** — Two complementary validation strategies:

- **Hold-out test set**: Battery B0018 provides an unbiased final estimate
- **Leave-One-Group-Out (LOGO) cross-validation**: Each training battery held out once
  to assess cross-battery generalization — the most realistic proxy for deployment

**6. Deployment** — Best models serialized with `joblib` into a structured
`models/deployment/` directory, ready for integration into ground systems or onboard
hardware.

---

## 📊 Results

Full performance tables, visualizations, and cross-validation breakdowns are documented
in [`Project_Report.pdf`](./Project_Report.pdf) and [`Presentation.pdf`](./Presentation.pdf).

---

## 👥 Team

Sayah Maroua · Zamiche Nour · Ameddah Mohamed · Gouicem Islem

---
