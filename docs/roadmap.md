# Verde roadmap

Verde is currently a California-focused research project. Its immediate question is:

> Can forecasting help schedule flexible AI workloads more intelligently within the California grid?

The longer-term research direction is broader:

> Can AI workloads be scheduled across time, geography, and distributed compute infrastructure while balancing carbon, electricity cost, compute availability, network constraints, and workload deadlines?

This roadmap describes a progression of research and engineering questions. It is not a commitment that every stage will be built, and future capabilities described here do not currently exist.

## Current scope: California research foundation

Phase 1 establishes the data and methodological foundation for an initial CAISO/California study:

- CAISO grid data and resource production data;
- NOAA GHCNh weather observations;
- CAISO OASIS market data as a candidate price source;
- EPA eGRID and EIA information as possible inputs to an estimated operational carbon signal;
- hourly canonical data in `America/Los_Angeles`;
- an intended historical study period of 2020–2024; and
- lightweight time-series forecasting with an initial 24-hour horizon.

The Phase 1 specification remains the source of truth for current data semantics, unresolved decisions, and implementation boundaries. The carbon target is not yet a measured hourly marginal signal, and no cost or environmental benefit has been established.

## Stage 1 — California forecasting foundation

The first research stage is to determine whether the California data foundation supports useful and reproducible forecasts.

Potential work includes:

- completing source-specific acquisition and validation;
- implementing persistence and historical-average baselines;
- defining and versioning an estimated operational carbon target if justified;
- forecasting electricity price and other supported grid variables;
- comparing lightweight statistical and machine-learning models;
- measuring forecast accuracy, latency, memory, training cost, and energy where practical; and
- documenting uncertainty, revisions, missingness, and chronological evaluation splits.

This stage should be considered complete only when its assumptions and limitations are explicit and its results are reproducible. A more accurate forecast is not automatically a more useful scheduling signal.

## Stage 2 — Time-aware workload scheduling

Once forecasting has been evaluated, Verde can investigate decisions rather than predictions alone:

> When should a flexible workload run?

The research could model:

- flexible batch workloads;
- workload duration and resource requirements;
- run-now versus delayed execution;
- deadlines and service-level constraints;
- carbon and electricity-cost objectives;
- delay penalties;
- forecast uncertainty; and
- comparisons with immediate-execution and simple scheduling baselines.

The goal would be to measure estimated scheduling outcomes, not to assume that forecast accuracy produces savings. Experiments should report deadline success, delay, cost, carbon estimates, and the computational cost of the forecasting and scheduling system itself.

## Stage 3 — Geographic workload scheduling

The next conceptual expansion is from:

> When should a workload run?

to:

> Where and when should a workload run?

This stage could compare multiple electricity grids or regions using:

- regional carbon-intensity estimates;
- electricity prices;
- renewable availability;
- GPU availability and regional capacity;
- data-transfer cost and network latency;
- workload and data locality;
- reliability and interruption risk; and
- deadline and service-level requirements.

California is the initial research environment, not a permanent geographic boundary. Expansion should occur only after the single-region methodology is understood well enough to distinguish genuine geographic effects from differences in data quality, boundaries, and measurement conventions.

## Stage 4 — Multi-region AI workloads

At this stage, Verde could investigate placing independent or loosely coupled workloads across several regions. The scheduler would need to reason about:

- candidate region selection;
- heterogeneous GPU types and capacities;
- regional electricity and carbon differences;
- workload placement and migration costs;
- data movement and locality;
- network latency and bandwidth;
- uncertainty in forecasts and capacity;
- failures and reliability; and
- deadline or SLA constraints.

Moving work between regions is not automatically beneficial. A lower-carbon or lower-price region may have higher transfer cost, insufficient capacity, higher latency, different reliability, or a less suitable accelerator. The full objective must account for those tradeoffs.

## Stage 5 — Distributed training

Distributed training introduces constraints that differ substantially from independent flexible batch jobs. Future research could examine:

- multi-GPU and multi-node training;
- synchronized distributed execution;
- checkpointing and restart time;
- preemption and resumption;
- GPU utilization and fragmentation;
- communication overhead;
- interconnect and network constraints; and
- placement across heterogeneous clusters.

A synchronized training job cannot simply be moved between countries or regions at arbitrary times. It may require a coordinated allocation of compatible GPUs, stable high-bandwidth communication, checkpoint boundaries, and a feasible migration or restart plan. These constraints should be modeled explicitly before making scheduling claims.

## Stage 6 — International and global scheduling

If regional scheduling produces meaningful evidence, Verde could explore a scheduler that reasons across North America, Europe, Asia, and other regions.

A possible future objective could combine:

```text
carbon
+ electricity cost
+ compute cost
+ network and data-transfer cost
+ delay penalty
+ reliability risk
```

subject to:

```text
GPU capacity
workload requirements
data locality
network constraints
deadline or SLA
regional availability
```

The international problem is substantially more complex than the initial California problem. It introduces differences in grid boundaries, market designs, time zones, data-access policies, carbon accounting, regulations, network conditions, and infrastructure availability. Global comparisons must therefore use carefully aligned definitions and uncertainty reporting.

International movement of workloads should never be treated as inherently carbon-reducing. Any claimed benefit would need to include compute energy, network and data-transfer energy, operational constraints, reliability effects, and the relevant emissions methodology.

## Stage 7 — Production infrastructure

Only after the research stages demonstrate useful, reproducible behavior might Verde explore production-oriented engineering such as:

- real-time data ingestion;
- real-time forecasting;
- workload submission APIs;
- Kubernetes and cloud integrations;
- GPU-cluster integrations;
- Slurm or other scheduler integrations;
- monitoring and workload telemetry;
- dynamic rescheduling;
- failure handling and rollback;
- evaluation infrastructure; and
- reproducible production benchmarks.

These are future directions, not current Verde capabilities or Phase 1 requirements. Production use would require independent validation of data quality, forecasts, energy measurements, reliability, security, and workload performance.

## Research principles

Verde should evolve through evidence rather than by accumulating features.

1. **Validate each stage before expanding scope.** A multi-region system should not be built to avoid answering unresolved single-region questions.
2. **Prefer measurable experiments over claims.** Compare against immediate execution, simple forecasting, and other meaningful baselines.
3. **Preserve reproducibility.** Record source versions, transformations, hardware, configurations, and evaluation splits.
4. **Quantify uncertainty.** Forecast uncertainty, source revisions, capacity uncertainty, and methodological limitations should be visible in results.
5. **Account for the full cost of moving work.** Include compute, transfer, network, latency, migration, and restart costs where relevant.
6. **Balance competing constraints.** Carbon should not be optimized while ignoring electricity cost, compute availability, network conditions, reliability, or deadlines.
7. **Respect workload structure.** Independent batch jobs and synchronized distributed training require different scheduling models.
8. **Avoid unsupported environmental or financial claims.** Benefits must be demonstrated experimentally and labeled as measured, estimated, or simulated according to the evidence.

## Current versus future capabilities

### Current Phase 1 scope

- California and CAISO;
- historical research data;
- hourly canonical timeline;
- `America/Los_Angeles` canonical timezone;
- source contracts and provenance rules;
- an initial 24-hour forecasting problem; and
- research into lightweight forecasting models.

### Future research directions

- time-aware workload scheduling;
- multi-region placement;
- GPU-aware scheduling;
- distributed training placement;
- international grid comparisons;
- global orchestration;
- real-time ingestion and forecasting; and
- production integrations.

None of the future directions should be presented as implemented until the corresponding research and engineering work has been completed and evaluated.

## Vision

Verde begins as a California grid forecasting and workload-scheduling research project and may eventually evolve into a system that reasons across **when, where, and how AI workloads consume compute**, particularly as increasingly large GPU workloads create substantial electricity, carbon, and infrastructure costs.
