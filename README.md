# Stochastic Interest Rate Modelling and Prediction
## Cox-Ingersoll-Ross (CIR) Model — Finance Club IIT Roorkee Open Projects 2026

---

## Project Overview

This project implements, calibrates, and extends the **Cox-Ingersoll-Ross (CIR)** stochastic short-rate model on real historical zero-coupon bond yield data spanning 9 maturity tenors (3M to 30Y). The core challenge is to reconstruct the entire yield curve using **only the 3-Month yield** as input on each test day.

---

## Results Summary

| Model | Out-of-Sample R² | RMSE | Status |
|-------|-----------------|------|--------|
| **Base CIR** | **0.8922** | 22.05 bps |  PASS (≥ 0.85) |
| CIR++ Extension | 0.8342 | 27.34 bps |  Implemented & Analysed |

### Per-Maturity Performance (Base CIR)

| Maturity | R² | RMSE (bps) |
|----------|-----|------------|
| 6M | 0.9944 | 5.91 |
| 9M | 0.9672 | 13.07 |
| 1Y | 0.9094 | 19.81 |
| 2Y | 0.3845 | 36.69 |

---

## Notebook
 
Open Colab Link: [Click Here](https://colab.research.google.com/drive/1DAbrPikzw272IELtJZ1jlzH-pcQFQ2-g?usp=sharing)
---


## Repository Structure

```text
Finance_Club_Open_Project/
├── README.md                           # Project documentation
└── finance-club-23115018-anurag-sain.ipynb   # Complete CIR Interest Rate Modelling notebook

```

---

## Dataset

| File | Description |
|------|-------------|
| `train_data.csv` | Daily yields 2016–2024, all 9 maturities |
| `test_data.csv` | Daily yields 2024–2026, maturities 3M–2Y |
| `test_data_3M.csv` | Test input — 3M yield only (prediction input) |

**Maturities:** 3M, 6M, 9M, 1Y, 2Y, 5Y, 10Y, 20Y, 30Y

---

## Methodology

### 1. Data Engineering
- Parsed and cleaned daily time-series yield data
- Linear time interpolation for missing values
- Rolling z-score outlier detection and replacement (window=60, threshold=4σ)

### 2. CIR Model — Mathematical Framework

The instantaneous short rate $r_t$ follows:

$$dr_t = \kappa(\theta - r_t)\,dt + \sigma\sqrt{r_t}\,dW_t$$

| Parameter | Symbol | Value |
|-----------|--------|-------|
| Mean reversion speed | κ | Calibrated via MLE |
| Long-run mean | θ | Calibrated via MLE |
| Volatility | σ | Calibrated via MLE |

**Zero-coupon bond price:**

$$P(t,T) = A(\tau)\,e^{-B(\tau)\,r_t}$$

**Continuously compounded yield:**

$$y(\tau) = \frac{B(\tau)\,r_t - \ln A(\tau)}{\tau}$$

### 3. Calibration — Maximum Likelihood Estimation (MLE)

- **Method:** MLE with L-BFGS-B optimiser, 6 diverse random restarts
- **Panel:** Subsampled training data (every 10th day) for computational efficiency
- **Noise model:** Gaussian observation error $\varepsilon \sim \mathcal{N}(0, \eta^2)$
- **Why MLE over OLS:** Provides asymptotically efficient estimates, principled model comparison via AIC/BIC, and naturally handles the nonlinear CIR yield mapping

### 4. Prediction Challenge

- **Input allowed:** Only the 3M yield on each test day
- **Output:** Full yield curve for 6M, 9M, 1Y, 2Y
- **Method:** Plug 3M rate as short-rate proxy $r_t$ into calibrated CIR yield formula

### 5. Extension — CIR++ (Brigo & Mercurio, 2001)

The CIR++ model augments the short rate with a deterministic shift:

$$r_t^{++} = x_t + \phi(t)$$

where $\phi(\tau)$ corrects the systematic bias of the base CIR:

$$\phi(\tau) = \mathbb{E}_t\left[y_{\text{actual}}(\tau) - y_{\text{CIR}}(r_t, \tau)\right]$$

A cubic spline is fitted through the shift nodes for smooth interpolation across maturities.

**Finding:** CIR++ underperformed the base model on the test set (R²=0.8342 vs 0.8922) because the static shift φ(τ) — estimated from 2016–2024 training data — overcorrects in the high-rate regime of 2024–2026. The base CIR anchors directly to the observed 3M rate each day, making it naturally adaptive to the current rate level.

---

## Key Findings

-  Base CIR achieves **R² = 0.8922** — exceeds the 0.85 verification threshold
-  Short maturities (6M–1Y) predicted with **R² > 0.90** — strong single-factor fit
-  2Y maturity is harder (R²=0.38) — sits at the boundary between policy rate and term premium, requiring a second factor to capture independently
-  Shock half-life = ln(2)/κ — implies persistent rate dynamics consistent with multi-year monetary policy cycles
-  CIR++ static shift fails under regime change — limitation of deterministic extensions

---

## Calibrated Parameters

| Parameter | Value | Interpretation |
|-----------|-------|----------------|
| κ | ~0.17 | Slow mean reversion — shocks persist ~4 years |
| θ | ~2.72% | Long-run equilibrium rate |
| σ | ~0.10 | Moderate volatility scaling |

---

## Limitations

| Limitation | Implication |
|------------|-------------|
| Single-factor model | Cannot capture independent slope/curvature moves |
| 3M proxy for $r_t$ | Introduces measurement error — biases σ upward |
| Static CIR++ shift | Fails during structural regime changes |
| Feller condition | May be violated in low-rate, high-volatility regimes |

---

*Finance Club, IIT Roorkee — Open Projects 2026*
