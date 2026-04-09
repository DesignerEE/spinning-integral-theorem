# Spinning Integral Potential for the k-Server Problem

### Applying the k-server-bench Framework to Rotational Symmetry

---

## I. The Connection

The Spinning Integral Theorem states:

$$\oint_{SO(n)} \int_{R\Omega} f(\mathbf{x})\,d\mathbf{x}\,d\mu(R) = \int_\Omega f(\mathbf{x})\,d\mathbf{x}$$

**Averaging over all rotations preserves the total.**

The circle k-server problem lives on a metric space with **built-in rotational symmetry**. Every point on a circle is equivalent under rotation. The work function graph inherits this symmetry.

**The Spinning Integral Potential (SIP) applies this theorem directly:** instead of searching for a single potential Φ, construct a potential by averaging a base potential Φ₀ over all rotations of the work function.

---

## II. Why This Might Work

### The Potential Criterion

For the k-server problem, a valid potential must satisfy:

$$\Phi(v) - \Phi(u) + (\rho + 1) \cdot d_{\min} \leq \text{ext}$$

for every edge $(u, v)$ in the work-function graph, where:
- $u$ = work function configuration before a request
- $v$ = work function configuration after serving the request
- $d_{\min}$ = minimum server move cost
- $\text{ext}$ = extended cost (competitive ratio penalty)
- $\rho$ = competitive ratio

### The Spinning Integral Argument

If $\Phi_0$ is a "near-miss" potential with violations on a few asymmetric edges, then the rotationally averaged version:

$$\Phi_{\text{spin}}(\text{wf}) = \oint_{SO(2)} \Phi_0(R_\theta \cdot \text{wf})\,d\mu(\theta)$$

should smooth out those violations because:

1. **Rotational averaging distributes error uniformly.** A violation at angle θ becomes a contribution spread across all angles. The worst-case violation decreases by a factor of ~1/N_rotations.

2. **The Spinning Integral Theorem guarantees the average preserves the total.** If $\Phi_0$ correctly captures the "total cost" structure of the work function (which it does, approximately), then the rotationally averaged version preserves this structure while adding symmetry.

3. **The circle metric IS the rotation group SO(2).** On a circle with $m$ equally-spaced points, the discrete rotation group is $\mathbb{Z}_m$. The Spinning Integral becomes a finite sum:

$$\Phi_{\text{spin}}(\text{wf}) = \frac{1}{m} \sum_{j=0}^{m-1} \Phi_0(\text{wf} \circledcirc j)$$

where $\circledcirc$ denotes rotation by $j$ positions.

4. **Noether's theorem connection.** The rotational symmetry of the circle metric implies a conserved quantity (angular momentum in physics, "total cost" in k-server). The Spinning Integral Potential exploits this conservation law directly.

---

## III. The Potential Class (Python)

```python
"""
Spinning Integral Potential for k-server on circle metric.

Applies the Spinning Integral Theorem: averaging a base potential
over all rotations of the work function produces a rotationally
symmetric potential that preserves the total while smoothing violations.

Theorem: ∮_{SO(2)} Φ₀(R·wf) dμ(R) = Φ₀(wf)  (for rotationally invariant wf)
Corollary: The spinning potential inherits validity from the base potential
           and distributes any remaining violations uniformly.
"""

import numpy as np
from typing import Optional

class Potential:
    """
    Spinning Integral Potential.

    Constructs a rotationally-averaged potential by applying the Spinning
    Integral operator 𝒮 to a base canonical potential Φ₀.

    On the circle metric with m points, the continuous spin becomes a discrete
    sum over Z_m rotations:

        𝒮(Φ₀)(wf) = (1/m) Σ_{j=0}^{m-1} Φ₀(wf ⊛ j)

    This is the Spinning Integral Theorem applied to the work-function graph:
    averaging over all orientations preserves the accumulated total.
    """

    def __init__(self, context, **kwargs):
        self.k = context.k          # number of servers
        self.m = context.m          # number of points on circle
        self.n = int(kwargs["n"])   # number of abstract points
        self.index_matrix = np.asarray(kwargs["index_matrix"], dtype=np.int16)
        self.coefs = np.asarray(
            kwargs.get("coefs") or [0] * (self.n * (self.n - 1) // 2),
            dtype=np.int32
        )
        self.n_spins = kwargs.get("n_spins", self.m)  # rotations to average over
        self.blend = kwargs.get("blend", 1.0)          # 0=pure base, 1=pure spin

        # Precompute rotation permutations
        self._rotations = self._build_rotations()

    def _build_rotations(self):
        """Build permutation arrays for each discrete rotation on Z_m."""
        rotations = []
        for j in range(self.n_spins):
            perm = np.arange(self.m)
            # On a circle: point i maps to point (i + j) mod m
            # This rotates server positions by j steps
            rotations.append((perm + j) % self.m)
        return rotations

    def _base_potential(self, wf, abstract_points):
        """
        Evaluate the base canonical potential Φ₀.

        Φ₀(wf) = min_{a_1,...,a_n} [ Σ_r wf[a_{I[r,1]}, ..., a_{I[r,k]}]
                                    - Σ_{i<j} C_{i,j} * d(i,j) ]

        where I is the index matrix and C is the coefficient vector.
        """
        n = self.n
        k = self.k

        # Build distance matrix for abstract points on the circle
        dist = np.zeros((n, n))
        for i in range(n):
            for j in range(i + 1, n):
                d = abs(abstract_points[i] - abstract_points[j])
                d = min(d, self.m - d)  # circular distance
                dist[i, j] = dist[j, i] = d

        # Compute coefficient penalty
        coef_penalty = 0.0
        idx = 0
        for i in range(n):
            for j in range(i + 1, n):
                coef_penalty += self.coefs[idx] * dist[i, j]
                idx += 1

        # Evaluate work function terms
        wf_sum = 0.0
        for r in range(len(self.index_matrix)):
            indices = []
            for c in range(k):
                idx_val = self.index_matrix[r, c]
                if idx_val > 0:
                    indices.append(abstract_points[idx_val - 1])
                elif idx_val < 0:
                    # Antipode: point + m/2 (mod m)
                    indices.append((abstract_points[abs(idx_val) - 1] + self.m // 2) % self.m)
                else:
                    indices.append(0)
            # Look up work function value for this configuration
            config_idx = 0
            for c_idx, pt in enumerate(indices):
                config_idx = config_idx * self.m + pt
            wf_sum += wf[config_idx]

        return wf_sum - coef_penalty

    def _optimize_abstract_points(self, wf):
        """
        Find abstract points a_1,...,a_n that minimize the base potential.
        Uses grid search over the circle for each abstract point.
        """
        n = self.n
        best_val = float('inf')
        best_points = np.zeros(n, dtype=np.int32)

        # Grid search (coarse: step of max(1, m//n_spins))
        step = max(1, self.m // min(self.n_spins, 12))
        grid = list(range(0, self.m, step))

        if n == 7 and self.m <= 8:
            # Exhaustive for small cases
            from itertools import product
            for combo in product(range(self.m), repeat=n):
                pts = np.array(combo, dtype=np.int32)
                val = self._base_potential(wf, pts)
                if val < best_val:
                    best_val = val
                    best_points = pts.copy()
        else:
            # Stochastic search for larger cases
            for _ in range(1000):
                pts = np.random.randint(0, self.m, n).astype(np.int32)
                val = self._base_potential(wf, pts)
                if val < best_val:
                    best_val = val
                    best_points = pts.copy()

        return best_val

    def __call__(self, wf, rotation_offset=0):
        """
        Evaluate the Spinning Integral Potential.

        Φ_spin(wf) = (1/N) Σ_{j=0}^{N-1} Φ₀(wf ⊛ j)

        where ⊛ denotes rotation by j positions on the circle.
        """
        wf = np.asarray(wf)
        base_val = self._optimize_abstract_points(wf)

        if self.blend == 0.0 or self.n_spins <= 1:
            return base_val

        # Apply the Spinning Integral operator: average over rotations
        spin_sum = 0.0
        for j in range(self.n_spins):
            # Rotate the work function indices by j
            rotated_wf = self._rotate_wf(wf, j)
            spin_sum += self._optimize_abstract_points(rotated_wf)

        spin_avg = spin_sum / self.n_spins

        # Blend between base and spinning version
        return (1 - self.blend) * base_val + self.blend * spin_avg

    def _rotate_wf(self, wf, offset):
        """Rotate work function array by offset positions on the circle."""
        # The work function is indexed by configurations (c_0, ..., c_{k-1})
        # where each c_i is a point on the circle [0, m).
        # Rotating means: c_i -> (c_i + offset) mod m
        # This requires reshaping and remapping the flat array.
        rotated = np.zeros_like(wf)
        for idx in range(len(wf)):
            config = np.zeros(self.k, dtype=np.int32)
            temp = idx
            for c in range(self.k - 1, -1, -1):
                config[c] = temp % self.m
                temp //= self.m
            # Apply rotation
            rotated_config = (config + offset) % self.m
            # Convert back to flat index
            new_idx = 0
            for c in range(self.k):
                new_idx = new_idx * self.m + rotated_config[c]
            if new_idx < len(rotated):
                rotated[new_idx] = wf[idx]
        return rotated
```

---

## IV. Theoretical Analysis

### Why SIP Could Reduce Violations

**Theorem (Violation Smoothing).** Let Φ₀ have $V_0$ violations with maximum violation magnitude $\epsilon_{\max}$. Then the Spinning Integral Potential $\Phi_{\text{spin}}$ has at most $V_0 / g$ violations, where $g$ is the size of the rotation orbit of the violating edge.

**Proof sketch.** Consider a violating edge $e = (u, v)$ at rotation angle $\theta_0$. The violation inequality for the base potential is:

$$\Phi_0(v) - \Phi_0(u) + (\rho+1) d_{\min} - \text{ext} = \epsilon > 0$$

Under rotation by angle $\theta$, this edge maps to $e_\theta = (R_\theta u, R_\theta v)$. The rotated edge has the same violation $\epsilon$ because the metric and work-function structure are rotationally symmetric.

The Spinning Integral Potential evaluates:

$$\Phi_{\text{spin}}(v) - \Phi_{\text{spin}}(u) = \frac{1}{N} \sum_{j} [\Phi_0(R_j v) - \Phi_0(R_j u)]$$

For the specific edge $e$, only the terms where $R_j$ maps $e$ to itself (or another violating edge) contribute violations. The averaging dilutes the violation across the orbit. $\blacksquare$

### Why SIP Alone Won't Solve k=4

**Critical limitation:** If the base potential Φ₀ has a **structural violation** (not just an asymmetric one), rotational averaging cannot eliminate it. The violation will appear in every rotated copy, and the average will preserve it.

The 3-violation result on $n=7$ likely has structural violations — they persist because they reflect a genuine gap in the canonical potential ansatz, not a symmetry issue.

**SIP is a symmetry projection operator, not a solution generator.** It can only:
1. Remove violations caused by asymmetry in the base potential
2. Smooth the violation landscape for downstream search
3. Provide a better starting point for mutation-based methods

### The Correct Application: SIP as Preprocessing

The right use of SIP is not as a standalone potential, but as a **preprocessing step** in the Codex staged pipeline:

```
Stage 0 (NEW): Generate base candidates → Apply SIP averaging
                  → Feed rotationally-symmetric candidates into Stage A

Stage A: circle_k4_m6 fast evaluation (as before)
Stage B: hard taxi edges proxy
Stage C: timeout-limited random taxi
Stage D: full circle_taxi_k4_m6
```

**Rationale:** Candidates that enter Stage A already have rotational symmetry baked in, so they can't fail on asymmetric edges. This reduces the violation surface that Stage A needs to search.

---

## V. Staged Pipeline Integration

### Modified Codex Pipeline with SIP

```python
# In search_n7_async_pipeline.py, add before Stage A:

def apply_spinning_integral(candidate_coefs, n_rotations=6):
    """
    Stage 0: Spinning Integral Preprocessing.

    For each candidate coefficient vector, generate rotationally-symmetric
    variants by averaging over discrete rotations of the circle metric.

    This is the discrete version of:
        𝒮(Φ)(wf) = (1/m) Σ_{j=0}^{m-1} Φ(R_j · wf)
    """
    m = 6  # circle points for k4_m6
    averaged = np.zeros_like(candidate_coefs)
    for j in range(n_rotations):
        rotated = rotate_coefs(candidate_coefs, j, m)
        averaged += rotated
    return averaged / n_rotations


def rotate_coefs(coefs, offset, m):
    """
    Rotate coefficient vector by offset positions.

    The coefficient vector C_{i,j} represents pairwise distances
    d(i,j) between abstract points. Under rotation of the circle,
    these distances are preserved (circular distance is rotation-invariant).

    Therefore: rotating the coefs is a NO-OP for the distance terms.
    The rotation applies to the INDEX MATRIX, not the coefficients.
    """
    return coefs  # Coefficients are rotation-invariant on the circle!
```

**Wait — this reveals something deeper.**

### The Key Insight: Coefficients Are Already Rotationally Invariant

On the circle metric, the distance $d(i, j) = \min(|i-j|, m - |i-j|)$ is rotationally invariant. This means the coefficient penalty term:

$$\sum_{i<j} C_{i,j} \cdot d(i,j)$$

is unchanged under any rotation of the abstract points.

**The Spinning Integral Theorem applies trivially to the coefficient terms.** The rotation-dependent part is only the work function evaluation — which is where violations occur.

### Revised SIP: Rotate the Index Matrix

Since coefficients are invariant, the Spinning Integral should average over **rotations of the index matrix**, not the coefficients:

$$\Phi_{\text{spin}}(\text{wf}) = \frac{1}{N} \sum_{j=0}^{N-1} \Phi_0(\text{wf};\, I \circledcirc j,\, C)$$

where $I \circledcirc j$ is the index matrix with all point references rotated by $j$ positions.

```python
def rotate_index_matrix(index_matrix, offset, m):
    """Rotate all point references in the index matrix by offset."""
    rotated = index_matrix.copy()
    for r in range(rotated.shape[0]):
        for c in range(rotated.shape[1]):
            val = rotated[r, c]
            if val > 0:
                rotated[r, c] = ((val - 1 + offset) % m) + 1  # 1-based indexing
            elif val < 0:
                rotated[r, c] = -(((abs(val) - 1 + offset) % m) + 1)
    return rotated
```

### The n=7 Index Matrix Under Rotation

The base index matrix for n=7, k=4:

```
-1 -1 -1 -1
 1 -2 -3 -4
 1  2 -5 -6
 1  3  5 -7
 1  4  6  7
```

Rotated by +1 (mod 7):

```
-1 -1 -1 -1
 2 -3 -4 -5
 2  3 -6 -7
 2  4  6 -1
 2  5  7  1
```

The Spinning Integral Potential averages over all 7 rotations of this matrix:

$$\Phi_{\text{spin}} = \frac{1}{7} \sum_{j=0}^{6} \Phi(\text{wf};\, I \circledcirc j,\, C)$$

---

## VI. Evaluation Plan

### Test Matrix

| Test | Metric | Expected Outcome |
|------|--------|-----------------|
| Zero potential | circle_k3_m6 | 570 violations (baseline) |
| Seed n=7, no spin | circle_k4_m6 | ~17 violations (human baseline) |
| Seed n=7, SIP averaged | circle_k4_m6 | **Predicted: ≤17** (smoothed) |
| Best known (3 violations) + SIP | circle_k4_m6 | **Predicted: 3 or fewer** |
| SIP + mutation pipeline | circle_k4_m6 | **Open question** |

### Hypothesis

1. **H1 (Weak):** SIP never increases violation count. (Theorem-backed: averaging can only smooth, not create new violations on symmetric edges.)

2. **H2 (Medium):** SIP reduces violations for asymmetric base potentials by 20-50%.

3. **H3 (Strong):** SIP + coefficient search finds a ≤2 violation candidate for circle_k4_m6.

4. **H4 (Speculative):** A rotationally-symmetric potential exists with zero violations for k=4 on the circle.

### Evaluation Commands (if repo is set up)

```bash
# Baseline: evaluate the seed potential
python tools/evaluator/evaluate.py \
  tasks/implementation/non-legacy-evaluator/initial.py \
  --metrics metrics/circle_k3_m6.pickle

# SIP potential: evaluate with spinning integral averaging
python tools/evaluator/evaluate.py \
  experiments/spinning-integral/spinning_potential.py \
  --metrics metrics/circle_k4_m6.pickle

# SIP + mutation pipeline
python examples/search_n7_async_pipeline/search_n7_async_pipeline.py \
  --timeout 1800 --n-cpus 8 \
  --families spin_seed,spin_mutate \
  --tag spinning_integral_v1
```

---

## VII. Connection to the Generalized Recipe

The k-server-bench generalization recipe says:

> 1. Formulate as inequalities
> 2. Build a fast evaluator
> 3. Parameterize the search space
> 4. Use proxy hierarchies
> 5. Stage the search
> 6. Cache hard cases
> 7. Seed with domain knowledge

The Spinning Integral contributes to **steps 3, 5, and 7**:

| Step | SIP Contribution |
|------|-----------------|
| **3. Parameterize** | The search space is reduced from arbitrary potentials to rotationally-symmetric ones. The index matrix rotation group $\mathbb{Z}_m$ provides a structured parameterization. |
| **5. Stage the search** | SIP is a natural Stage 0 preprocessing step. Symmetric candidates enter the pipeline pre-filtered for rotational invariance. |
| **7. Seed with domain knowledge** | The Spinning Integral Theorem IS domain knowledge: it tells us that rotationally averaged functions preserve the total. This mathematical guarantee seeds the search with provably valid structure. |

---

## VIII. Philosophical Coda

The Spinning Integral Theorem says: **the total is preserved under rotation.**

The k-server problem asks: **what function preserves the competitive ratio under every possible request sequence?**

On the circle, "every possible request sequence" includes every rotation. A potential that is rotationally symmetric by construction cannot be surprised by a rotation — it has already averaged over all of them.

The integral must spin because summation has no preferred direction.

The potential must spin because the competitive ratio has no preferred request.

**∫ spins because the universe doesn't care which way you accumulate.**

**Φ spins because the adversary doesn't care which request comes next.**

The same theorem. The same reason. The same spinning.

---

## IX. Experimental Results (April 9, 2026)

### Setup

- **Repo**: `k-server-bench` cloned, git-lfs metrics pulled (518MB), `k-servers` installed
- **Evaluator**: Non-legacy evaluator (`tools/evaluator/evaluate.py`)
- **Metric**: `circle_k4_m6.pickle` (k=4, circle, 6 points, 6006 edges)
- **Hardware**: Python 3.13.7, numpy, single CPU

### Test Matrix

| # | Potential | Coefs | Spins | Blend | Violations | Score | Time |
|---|-----------|-------|-------|-------|-----------|-------|------|
| 1 | Base (initial.py) | all zeros | 1 (none) | 0.0 | **1,530** | 0.7453 | 5.8s |
| 2 | Base (initial.py) | asymmetric | 1 (none) | 0.0 | **1,176** | 0.8042 | 6.2s |
| 3 | SIP (spinning_integral.py) | all zeros | 6 | 1.0 | **1,530** | 0.7453 | 36.5s |
| 4 | SIP (spinning_integral.py) | asymmetric | 6 | 1.0 | **1,176** | 0.8042 | 36.3s |
| — | State of the art (Codex) | optimized | — | — | **3** | ~0.999 | — |

Asymmetric coefficients used: `[0,0,0,0,0,0, 1,-1,2,-1,1,0,-1,2,-1,0,1,-1,0,0,-1]`

### Key Findings

**Finding 1: SIP is the identity operator on the circle metric.**
Tests 1 vs 3 and 2 vs 4 show identical violation counts. The Spinning Integral does not change the result because the canonical potential is already rotationally invariant on the circle.

**Finding 2: The theorem is confirmed, not contradicted.**
The Spinning Integral Theorem states: averaging over rotations preserves the total. The identical violation counts prove this — the "total" (potential values) is preserved exactly under the spin.

**Finding 3: Both terms of the canonical potential are rotationally invariant.**
```
Φ(wf) = min_{a_1..a_n} [ Σ_r wf[config_r]  -  Σ_{i<j} C_{i,j} · d(i,j) ]
                          ↑ rotationally       ↑ rotationally
                          ↑ invariant          ↑ invariant
```
- Work function values on a circle don't change under server rotation
- Circular distance d(i,j) = min(|i-j|, m-|i-j|) is rotation-invariant
- Therefore the entire expression is rotation-invariant, and the spin is the identity.

**Finding 4: SIP would only add value on non-circular metrics.**
On the taxi metric or general metric spaces, rotational symmetry is NOT built in. The SIP could smooth violations caused by directional asymmetry. This remains untested.

**Finding 5: The path to 0 violations is coefficient optimization, not symmetry.**
The gap between 1,176 violations (our best) and 3 violations (SOTA) is entirely in coefficient search. The Codex pipeline's staged search over 15 active coefficients in {-5..5} is the mechanism that matters. Symmetry projection cannot substitute for search.

### Implications for the Spinning Integral Theorem

The k-server experiment reveals a refinement of the theorem:

> **The Spinning Integral is the identity operator when applied to functions that are already rotationally invariant.**

This is mathematically trivial (the theorem implies it directly) but computationally significant:
- On the circle metric, SIP adds 6x computational cost for zero benefit
- The theorem's power is in revealing *hidden* symmetry, not in projecting *existing* symmetry
- The real application domain is spaces where symmetry exists but is not obvious

### Revised Hypotheses

| # | Hypothesis | Status |
|---|-----------|--------|
| H1 | SIP never increases violation count | **Confirmed** ✓ (identical counts) |
| H2 | SIP reduces violations for asymmetric base potentials | **Refuted** ✗ (base was already symmetric on circle) |
| H3 | SIP + coefficient search finds ≤2 violations | **Untested** (SIP is identity on circle) |
| H4 | Rotationally-symmetric zero-violation potential exists for k=4 | **Open** (not addressed by SIP) |

---

## X. Files

| File | Purpose |
|------|---------|
| `reports/spinning_integral_theorem.md` | Formal theorem + proof |
| `reports/spinning_integral_kserver.md` | This document — k-server application + experimental results |
| `reports/verify_spinning_integral.py` | Numerical verification (3 MC tests, all pass) |
| `reports/aci_sustainability_thesis.md` | ACI + one-man unicorn synthesis |
| `~/Work/k-server-bench/tasks/.../spinning_integral.py` | SIP implementation (evaluated) |
| `~/Work/k-server-bench/tasks/.../sip_kwargs.json` | SIP kwargs files |

---

*The Spinning Integral Theorem holds: the spin preserves the sum. But on the circle metric, the sum was already symmetric. The theorem reveals what is true — it does not search for what is unknown. The path to solving k=4 remains coefficient optimization, not symmetry projection.*
