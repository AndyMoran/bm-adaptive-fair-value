# Walk‑Forward Adaptive Fair‑Value Modelling for GB Balancing Spreads

---

# Project Overview

This project investigates whether the GB electricity Balancing Mechanism (BM) exhibits persistent pricing disequilibrium or whether balancing spreads rapidly adapt to changing operational conditions.

Using publicly available BMRS data from Elexon, the project builds a reproducible engineering and modelling framework for analysing:

- balancing spreads,
- balancing actions,
- market-state dynamics,
- volatility clustering,
- and conditional fair value.

In this project, **fair value** refers to:

> the model-estimated expected balancing spread conditional on observed market-state variables.

The original hypothesis was that balancing spreads might exhibit persistent inefficiency or sustained stress premiums during periods of system stress.

However, the project ultimately finds that adaptive walk-forward retraining diminished but did not eliminate the residual stress structure identified by static fair-value models

This suggests that balancing-market behaviour reflects a combination of:
- evolving equilibrium conditions,
and:
- genuinely persistent operational stress during extreme market periods.

The evidence therefore supports a hybrid interpretation in which:
- equilibrium adapts dynamically through time,
while:
- substantial stress persistence remains visible during severe market conditions.
  
---

# Core Research Question

> Does the GB Balancing Mechanism exhibit persistent pricing inefficiency, or does fair value adapt dynamically during changing operational conditions?

---

# Key Quantitative Findings

| Metric | Static Model (NB05) | Walk-Forward Model (NB06) |
|---|---|---|
| MAE | ~1.4 | ~2.9 |
| Evaluation structure | Single terminal holdout | Sequential adaptive retraining |
| Evaluation rigour | Moderate | High |
| Residual stress during late-2022 | Prominent clustering | Similar prominent clustering |
| Residual behaviour | Persistent stress clustering during late-2022 | Modestly stabilised but still persistent |
| Equilibrium assumption | Static | Dynamically evolving |

**Interpretation note**

The higher walk-forward MAE reflects:
- sequential out-of-sample evaluation across evolving market conditions,
rather than:
- evaluation against a single terminal holdout window.

The static model was evaluated against a single late-2022
holdout regime, allowing it to fit the crisis stress period
more closely.

By contrast, the walk-forward framework was evaluated
sequentially across evolving market conditions and therefore
represents a more stringent and realistic test of equilibrium
adaptation through time.

**Key interpretation**

Adaptive retraining partially improved equilibrium estimation
through time, but substantial residual stress structure
persisted during the late-2022 energy crisis.

The evidence therefore suggests that balancing-market stress
reflected both:
- evolving equilibrium conditions,
and:
- genuinely difficult operational stress regimes.

---

# Main Findings

## 1. Balancing spreads exhibit strong structure

The BM does not behave randomly.

Observed structure includes:
- autocorrelation,
- volatility clustering,
- stress regimes,
- and persistent temporal dependence.

Balancing behaviour appears highly state-dependent rather than purely noisy.

---

## 2. Operational balancing activity is highly informative

Feature importance analysis consistently showed that balancing activity variables:
- total balancing MW,
- balancing action intensity,
- and intervention frequency

contain substantial explanatory power for balancing spreads.

Example high-importance features included:
- `total_action_mw`
- `action_count`
- `spread_volatility_12`
- `spread_lag_1`
- `spread_rolling_mean_4`

Important caveat:

Some operational activity variables are contemporaneous with the target spread and therefore partially describe price formation itself rather than purely forward-looking prediction.

This distinction is explicitly acknowledged throughout the modelling process.

---

## 3. Static fair-value models initially suggested persistent disequilibrium

A static XGBoost framework appeared to identify:
- sustained stress periods,
- widening residual spreads,
- and possible persistent disequilibrium during late-year stress conditions.

At first glance, this suggested the existence of:
- flexibility scarcity premiums,
- or persistent balancing inefficiency.

This stage intentionally served as a methodological “false positive” test.

---

## 4. Walk-forward retraining partially stabilised equilibrium estimation

The evaluation period coincided with the late-2022 European
energy crisis, characterised by exceptional gas-market and
power-market volatility.

This broader macro-energy shock likely contributed to the
persistent residual stress that even the adaptive framework
could not fully eliminate.

The project’s central methodological test emerged in Notebook 06.

Once the model was retrained adaptively using walk-forward equilibrium estimation:
- some residual instability was reduced,
- equilibrium estimates adapted more smoothly through time,
but:
- substantial residual stress structure persisted during the late-2022 stress regime.

Walk-forward structure:
- expanding historical training window,
- retrained every ~7 days,
- strictly time-ordered validation,
- no look-ahead leakage.

This suggests that:
- part of the apparent disequilibrium identified by static models reflected evolving equilibrium conditions,
while:
- part reflected genuinely persistent operational stress that remained difficult to model adaptively.

The evidence therefore supports a hybrid interpretation combining:
- adaptive equilibrium dynamics,
with:
- residual operational stress persistence during extreme market conditions.

---

## 5. The BM appears operationally adaptive

The combined evidence suggests that:
- balancing spreads reprice rapidly,
- operational equilibrium evolves continuously,
- and NESO balancing processes appear broadly effective at restoring conditional equilibrium during changing system conditions.

Persistent structural disequilibrium does not appear continuously present at system-wide level, but episodic residual stress remains visible during extreme market conditions.

---

# Practitioner Takeaways

## For market analysts and traders

- Static fair-value models may overestimate the persistence of balancing stress.
- Apparent scarcity premiums may reflect rapid repricing rather than persistent structural imbalance.
- Real-time equilibrium estimation is likely more informative than static spread assumptions.
- Extreme balancing stress appears episodic and operationally driven rather than continuously persistent.

---

# Engineering Pipeline

A major component of the project involved building a robust and reproducible data engineering framework.

Key features include:

- incremental BMRS API downloading,
- retry logic and resilient sessions,
- parquet persistence,
- deduplication,
- resumable ingestion,
- corruption handling,
- and reproducible directory structures.

The project evolved substantially from:
- exploratory notebook analysis,
toward:
- a production-style research workflow.

---

# Dataset Architecture

| Dataset | Description |
|---|---|
| BOA | Balancing acceptance prices |
| BOALF | Balancing acceptance volumes and actions |
| BMRS system data | System pricing and balancing metrics |
| Wind generation | GB wind output |
| Regional demand | GSP-group demand proxies |
| Engineered features | Lags, volatility, ramps, stress indicators |

Primary storage format:
- Parquet

Primary tools:
- Python
- pandas
- scikit-learn
- XGBoost

---

# Notebook Structure

## Notebook 01 — Data Build

### Objective
Construct the canonical balancing dataset.

### Main tasks
- ingest BOA and BOALF data,
- merge balancing actions and prices,
- engineer balancing variables,
- create settlement-period timestamps,
- aggregate balancing activity.

### Outputs
- `bm_actions.parquet`
- `bm_sp_aggregated.parquet`
- processed modelling datasets.

---

## Notebook 02 — Exploratory Analysis

### Objective
Explore balancing spread behaviour and volatility.

### Main findings
- balancing spreads are highly non-normal,
- large outliers exist,
- volatility clusters through time,
- high-wind periods compress balancing spreads.

---

## Notebook 03 — Feature Engineering

### Objective
Build an enriched market-state dataset.

### Main tasks
- lag construction,
- rolling volatility features,
- wind and demand ramp variables,
- balancing stress metrics.

### Key engineered variables
- rolling spread volatility,
- lagged spreads,
- balancing activity intensity,
- stress-state indicators.

---

## Notebook 04 — Continuous Market State

### Objective
Investigate temporal market-state dynamics.

### Main findings
- balancing spreads exhibit strong autocorrelation,
- stress periods cluster temporally,
- volatility persistence is substantial,
- balancing conditions are strongly state-dependent.

---

## Notebook 05 — Static Fair-Value Estimation

### Objective
Estimate balancing fair value using static XGBoost models.

### Methodology
- time-ordered train/test split,
- XGBoost regression,
- residual analysis.

### Initial findings
- apparent persistent disequilibrium,
- widening late-year residuals,
- and possible stress-premium behaviour.

This notebook intentionally serves as the “false positive” stage of the research process.

---

## Notebook 06 — Walk-Forward Adaptive Equilibrium

### Objective
Test whether balancing equilibrium evolves dynamically through time.

### Methodology
- walk-forward retraining,
- adaptive fair-value estimation,
- rolling equilibrium updating.

### Main finding

Walk-forward retraining partially stabilised equilibrium
estimation, but substantial residual stress structure
persisted during the late-2022 stress regime.

### Interpretation

The adaptive framework demonstrated that part of the apparent
disequilibrium identified by static models reflected evolving
equilibrium conditions.

However, persistent residual stress during late 2022 suggests
that genuine operational stress regimes also remained difficult
to fully capture using adaptive market-state modelling alone.

A residual-volatility comparison chart showed that while
adaptive retraining dampened some residual expansion, the
late-2022 stress cluster remained visibly elevated relative
to calmer market periods.

---

# Methodological Lessons

One of the project’s most important lessons is methodological rather than purely predictive.

Static models can easily mistake:
- evolving equilibrium conditions,
for:
- persistent structural inefficiency.

Walk-forward retraining demonstrated that:
- non-stationarity matters critically in energy markets,
- and adaptive equilibrium estimation can radically alter interpretation.

The project therefore highlights the importance of:
- temporal validation,
- adaptive retraining,
- and careful treatment of contemporaneous operational features.

---

# Limitations

Several important limitations remain:

- BM operational activity may partially encode contemporaneous pricing behaviour.
- The project primarily analyses system-level equilibrium rather than true nodal power-flow behaviour.
- Transmission topology and physical network flows are not explicitly modelled.
- Some locational stress events appear sparse and network-driven rather than easily captured using generic market-state variables.
- The evaluation period includes the late-2022 European energy crisis, which introduced unusually elevated gas-market and power-market volatility.

These limitations strongly influenced the project’s final interpretation.

---

# Future Research

A future extension project may investigate:
- regional constraint persistence,
- infrastructure delivery risk,
- and locational balancing stress in the GB North-West system.

This future work would likely require:
- transmission-flow data,
- network topology information,
- and more explicit physical-system modelling.

It is intentionally treated as a separate future research direction rather than part of the current completed study.

---

# Final Conclusion

The GB Balancing Mechanism is neither perfectly efficient nor
structurally mispriced.

Instead, it appears to operate as a highly adaptive system
that reprices rapidly under normal conditions, yet remains
subject to episodic operational stress that becomes difficult
to fully capture during extreme energy-market crises.
