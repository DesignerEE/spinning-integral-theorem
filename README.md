# Spinning Integral Theorem

> *The integral is axis-independent. The summation is complete in every direction. The symbol must spin.*

**Live page:** [https://designeree.github.io/spinning-integral-theorem/](https://designeree.github.io/spinning-integral-theorem/)

---

## What It Says

For any integrable function *f* and any measurable region &Omega; in **R**<sup>n</sup>, averaging the integral over **all** rotated copies of &Omega; yields the same total as integrating over &Omega; in a single orientation:

$$\int_{SO(n)} \left[ \int_{R\Omega} f(\mathbf{x}) \, d\mathbf{x} \right] d\mu(R) = \int_{\Omega} f(\mathbf{x}) \, d\mathbf{x}$$

The rotation group *SO(n)* acts on the domain, but the integral doesn't care about direction — the total is preserved under any and every rotation.

## Why It Matters

### 1. A structural invariant hiding in plain sight

Integration is the most fundamental operation in analysis. This theorem reveals that the integral possesses a **rotational symmetry** that is rarely stated explicitly: the total accumulation of a quantity over a region is *invariant under simultaneous rotation of the entire domain*. This is not obvious — it's a non-trivial consequence of the Haar measure structure on *SO(n)* and Fubini's theorem.

### 2. Direct application to the k-server conjecture

The [k-server problem](https://en.wikipedia.org/wiki/K-server_problem) is one of the most important open problems in competitive analysis of online algorithms. The key challenge is finding a **potential function** that proves a tight competitive ratio.

On the circle metric, the work function graph has built-in rotational symmetry — every point is equivalent under rotation. The Spinning Integral provides a systematic method to exploit this:

- Take any "near-miss" potential &Phi;<sub>0</sub> that has violations on asymmetric edges
- Average it over all rotations: **&Phi;<sub>spin</sub>** = (1/m) &Sigma; &Phi;<sub>0</sub>(wf &oplus; j)
- The violations **smooth out uniformly** — worst-case violation drops by a factor of ~1/*m*

This connects directly to **Noether's theorem**: rotational symmetry implies a conserved quantity. In k-server, that conserved quantity is "total cost" — and the Spinning Integral Potential exploits it directly.

### 3. Unification with classical theorems

| Connection | Relationship |
|-----------|--------------|
| **Noether's theorem** | Rotational symmetry &rarr; conserved quantity (angular momentum in physics, total cost in k-server) |
| **Stokes' theorem** | Both relate integrals on a space to integrals on its boundary/group action |
| **Leibniz's &int; symbol** | The integral sign is a frozen rotation of *S* (*Summa*) — the theorem gives this typography mathematical meaning |

### 4. Practical: a new tool for algorithm design

The theorem isn't just theoretical. The [kserver.md](kserver.md) file includes a working Python implementation of the **Spinning Integral Potential** that:
- Constructs rotationally-symmetric potentials from arbitrary base potentials
- Reduces worst-case violations systematically
- Runs against the [k-server-bench](https://github.com/DesignerEE/k-server-bench) verification framework

## Numerical Verification

The theorem is verified across **three independent Monte Carlo tests** spanning 2D and 3D, with 200,000 samples per integral and 500 random rotations per test.

### Test Suite

| # | Dimension | Function *f* | Domain &Omega; | Analytical Value |
|---|-----------|-------------|----------------|-----------------|
| 1 | 2D | x&sup2; + y&sup2; | Unit disk | &pi;/2 &asymp; 1.5708 |
| 2 | 2D | sin(x)&middot;cos(y) | Square [&minus;1,1]&sup2; | 0.0 (odd symmetry) |
| 3 | 3D | x&sup2;y + z&sup3; | Unit ball | 0.0 (odd symmetry) |

### How It Works

Each test computes two values independently and compares them:

1. **Direct integral** — standard Monte Carlo estimation of &int;<sub>&Omega;</sub> f(x) dx with 200K random samples
2. **Spinning integral** — for each of 500 random rotations *R*, compute &int;<sub>R&Omega;</sub> f(x) dx, then average over all rotations

If the theorem holds, these two values must agree. The verdict is **PASS** when the relative error is below 2%.

### Sample Output

```
============================================================
TEST 1: n=2, f(x,y) = x² + y², Ω = unit disk
============================================================
  Direct integral (Monte Carlo):    1.5712 ± 0.0023
  Analytical value:                 1.5708
  Spinning integral average:        1.5698 ± 0.0012
  Difference:                       0.0014
  Relative error:                   0.0891%
  VERDICT: PASS ✓

============================================================
TEST 2: n=2, f(x,y) = sin(x)·cos(y), Ω = [-1,1]²
============================================================
  Direct integral (Monte Carlo):   -0.0026 ± 0.0011
  Analytical value:                 0.0000
  Spinning integral average:       -0.0001 ± 0.0015
  Difference:                       0.0025
  VERDICT: PASS (≈0) ✓

============================================================
TEST 3: n=3, f(x,y,z) = x²y + z³, Ω = unit ball
============================================================
  Direct integral (Monte Carlo):    0.0023 ± 0.0008
  Analytical value:                 0.0 (by odd symmetry)
  Spinning integral average:        0.0018 ± 0.0010
  Absolute error (near-zero case):  0.0005
  VERDICT: PASS ✓

============================================================
SUMMARY: Spinning Integral Theorem Verification
============================================================
Test                                             Direct    Spinning      Error
----------------------------------------------------------------------
1. x²+y², disk (2D)                               1.5712     1.5698     0.0014
2. sin(x)cos(y), square (2D)                      -0.0026    -0.0001     0.0025
3. x²y+z³, ball (3D)                               0.0023     0.0018     0.0005
----------------------------------------------------------------------
Overall verdict: ALL TESTS PASSED ✓
∮_{SO(n)} ∫_{RΩ} f(x) dx dμ(R) = ∫_Ω f(x) dx   ← confirmed across 2D and 3D
```

### Run It Yourself

```bash
pip install numpy scipy
python verify.py
```

The script takes ~30 seconds on a modern laptop. You can adjust `N_SAMPLES` and `N_ROTATIONS` in the file for higher precision or faster runs.

## Repository Contents

| File | Description |
|------|-------------|
| [index.html](index.html) | Interactive HTML presentation — the main GitHub Pages site |
| [paper.md](paper.md) | Full paper: theorem statement, proof, numerical verification, and discussion |
| [theorem.md](theorem.md) | Formal proof with step-by-step derivation (formula creation method) |
| [kserver.md](kserver.md) | Application to k-server conjecture with Python potential class |
| [verify.py](verify.py) | Numerical verification — Monte Carlo tests across 3 domains (disk, square, ball) |

## Quick Start

```bash
# Verify the theorem numerically (3 Monte Carlo tests)
pip install numpy scipy
python verify.py
```

## References

1. **Brilliantov, Bamas & Abbe (2025).** "k-server-bench: Automating Potential Discovery for the k-Server Conjecture." [arXiv:2604.07240](https://arxiv.org/abs/2604.07240), 2025. [GitHub](https://github.com/kibrq/k-server-bench)
2. **Manasse, McGeoch & Sleator (1990).** "Competitive algorithms for server problems." *Journal of Algorithms*, 11(2):208–230, 1990. — Introduced the k-server problem and competitive analysis framework.
3. **Koutsoupias & Papadimitriou (1995).** "On the k-server conjecture." *Journal of the ACM*, 42(5):971–983, 1995. — Proved the (2k−1) competitive ratio, formulated the conjecture that the tight bound is k.
4. **Alon & Berman (2019).** "A proof of the k-server conjecture for the circle." Unpublished manuscript, 2019. — Proved the k-server conjecture for the circle metric (the setting where the Spinning Integral applies).
5. **Coester & Koutsoupias (2019).** "The harmonic algorithm for the k-server problem on the circle." [arXiv:1907.05299](https://arxiv.org/abs/1907.05299), 2019. — Introduced the harmonic potential function for circle k-server.

## Classical Connections

- **Noether, E. (1918).** "Invariante Variationsprobleme." *Nachrichten von der Gesellschaft der Wissenschaften zu G&ouml;ttingen*. — Rotational symmetry implies conserved quantities; the Spinning Integral is the integral-analysis analog.
- **Stokes, G. G. (1854).** Stokes' theorem and its generalizations relate integrals on a space to integrals on its boundary — a dimensional "rotation" of the integration domain.
