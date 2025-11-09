# Implied-Volatiliy-Models-via-CNN
Apply differnent models to find implied volatility, deep calibration via CNN
Learning to Reprice Option Smiles with Heston, Bates, and Kou (via COS)

I trained a compact CNN (“surface calibrator”) that takes a grid of total variances and outputs model parameters, then reprices the full implied-volatility (IV) surface using a fast, stable Fourier–COS pricer. The same pipeline works across three classic models: Heston, Bates, and Kou.

## What I built

# A clean learning-to-reprice framework: CNN → parameters → COS → full surface, in milliseconds.

# The same COS pricing engine for generating labels and for evaluation, eliminating engine drift.

# A consistent diagnostics workflow (plots + metrics) to sanity-check: true smiles, label-repriced smiles, and net-repriced smiles.

## Quick introductions to the models

# Heston — stochastic variance with mean reversion and stock/vol correlation; good at equity skew with a single vol-of-vol control.
Parameters: kappa, theta, sigma, rho, v0.

# Bates — Heston plus Merton jumps; adds jump intensity/size/dispersion to improve short-maturity wings.
Parameters: Heston + lambda, muJ, sigJ.

# Kou — GBM with double-exponential jumps (asymmetric up/down tails), often matching equity left-skew well.
Parameters: sigma, lambda, p, eta1, eta2 (with eta1 > 1).

All three are priced with the COS method (Fourier–cosine expansion) and share the same clamped IV inversion to prevent smile plateaus from pathological prices.

## Two simple tricks that made it work

# COS everywhere
I price calls from each model’s characteristic function with the COS method. It’s spectral, vectorizable, and very fast. Using the same COS setup (truncation L, number of modes N) for labels and for repricing keeps numerics stable across maturities and moneyness and removes hidden biases.

# Forward moneyness + clamped IV inversion
I price on forward-moneyness m = K / F_T so smiles align across different S, r, and q. Before inverting price → IV, I clamp prices to the no-arbitrage band:

Lower bound: max(0, S*exp(-q*T) - K*exp(-r*T))

Upper bound: S*exp(-q*T)
This prevents spurious IV spikes and makes “label vs net” plots overlap.

## Training choices that stabilized learning

# Z-scored targets + constrained decoding
The network predicts z-scores; at inference I un-z and map to valid domains (softplus for theta, sigma, v0; tanh for rho; clamps for jump parameters).

# Per-sample market inputs
Repricing uses each sample’s own S, r, q, so one calibrator works across term structures and dividend policies.

# Diagnostics first
For a fixed sample I always plot: (1) true smiles; (2) true vs label-repriced (sanity); (3) true vs net-repriced. I also report per-tenor RMSE and ATM bias to see where misspecification shows up (e.g., short-T wings).

## What the results say

# On synthetic data, label vs repriced smiles are nearly identical—my COS pricer and IV inversion match the dataset.

# Net vs true errors are small but structured: Heston can be too linear in the wings; Bates improves short-T left tail; Kou often matches asymmetric skew better.

# With yfinance backtests on SPY (target DTE), in-sample leaders vary by day, and Kou frequently helps out-of-sample on the same maturity because of its asymmetric jumps. I report both plain RMSE and vega-weighted RMSE to make the comparison more economic.

## Why this matters (my contributions)

# A reproducible CNN + COS pipeline that turns IV surfaces into parameters and back into prices/IV fast and consistently.

# Practical guardrails that make academic models behave on real quotes: forward-moneyness grids, no-arb clamping, z-scored training with constrained outputs, and consistent label vs net checks.

# A lightweight market backtest loop (yfinance) that compares models in-sample and out-of-sample using RMSE and vega-weighted RMSE.

## What I’d do next

# Add a mixture/ensemble head to blend Heston/Bates/Kou by tenor.

# Train with vega-weighted or bid/ask-aware losses.

# Enforce term-structure consistency by jointly fitting multiple DTEs with shared parameters.

# Run a rolling OOS evaluation to track regime-dependent model rankings.
