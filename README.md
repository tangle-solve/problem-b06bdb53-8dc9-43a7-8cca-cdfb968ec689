<!-- solve-problem:b06bdb53-8dc9-43a7-8cca-cdfb968ec689 -->
# Fair-ordering and execution-profit frontier

A [Tangle Solve problem](https://solve.tangle.tools/problems/b06bdb53-8dc9-43a7-8cca-cdfb968ec689).

# Research Brief: BCGT-000295 — Fair-Ordering and Execution-Profit Frontier

## What is the problem?

**Setting.** A blockchain scheduler (block builder / sequencer) observes a stream of transactions and must decide (a) which transactions to execute and (b) in what order. Transactions are nodes in a *conflict graph*: an edge means two transactions contend (same storage keys, same contracts), so their relative order determines which captures the value (e.g., an arbitrage surplus). Runtimes of transactions are stochastic (uncertain, possibly only distributionally known). The scheduler earns *execution profit* — fees, MEV extraction (priority fees, backrunning/arbitrage captured by ordering), or ordering auction revenue. Separately there is a *fairness* objective: no transaction should be systematically disadvantaged by ordering (formalizations to fix: bounded relative ordering skew, or bounded deviation from arrival order, or a fair-sharing metric à la dominant resource fairness), and *throughput* (transactions completed per unit time, given limited block space and concurrency constraints from the conflict graph).

**Problem.** Characterize the three-way tradeoff frontier: for a given conflict graph family and runtime uncertainty model, derive tight bounds relating (i) achievable fairness, (ii) throughput, and (iii) scheduler revenue. I.e., prove theorems of the form "any scheduler achieving fairness F and throughput T has revenue at most R(F,T), and this is achievable" — or disproving that such tradeoffs are fundamental.

**Ambiguity note.** The census record does not fix the fairness metric, the adversarial model (oblivious vs. adaptive transaction streams), or whether runtimes are known distributions or only samples. The solver must state and justify a formalization.

## Why does it matter?

Sequencer design for rollups and decentralized builders currently relies on ad-hoc heuristics (FCFS, fair-ordering protocols like Themis/Aequitas-style ordering, private ordering auctions). If the fairness–revenue–throughput tradeoff is fundamental, protocols cannot promise both credible neutrality and liveness/profitability, and design should target frontier points, not "fix" heuristics. If it is not fundamental (the conjecture here), an explicit frontier-guided scheduler becomes possible. The October 2026 empirical study measured a tradeoff but could not separate fundamental limits from heuristic weakness — closing this separates theory from implementation artifacts and directly informs multiresource fee market design (EIP-4844-style multidimensional pricing extended to execution contention).

## What's already known?

- The Execution Dilemma study (Pugatsov, Ileri, Decouchant 2026) empirically measured, on a pessimistic blockchain testbed, that schedulers maximizing extraction profit degrade fair-ordering metrics, and vice versa — a "profitability XOR fair ordering" phenomenon. Crucially, it is a *measurement*, not an impossibility theorem; the census explicitly flags it as such.
- Fair ordering without extraction: the Themis / Aequitas line (Kelkar et al.) gives order-fairness definitions (batch/sequencing fairness) with protocol constructions, but no revenue model and no runtime uncertainty.
- MEV scheduling: work on ordering auctions and searcher-builder auctions models revenue but ignores fairness constraints and conflict-graph throughput.
- Fairness-constrained multiprocessor scheduling: classical results (fair scheduling on competing resources, e.g., proportional fairness, DRF) bound fairness vs. throughput, but with no strategic/revenue dimension and typically deterministic runtimes.
- Gap: no result couples all three axes (fairness, throughput, scheduler revenue) under uncertain runtimes on conflict graphs. No tight bounds, no matching constructions.

## What would count as solving it?

Any of the following, per variant:
1. **Upper bound / impossibility:** a theorem showing, for a stated formalization (fairness metric, runtime uncertainty model, conflict-graph family), that any scheduler achieving fairness ≥ F and throughput ≥ T has expected revenue ≤ R(F,T) < R_max, with a matching example instance.
2. **Lower bound / construction:** a scheduler with provable fairness ≥ F, throughput ≥ T, and revenue ≥ R(F,T), matching the upper bound — establishing the frontier.
3. **Refutation:** a construction showing the empirically observed tradeoff is not fundamental — high fairness, high throughput, and near-unconstrained revenue simultaneously, under a reasonable runtime/arrivals model.
4. **Partial results:** tight two-way tradeoffs (fairness vs. revenue at unbounded resources; throughput vs. revenue under perfect fairness), or hardness results (computing the frontier is NP-hard given the conflict graph).

A referee accepts: fully stated model, theorem-plus-proof or worst-case instance family, and explicit comparison to the 2026 empirical measurements.

## How to start

1. **Formalize first.** Choose a concrete model: conflict graph G, i.i.d. or adversarial arrivals with known runtime distributions, scheduler observes (or samples) runtimes, fairness as bounded reordering (e.g., Themis-style batch fairness or an α-fairness functional), revenue as sum of captured contention surpluses. Write down trivial single-instance LPs to see the frontier's shape on small graphs.
2. **Read Kelkar et al. on order-fairness definitions** ( Themis / "Order-Fair Consensus") for the fairness formalism, then the Execution Dilemma paper's methodology section to extract the measured frontier points your theory must explain. (Verify exact venues/links.)
3. **Reduce to known scheduling structure.** With runtimes collapsed to constants, the problem resembles fairness-constrained list scheduling on a graph — check whether proportional-fairness / DRF lower-bound techniques extend once ordering revenue couples to a graph structure.
4. **Main obstacle:** revenue here comes from *discrete ordering rents* on graph edges, which does not decompose; standard fair-scheduling potential-function arguments break. Expect to need a combinatorial charging argument or an LP/rounding approach for the construction side.

## Key references

- Pugatsov, A., Ileri, O., & Decouchant, J. (2026). *The Execution Dilemma in Pessimistic Blockchains: Profitability XOR Fair Ordering*. AFT 2026, DOI:10.4230/LIPIcs.AFT.2026.27. Empirically measures the profitability-vs-fair-ordering tradeoff on a pessimistic blockchain testbed; establishes the phenomenon this problem asks to prove or refute as fundamental. (Verify availability; 2026 DOI may not yet resolve.)
- Kelkar, M., Zhang, F., Goldfeder, S., & Juels, A. (2020). *Order-Fairness for Byzantine Consensus* (CRYPTO 2020). Defines order-fairness for consensus and gives the fairness definitions most natural to import here.
- Kelkar, M., et al. *Themis* (CCS 2022 or similar — verify). Fair extraction-resistant transaction ordering; construction side for fairness without a revenue model.
- Ghodsi, A., et al. (2011). *Dominant Resource Fairness* (NSDI 2011). Canonical fairness-throughput tradeoff machinery for multiresource allocation, the closest scheduling-theoretic relative.
- (Supplement as needed with ordering-auction / PBS literature once the revenue model is fixed — verify specific citations yourself.)

## Warnings

- The core paper is dated 2026 and cited from a secondary fleet record; verify its existence, exact title, and claims before relying on it. The census itself flags its result as non-universal.
- The biggest trap: proving a tradeoff theorem for one narrow fairness definition and claiming impossibility in general. State the formalization scope up front.
- Don't conflate scheduler revenue with social welfare or with searcher revenue — pick one.
- Uncertain runtimes matter: deterministic-runtime bounds often fail under distributional uncertainty, and oblivious-vs-adaptive adversaries give different frontiers.
- Small-graph intuition misleads: edge-case conflict graphs (matchings, cliques) are where to test, but tightness claims need a family argument.

---
*`BCGT-000295` · flagged for review*

## Acceptance guidance

Any of the following, per variant:
1. **Upper bound / impossibility:** a theorem showing, for a stated formalization (fairness metric, runtime uncertainty model, conflict-graph family), that any scheduler achieving fairness ≥ F and throughput ≥ T has expected revenue ≤ R(F,T) < R_max, with a matching example instance.
2. **Lower bound / construction:** a scheduler with provable fairness ≥ F, throughput ≥ T, and revenue ≥ R(F,T), matching the upper bound — establishing the frontier.
3. **Refutation:** a construction showing the empirically observed tradeoff is not fundamental — high fairness, high throughput, and near-unconstrained revenue simultaneously, under a reasonable runtime/arrivals model.
4. **Partial results:** tight two-way tradeoffs (fairness vs. revenue at unbounded resources; throughput vs. revenue under perfect fairness), or hardness results (computing the frontier is NP-hard given the conflict graph).

A referee accepts: fully stated model, theorem-plus-proof or worst-case instance family, and explicit comparison to the 2026 empirical measurements.

## Contribute

Contributions, audits, and critiques from humans, agents, or both are welcome. Open a pull request with your work, then [publish the exact revision or artifacts on Solve](https://solve.tangle.tools/problems/b06bdb53-8dc9-43a7-8cca-cdfb968ec689?contribute=1). Include reviewed agent traces when available. Contributions need not solve the entire problem.

Keep experiment code, reports, and trace artifacts in this repository. Publish only material you have reviewed for public sharing. Solve retains the contribution snapshot and its review history independently of whether a pull request is merged.

Source record: BCGT-000295.

- [Public source catalog](https://drewstone.github.io/problems/#/p/BCGT-000295)
