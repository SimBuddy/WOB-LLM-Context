# WOB LLM Context


**WOB is an experimental methodology**. Its goal is to provide a means of refactoring source code for data intensive applications, and increase their effectiveness.

**WOB Compared with Conventional Refactoring**
Traditional refactoring improves code structure and maintainability.
Performance optimization finds slow parts and makes them faster.
Profile-guided optimization measures where execution time is being spent and concentrates effort there.
Adaptive systems vary behaviour depending on conditions, usually within a specific mechanism.


**WOB takes a broader system-level view:**
Is this processing justified here, at this level, and in this form?
A conventional optimizer might find an expensive function and make it faster.
WOB may instead discover that the larger system can avoid, reduce, defer, reuse, reorder, or selectively deepen that processing.
The distinction is:
Traditional optimization improves execution. WOB reconsiders how much processing should happen, where, and when.
WOB can refactor code that has been already been conventionally refactored, because it is doing something entirely different. 


**EXPERIMENT 2** - In the experiment below, WOB is being applied to a range of varying data processing algorithms

Collectively, these algorithms represent very different ways computers spend effort to solve difficult problems: searching large spaces, retrieving information, estimating uncertain quantities, solving mathematical systems, integrating functions, reasoning through game trees, and deciding whether logical constraints can be satisfied. They were useful as a broad test set because each exposes a different kind of computational workload—search depth, samples, iterations, precision, function evaluations, graph exploration, model calls, or branch processing—making them a strong cross-section for testing whether work can be allocated more selectively without sacrificing the required result.
## Algorithms and systems tested

WOB has been tested against a range of established algorithms and computational systems.

The purpose was not simply to demonstrate that workload could be reduced, but to determine whether selective workload allocation could improve the strongest sensible existing implementation or fixed policy.

### Conditioning-aware mixed-precision Horner evaluation — WOB improvement

This is the programme's clearest confirmed WOB improvement.

The workload involved polynomial evaluation where some inputs could safely use ordinary floating-point arithmetic while others genuinely required much more expensive arbitrary-precision evaluation.

WOB used a cheap conditioning signal to decide when high precision was necessary.

Canonical integrated result:

* 120 evaluation cases
* 43 high-precision evaluations avoided
* 77 cases escalated to high precision
* 0 quality failures
* 28.07% reduction in paired elapsed computational cost

Same-machine replication:

* identical 43/120 routing
* 0 quality failures
* 34.91% mean paired saving
* 95% bootstrap interval: 30.24%–39.39%

A later clean-environment reproduction measured a 30.21% saving.

This is currently the strongest demonstrated case in which WOB itself improved the cost/quality frontier over the best tested fixed policy.

---

### Conjugate Gradient / Jacobi-PCG — standard algorithmic improvement discovered

WOB analysis of iterative linear solving compared ordinary Conjugate Gradient (CG) with Jacobi-preconditioned CG.

Jacobi-PCG reduced iteration count by:

* approximately 22.4% at the relaxed target
* approximately 24.7% at the strict target

Per-iteration cost remained essentially unchanged.

However, this was **not counted as a new WOB controller success**.

The existing preconditioning technique was simply a better algorithmic intervention than adding another adaptive stopping mechanism.

The experiment therefore produced:

`PCG-PRECONDITIONER-WINS / CG-STANDARD-STOPPING-WINS`

---

### QUADPACK adaptive quadrature — native algorithm already embodies the principle

QUADPACK was compared with fixed Gauss–Legendre quadrature.

At the `1e-6` target:

* fixed Gauss–Legendre: 2,049 evaluations, 76.7% success
* QUADPACK: 196.7 mean evaluations, 96.7% success

At the `1e-10` target:

* fixed Gauss–Legendre: 2,049 evaluations, 70.0% success
* QUADPACK: 249.9 mean evaluations, 98.3% success

QUADPACK already uses local error estimates and selectively subdivides the regions that need more work.

No additional WOB controller improved it.

Verdict:

`QUAD-STANDARD-ADAPTIVE-WINS / QUAD-NATIVE-ADAPTIVE-ALREADY-WOB`

---

### Monte Carlo simulation — standard sequential stopping wins

WOB was tested on 80 stationary M/M/1 queueing simulations.

Large theoretical sample-count savings existed, but a standard confidence-interval stopping rule performed better than the additional WOB-specific stability rule.

On the held-out test:

* WOB-specific CI + stability rule: 5,750 mean samples
* textbook CI stopping: 2,444 mean samples
* both achieved 100% target success

Verdict:

`MC-STANDARD-STOPPING-WINS`

This is another case where an established adaptive method already captured the useful workload allocation.

---

### HNSW approximate nearest-neighbour search — fixed setting wins

HNSW was tested using `hnswlib 0.8.0` over the full 5,183-document SciFact corpus.

Search effort values tested:

* `ef=20`
* `40`
* `80`
* `160`
* `320`

There was large theoretical adaptive headroom:

* mean oracle `ef`: 32.67 versus 320
* theoretical `ef` reduction: 89.8%

However, rare difficult queries could not be identified safely using the cheap signals tested.

A fixed:

`ef=80`

already matched full labelled test recall at roughly 30% of the latency of `ef=320`.

Verdict:

`HNSW-CHEAP-PREDICTION-FAILED`

The useful optimisation was therefore a simpler fixed setting rather than an adaptive WOB controller.

---

### A* / weighted A* — ordinary A* wins

A*/weighted-A* was tested on 120 official Moving AI benchmark scenarios.

Weights tested:

* 1.0
* 1.1
* 1.25
* 1.5
* 2.0
* 3.0

Ordinary A* (`w=1`) produced:

* 100% optimal paths
* 0% regret
* fewer mean expansions than every tested weighted condition

Although a per-instance oracle found approximately 10.8–12.6% theoretical expansion headroom, cheap adaptive prediction did not survive validation.

Verdict:

`ASTAR-CHEAP-PREDICTION-FAILED`

No A* modification is claimed.

---

### Stockfish 18 / alpha-beta search — no WOB integration

Stockfish 18 was tested using 45 valid native benchmark positions.

A depth-22 search was used as the practical reference.

Results included:

* depth-20/reference move agreement: 93.3%
* strict oracle node reduction: 79.4%
* false apparent move stabilization in 37.8% of positions

A conservative stopping policy passed train and validation but failed on a held-out position.

Stockfish also already contains extensive native workload allocation through pruning, reductions, extensions, iterative deepening, transposition tables and time management.

Verdict:

`ALPHABETA-CHEAP-PREDICTION-FAILED`

No Stockfish modification or performance improvement is claimed.

---

### MiniSat / CDCL SAT — no remaining economic headroom

MiniSat was tested using 50 SATLIB instances.

Instrumentation observed:

* 17,618 learned clauses
* only 6.4% had no recorded future use
* those clauses accounted for only 3.3% of learned-clause watch inspections
* MiniSat already deleted 52.9% of learned clauses

Instrumentation itself added approximately 6.2% wall-clock cost, exceeding the optimistic remaining workload opportunity.

Verdict:

`SAT-CLAUSE-NO-ADAPTIVE-HEADROOM`

No MiniSat modification is claimed.

---

### LLM / RAG retrieval and context processing — no deployment improvement claimed

Several retrieval and LLM-related mechanisms were investigated, including:

* adaptive dense retrieval breadth
* fusion fan-out
* hierarchical retrieval
* chunk granularity
* chunk overlap
* prompt admission
* LLM reranker depth
* synthesis topology
* progressive context acquisition

These experiments produced useful negative results but no WOB deployment improvement that passed the full cost/quality tests.

Examples include:

* fixed retrieval settings outperforming adaptive routing
* zero chunk overlap outperforming all tested overlap levels on Qasper
* a fixed prompt cap outperforming adaptive admission rules
* progressive LLM context processing costing more because hard cases required an additional model call

No general claim is made that WOB improves LLM or RAG systems.

---

## Summary of algorithm outcomes

| Algorithm / system                | Result                               |
| --------------------------------- | ------------------------------------ |
| Mixed-precision Horner evaluation | **WOB improvement confirmed**        |
| Jacobi-PCG                        | Standard preconditioning improvement |
| QUADPACK                          | Native adaptive algorithm wins       |
| Monte Carlo                       | Standard sequential stopping wins    |
| HNSW                              | Fixed `ef` policy wins               |
| A* / weighted A*                  | Ordinary A* wins                     |
| Stockfish 18                      | Cheap WOB prediction failed          |
| MiniSat / CDCL SAT                | No residual adaptive headroom        |
| LLM/RAG mechanisms                | No deployment improvement claimed    |

The important distinction is that WOB does not require every investigation to produce a new adaptive mechanism.

Sometimes WOB identifies a genuine selective-workload improvement.

Sometimes it identifies a better fixed or established algorithm.

Sometimes it shows that the native algorithm already allocates work effectively.

And sometimes the correct result is to make no change at all.

**Details of amended Jacobi-PCG algorithm - unchanged since 1952.**

The solver loop stayed almost identical. The meaningful change was the introduction of a Jacobi preconditioner using the inverse of the matrix diagonal.
Plain CG path

In the original CG logic, the residual itself is used directly:

z = r.copy()
rho = float(np.dot(r, z))

Since z == r, that is ordinary CG.

The search direction then evolves as:

p = z.copy() if it == 1 else z + (rho / rho_prev) * p

and the standard CG update follows:

q = A @ p
alpha = rho / float(np.dot(p, q))

x += alpha * p
r -= alpha * q
Jacobi-PCG amendment

The amendment was essentially this initialization:

invdiag = 1 / A.diagonal()

and then, on every iteration, instead of:

z = r.copy()

you use:

z = invdiag * r

So the actual implementation in the experiment was:

invdiag = 1 / A.diagonal() if jacobi else None

for it in range(1, maxiter + 1):

    z = invdiag * r if jacobi else r.copy()

    rho = float(np.dot(r, z))

    p = z.copy() if it == 1 else z + (rho / rho_prev) * p

    q = A @ p

    alpha = rho / float(np.dot(p, q))

    x += alpha * p
    r -= alpha * q

    rho_prev = rho

That is essentially the whole algorithmic amendment.

What the amendment means mathematically

Plain CG effectively works directly with:

Ax=b

Jacobi-PCG uses the matrix diagonal as a cheap approximation to the matrix itself:

M=diag(A)

and applies:

M
−1
r

to the residual before constructing the next search direction.

So instead of treating every coordinate of the residual equally, the correction is scaled according to the local diagonal magnitude of the matrix.

Conceptually:

CG:
residual
   ↓
search direction

PCG:
residual
   ↓
cheap diagonal scaling
   ↓
better-conditioned search direction
Why it was economically attractive

The extra per-iteration work was just approximately:

invdiag * r

—a vector multiply.

Meanwhile each iteration already contains the much more substantial sparse matrix-vector multiplication:

q = A @ p

So the preconditioner was cheap relative to the native iteration.

And it reduced the number of iterations by:

22.4% at the 1e-3 solution-error target
24.7% at the 1e-6 target

while measured per-iteration timing was essentially unchanged:

CG: about 15.72 μs/iteration
Jacobi-PCG: about 15.71 μs/iteration

So the change was tiny in code but substantial in total workload.

The particularly WOB-like discovery

The obvious temptation would have been to build an additional adaptive WOB stopping mechanism.

But the experiment found that the better amendment was simply:

ADD ONE CHEAP PRECONDITIONING STEP
→ REMOVE ~22–25% OF WHOLE ITERATIONS
→ KEEP STANDARD RESIDUAL STOPPING

That is a very clean example of WOB + Nitpicker technique together discovering additional optimisations.

Rather than trying to optimize the stopping decision itself, the experiment discovered that changing the quality of each iteration removed more workload than making the stopping controller more sophisticated.
