# Verde

**Open-source research into carbon- and cost-aware AI workload scheduling.**

Verde investigates whether lightweight time-series forecasting can help schedule flexible AI workloads for lower-cost and lower-carbon execution—while still meeting workload deadlines.

> AI workloads do not always need to run right now. If a workload is flexible, can we intelligently move it to a better time?

## Status

Verde is in the research and early-development phase. The architecture, datasets, models, and scheduling strategies are still being evaluated and may change as experiments provide evidence.

No environmental or financial savings claims have been established. Where direct measurements are unavailable, results will be identified explicitly as estimates or simulations.

The first half of the data-foundation phase establishes the initial scope and contracts: CAISO/California, hourly data, canonical timezone `America/Los_Angeles`, and an intended initial study period of 2020–2024. Data source specifications, the canonical schema, and provenance requirements are documented in [`docs/`](docs/).

## Research question

> Can a lightweight time-series forecasting model enable intelligent scheduling of flexible AI workloads to reduce electricity cost and carbon emissions while maintaining workload deadlines?

The project focuses on the relationship between:

- forecast accuracy and scheduling outcomes;
- model complexity and computational cost;
- electricity price and carbon intensity;
- workload flexibility and deadlines; and
- measured, estimated, and simulated environmental impact.

## Hypotheses

1. Machine-learning time-series models can improve on simple forecasting baselines for relevant grid conditions.
2. Forecast-based scheduling can reduce estimated cost and/or carbon emissions compared with immediate execution while meeting deadlines.
3. A lightweight model can provide most of the practical scheduling benefit of a substantially more expensive model.

The most accurate model will not automatically be considered the best model. Verde will evaluate predictive performance alongside parameter count, model size, latency, memory use, training cost, and energy consumption where measurable.

## How it works

```text
Grid, weather, and market data
              │
              ▼
       Feature engineering
              │
              ▼
       Time-series forecasts
              │
              ▼
           Scheduler
              │
       ┌──────┴──────┐
       ▼             ▼
    Run now       Run later
```

The forecasting layer estimates future conditions such as carbon intensity, electricity price, and renewable availability. The scheduling layer uses those forecasts together with workload duration, flexibility, and deadline constraints to select an execution window.

The initial forecasting horizon is expected to be approximately 24 hours, subject to data availability and experimental results.

## Planned evaluation

### Forecasting

Verde will compare simple baselines and progressively more complex models, potentially including:

- persistence;
- historical averages;
- classical statistical models;
- small feed-forward networks;
- LSTMs; and
- temporal fusion transformers or other lightweight architectures.

Metrics may include MAE, RMSE, forecast bias, calibration, inference latency, and energy use.

### Scheduling

The primary control is immediate execution: run the workload as soon as it is submitted. Forecast-based strategies will be evaluated against that baseline using:

- estimated carbon emissions;
- electricity cost;
- cost and carbon reduction;
- workload delay;
- deadline success rate; and
- GPU utilization.

Experiments will also compare carbon-focused, cost-focused, and balanced objectives across different workload flexibility windows.

## Reproducibility

Reproducibility is a core project goal. Experiments should document, where practical:

- dataset provenance and versions;
- preprocessing and timestamp handling;
- train, validation, and test splits;
- random seeds and model configurations;
- hardware and software dependencies; and
- evaluation commands and outputs.

Results should be represented as code rather than relying only on manually run notebooks.

Raw, interim, and processed datasets are intentionally gitignored. No datasets have been downloaded or committed yet.

## Planned repository structure

The structure will evolve with the research:

```text
Verde/
├── data/
├── forecasting/
├── scheduler/
├── training/
├── evaluation/
├── experiments/
├── tests/
├── examples/
├── docs/
├── metadata/
├── configs/
├── README.md
├── LICENSE
└── pyproject.toml
```

Installation and usage instructions will be added once the first runnable implementation is available. Phase 1 is currently limited to data contracts, provenance documentation, and acquisition preparation. Future work may include Python APIs, pretrained artifacts potentially hosted on Hugging Face, Docker/Kubernetes/Slurm integrations, GPU-cluster integrations, and ML training pipelines; none are current Phase 1 requirements or supported unless documented here.

## Roadmap

- [x] Define the research question and initial hypotheses
- [ ] Identify datasets and geographic/grid scope
- [ ] Define forecasting targets and evaluation methodology
- [ ] Build reproducible data-ingestion and preprocessing pipelines
- [ ] Implement forecasting and immediate-execution baselines
- [ ] Implement and evaluate forecast-based scheduling
- [ ] Measure model efficiency and cost/carbon tradeoffs
- [ ] Publish reproducibility documentation and benchmark results
- [ ] Release a Python API and pretrained model artifacts

## Contributing

Verde is intended to be an open research project. Current Phase 1 contributions should prioritize data contracts, source documentation, acquisition reproducibility, and data-quality methodology. Future contributions may include datasets, forecasting models, scheduling algorithms, evaluation methods, integrations, tests, documentation, and reproducibility improvements.

Please distinguish established results from proposed ideas or hypotheses. Changes that affect research conclusions should include appropriate experiments or evaluation. Contribution guidelines will expand as the project matures.

## Research and production use

Verde is not currently a production system and does not guarantee electricity savings, carbon reductions, financial savings, workload performance, deadline compliance, or operational reliability. Production users should independently validate data sources, forecasts, energy measurements, and scheduling decisions.

## License and model artifacts

The project license will be specified in [`LICENSE`](LICENSE). Subject to licensing and data-availability constraints, trained model artifacts may be published with model documentation, potentially through Hugging Face, as future work. Model artifacts and deployment infrastructure are not requirements of the current Phase 1 data foundation.

## Vision

Verde asks whether AI infrastructure can become more environmentally and economically efficient by intelligently deciding when flexible workloads should run.

The long-term vision is open-source infrastructure that helps AI systems use computational resources more intelligently.
