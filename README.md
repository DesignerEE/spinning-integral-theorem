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

- **arXiv 2604.07240** — [k-server conjecture benchmark framework](https://arxiv.org/abs/2604.07240)
- **k-server-bench** — [DesignerEE/k-server-bench](https://github.com/DesignerEE/k-server-bench) — LLM-driven potential function search for the k-server problem
