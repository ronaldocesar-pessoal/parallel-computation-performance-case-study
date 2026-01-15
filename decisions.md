# Engineering decisions and trade-offs

This document describes the key engineering decisions behind the approach presented in this repository.  
The focus is not on a specific implementation, but on the reasoning that guided architectural and processing choices.

The original context involved a large-scale actuarial computation, but the decisions described here apply to any workload characterized by heavy computation, large datasets, and strict time constraints.

---

## 1. Identify the real bottleneck before scaling infrastructure

The initial performance problem was not caused by insufficient hardware.

The system was already running on machines capable of executing multiple threads, but the software architecture did not take advantage of this capacity. The workload was predominantly executed in a sequential manner, leaving most CPU cores idle.

**Decision:** focus first on software-level efficiency before considering hardware upgrades.

**Trade-off:** requires refactoring and careful analysis, but avoids unnecessary infrastructure costs.

---

## 2. Separate data loading from computation

One of the most impactful decisions was to remove disk IO from the critical execution path.

All data required for the computation is loaded and prepared in memory during application startup. Once the calculation phase begins, no further disk access is required.

**Why this matters:**
- Disk access is orders of magnitude slower than memory access
- Repeated IO during computation amplifies latency
- Predictable in-memory access dramatically improves throughput

**Trade-off:** increased memory usage in exchange for vastly improved execution time.

---

## 3. Prefer data parallelism over task parallelism

The workload exhibits a high degree of independence between individual records.

Instead of creating complex task orchestration, the solution applies **data parallelism**, partitioning the dataset and processing independent chunks concurrently.

This approach:
- simplifies reasoning
- reduces synchronization overhead
- maps naturally to multi-core CPU architectures

**Decision:** use parallel loops and partitioned data processing rather than fine-grained task graphs.

---

## 4. Be conservative with shared state

Parallel execution introduces risks related to concurrency and shared resources.

To minimize complexity:
- mutable shared state is avoided whenever possible
- aggregations are performed using local accumulators and combined at the end
- synchronization primitives are used sparingly and intentionally

**Decision:** prioritize correctness and predictability over maximum theoretical parallelism.

---

## 5. Measure, compare, and validate continuously

Performance improvements were validated through systematic comparison between:
- sequential execution
- parallel execution

Each refactoring step was evaluated to ensure that:
- results remained correct
- performance gains were measurable
- complexity did not grow disproportionately

**Decision:** rely on empirical measurement rather than intuition.

---

## 6. Optimize for repeatability and scenario analysis

A key requirement of the original context was the ability to run multiple scenarios within practical time limits.

Reducing execution time was not only about speed, but about enabling:
- frequent recalculation
- sensitivity analysis
- rapid decision support

**Decision:** treat performance as an enabler of better decision-making, not merely an optimization goal.

---

## 7. CPU parallelism before GPU acceleration

Although GPU-based acceleration (e.g., CUDA) can provide significant gains, it was deliberately not the first step.

The approach followed this order:
1. remove architectural bottlenecks
2. exploit CPU multi-core parallelism
3. stabilize and measure results
4. only then evaluate GPU acceleration if necessary

**Rationale:** many systems fail to fully utilize CPU parallelism, making GPU adoption premature and unnecessary in many cases.

---

## Final remarks

The decisions documented here are intentionally conservative.

They favor:
- simplicity
- correctness
- predictability
- long-term maintainability

These principles remain applicable regardless of programming language, framework, or execution environment, which is why the approach remains relevant years after the original study.
