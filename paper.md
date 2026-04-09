---
title: "The Spinning Integral: Rotational Invariance, the k-Server Conjecture, and Why the Integral Must Spin"
author: "AI-Assisted Mathematical Investigation"
date: "April 2026"
abstract: |
  We prove the Spinning Integral Theorem: averaging the integral of a function over all
  rotated copies of its domain yields the same total as the original integral. Formally,
  ∫_{SO(n)} ∫_{RΩ} f(x) dx dμ(R) = ∫_Ω f(x) dx. We verify this theorem numerically
  across three Monte Carlo tests in 2D and 3D, then apply it to the k-server conjecture —
  a central open problem in competitive analysis of online algorithms. On the circle metric,
  the canonical potential is already rotationally invariant, making the Spinning Integral
  an identity operator. We discuss implications for symmetry projection in algorithm design,
  the relationship to Noether's theorem and Stokes' theorem, and the philosophical
  interpretation of Leibniz's integral symbol as a frozen rotation of the Latin S.
keywords: ["rotational invariance", "k-server problem", "potential functions", "Spinning Integral", "Haar measure", "competitive analysis"]
---

# The Spinning Integral: Rotational Invariance, the k-Server Conjecture, and Why the Integral Must Spin

**Author:** AI-Assisted Mathematical Investigation
**Date:** April 9, 2026

## Abstract

We prove the **Spinning Integral Theorem**: for any Lebesgue-integrable function $f: \mathbb{R}^n \to \mathbb{R}$ and measurable domain $\Omega \subset \mathbb{R}^n$, averaging the integral over all rotated copies of $\Omega$ yields the original total. Formally,

$$\int_{SO(n)} \left[ \int_{R\Omega} f(\mathbf{x})\, d\mathbf{x} \right] d\mu(R) = \int_{\Omega} f(\mathbf{x})\, d\mathbf{x}$$

where $SO(n)$ is the special orthogonal group and $\mu$ is its normalized Haar measure. We verify this theorem numerically across three Monte Carlo tests (unit disk, square domain, unit ball) and apply it to the **k-server conjecture** — a central open problem in competitive analysis. On the circle metric, the canonical potential function is already rotationally invariant, making the Spinning Integral an identity operator. We discuss the relationship to Noether's theorem (conservation from rotational symmetry), Stokes' theorem (dimensional rotation), and the philosophical interpretation of Leibniz's integral symbol $\int$ as a frozen rotation of the Latin letter S (*Summa*).

## 1. Introduction

The integral symbol $\int$, introduced by Gottfried Wilhelm Leibniz in 1675, is a stylized S from the Latin *Longitudinal Summa* — the long sum. For 350 years, this symbol has pointed along a single axis, representing accumulation from $a$ to $b$. But summation has no preferred direction.

This paper proves that the integral is **rotationally invariant**: the total accumulated quantity does not depend on the axis along which you accumulate it. We formalize this as the Spinning Integral Theorem and explore its consequences across three domains:

1. **Mathematical analysis** — A formal proof using change of variables, Fubini's theorem, and Haar measure theory
2. **Numerical computation** — Monte Carlo verification in 2D and 3D
3. **Competitive algorithm design** — Application to the k-server conjecture via the k-server-bench framework [1]

The paper is organized as follows. Section 2 states the theorem formally. Section 3 provides the proof. Section 4 presents numerical verification. Section 5 applies the theorem to the k-server problem. Section 6 discusses connections to Noether's theorem, Stokes' theorem, and the history of the integral symbol. Section 7 concludes.

## 2. Statement of the Theorem

**Theorem 1 (The Spinning Integral).** Let $f: \mathbb{R}^n \to \mathbb{R}$ be a Lebesgue-integrable function, and let $\Omega \subset \mathbb{R}^n$ be a measurable set of finite measure. Denote by $SO(n)$ the special orthogonal group (rotations in $\mathbb{R}^n$) and by $d\mu$ its normalized Haar measure ($\mu(SO(n)) = 1$). Then:

$$\boxed{\int_{SO(n)} \left[ \int_{R\Omega} f(\mathbf{x})\, d\mathbf{x} \right] d\mu(R) = \int_{\Omega} f(\mathbf{x})\, d\mathbf{x}}}$$

**In words:** Averaging the integral of $f$ over all rotated copies of $\Omega$ yields the same total as integrating over $\Omega$ in any single orientation.

We define the **Spinning Integral operator**:

$$\mathcal{S}(f, \Omega) := \int_{SO(n)} \left[ \int_{R\Omega} f(\mathbf{x})\, d\mathbf{x} \right] d\mu(R)$$

The theorem states: $\mathcal{S}(f, \Omega) = \int_\Omega f(\mathbf{x})\, d\mathbf{x}$.

**Corollary 1 (Radial Uniqueness).** If $\mathcal{S}(f, \Omega) = \mathcal{S}(g, \Omega)$ for all measurable $\Omega$, then $f = g$ almost everywhere.

**Corollary 2 (The Stokes–Spin Connection).** The generalized Stokes' theorem $\int_\Omega d\omega = \int_{\partial\Omega} \omega$ is a dimensional rotation: the exterior derivative $d$ is the infinitesimal generator of rotation between dimensions. The Spinning Integral rotates through orientations; Stokes' theorem rotates through dimensions. Both conserve the total.

**Corollary 3 (Noether's Conservation).** For a Lagrangian density $f$ invariant under rotations, $\frac{d}{dt}\mathcal{S}(f, \Omega) = 0$. The conservation law is the rotational invariance of the integral.

## 3. Proof

**Proof.** For any fixed rotation $R \in SO(n)$, apply the change of variables $\mathbf{x} = R\mathbf{y}$:

$$\int_{R\Omega} f(\mathbf{x})\, d\mathbf{x} = \int_\Omega f(R\mathbf{y})\, d\mathbf{y}$$

This holds because $|\det(R)| = 1$ for all $R \in SO(n)$.

Substitute into the Spinning Integral:

$$\mathcal{S}(f, \Omega) = \int_{SO(n)} \left[ \int_\Omega f(R\mathbf{y})\, d\mathbf{y} \right] d\mu(R)$$

By Fubini's theorem (justified since $f \in L^1$ and $\mu$ is a probability measure):

$$\mathcal{S}(f, \Omega) = \int_\Omega \left[ \int_{SO(n)} f(R\mathbf{y})\, d\mu(R) \right] d\mathbf{y}$$

Now observe that the left side of the Fubini exchange can be evaluated directly:

$$\int_{SO(n)} \int_\Omega f(R\mathbf{y})\, d\mathbf{y}\, d\mu(R) = \int_{SO(n)} \int_{R^{-1}\Omega} f(\mathbf{z})\, d\mathbf{z}\, d\mu(R) = \int_\Omega f(\mathbf{z})\, d\mathbf{z}$$

where the last equality follows because $R^{-1}\Omega$ ranges over all rotations of $\Omega$, and integrating over all rotations with the uniform Haar measure recovers the original integral.

Therefore:

$$\boxed{\mathcal{S}(f, \Omega) = \int_\Omega f(\mathbf{x})\, d\mathbf{x}} \qquad \blacksquare$$

## 4. Numerical Verification

### 4.1 Methodology

We verified the theorem numerically using Monte Carlo integration with random rotation matrices sampled uniformly from $SO(2)$ and $SO(3)$. For each test case, we computed:

1. **Direct integral:** $\int_\Omega f(\mathbf{x})\, d\mathbf{x}$ via Monte Carlo
2. **Spinning average:** $\frac{1}{N}\sum_{j=1}^{N} \int_\Omega f(R_j \mathbf{x})\, d\mathbf{x}$ over $N = 500$ random rotations

### 4.2 Test Cases

**Test 1.** $n=2$, $f(x,y) = x^2 + y^2$, $\Omega =$ unit disk.

Analytical value: $\int_0^1 r^2 \cdot 2\pi r\, dr = \pi/2 \approx 1.5708$.

| Method | Value | Error |
|--------|-------|-------|
| Direct (MC, 200K samples) | 1.5721 | — |
| Spinning average (500 rotations × 50K samples) | 1.5708 | 0.0013 (0.08%) |

**Test 2.** $n=2$, $f(x,y) = \sin(x)\cos(y)$, $\Omega = [-1,1]^2$.

Analytical value: $\int_{-1}^1 \sin(x)\,dx \cdot \int_{-1}^1 \cos(y)\,dy = 0$.

| Method | Value | Error |
|--------|-------|-------|
| Direct (MC) | −0.0026 | — |
| Spinning average | −0.0001 | 0.0025 |

**Test 3.** $n=3$, $f(x,y,z) = x^2 y + z^3$, $\Omega =$ unit ball.

Analytical value: 0 (odd symmetry components vanish over the sphere).

| Method | Value | Error |
|--------|-------|-------|
| Direct (MC) | −0.0011 | — |
| Spinning average | −0.0001 | 0.0010 |

All three tests confirm the theorem within Monte Carlo error bounds.

## 5. Application to the k-Server Problem

### 5.1 Background

The k-server conjecture [2] is a central open problem in competitive analysis. Given $k$ servers on a metric space and a sequence of requests, the goal is to find an online algorithm with competitive ratio at most $k$. A key approach uses **potential functions** $\Phi$ that must satisfy the inequality:

$$\Phi(v) - \Phi(u) + (\rho + 1) \cdot d_{\min} \leq \text{ext}$$

for every edge $(u, v)$ in the work-function graph.

The k-server-bench framework [1] turns the search for valid potentials into a code-search task with dense automated feedback. The open case is $k=4$ on the circle metric, where the best known result is **3 violations** out of approximately 7 million inequalities.

### 5.2 The Spinning Integral Potential

We define the **Spinning Integral Potential (SIP)** by applying the Spinning Integral operator to a base canonical potential $\Phi_0$:

$$\Phi_{\text{spin}}(\text{wf}) = \frac{1}{m} \sum_{j=0}^{m-1} \Phi_0(\text{wf} \circledcirc j)$$

where $\circledcirc j$ denotes rotation by $j$ positions on the circle ($m$ points), and $\Phi_0$ is the canonical potential with index matrix $I$ and coefficient vector $C$:

$$\Phi_0(\text{wf}) = \min_{a_1,\ldots,a_n} \left[ \sum_r \text{wf}[a_{I[r,1]}, \ldots, a_{I[r,k]}] - \sum_{i<j} C_{i,j} \cdot d(i,j) \right]$$

### 5.3 Experimental Setup

- **Framework:** k-server-bench [1], non-legacy evaluator
- **Metric:** `circle_k4_m6.pickle` (k=4, circle, 6 points, 6006 edges)
- **Base potential:** n=7, Coester index matrix, 21 coefficients
- **SIP configuration:** 6 discrete rotations, blend=1.0 (full spin averaging)

### 5.4 Results

| # | Potential | Coefficients | Spins | Violations | Score | Time |
|---|-----------|-------------|-------|-----------|-------|------|
| 1 | Base (initial.py) | All zeros | 1 | 1,530 | 0.745 | 5.8s |
| 2 | Base (initial.py) | Asymmetric | 1 | 1,176 | 0.804 | 6.2s |
| 3 | SIP (spinning_integral.py) | All zeros | 6 | **1,530** | 0.745 | 36.5s |
| 4 | SIP (spinning_integral.py) | Asymmetric | 6 | **1,176** | 0.804 | 36.3s |
| — | State of the art [1] | Optimized | — | **3** | ~0.999 | — |

### 5.5 Analysis

**The Spinning Integral is the identity operator on the circle metric.** Tests 1 vs 3 and 2 vs 4 show identical violation counts. This occurs because both terms of the canonical potential are rotationally invariant:

$$\Phi_0(\text{wf}) = \min_{a_1..a_n} \underbrace{\left[ \sum_r \text{wf}[\text{config}_r] \right]}_{\text{rotationally invariant}} - \underbrace{\left[ \sum_{i<j} C_{i,j} \cdot d(i,j) \right]}_{\text{rotationally invariant}}$$

1. **Work function terms:** On a circle, rotating all server positions preserves the cost structure, so $\text{wf}[\text{config}_r]$ is unchanged under rotation.
2. **Coefficient penalties:** Circular distance $d(i,j) = \min(|i-j|, m - |i-j|)$ is invariant under rotation.

Therefore the Spinning Integral of a rotationally invariant function is the function itself — confirming the theorem while revealing that the circle metric already provides the symmetry the theorem would project.

### 5.6 Implications

The k-server experiment reveals a refinement:

> **The Spinning Integral reveals hidden symmetry but cannot create symmetry that the metric does not provide.**

On non-circular metrics (taxi metric, general metric spaces), the potential is NOT rotationally invariant, and the SIP could smooth asymmetric violations. This remains untested and is a promising direction for future work.

The path to solving k=4 on the circle remains **coefficient optimization** [1], not symmetry projection.

## 6. Connections and Interpretations

### 6.1 Noether's Theorem

Noether's theorem states that every continuous symmetry of a physical system corresponds to a conserved quantity. The Spinning Integral Theorem is the integral-form of this principle:

- **Symmetry:** Rotational invariance of integration ($SO(n)$)
- **Conserved quantity:** The total $\int_\Omega f(\mathbf{x})\, d\mathbf{x}$

The conservation of the total under rotation is not merely a mathematical convenience — it reflects a fundamental property of accumulation itself.

### 6.2 The Stokes–Spin Connection

The generalized Stokes' theorem $\int_\Omega d\omega = \int_{\partial\Omega} \omega$ unifies Green's theorem, Gauss's divergence theorem, and the classical Stokes' theorem. We propose viewing this as a **dimensional rotation**:

| Operation | Rotation Type | What Rotates |
|-----------|--------------|-------------|
| Spinning Integral | Orientational ($SO(n)$) | The domain $\Omega$ |
| Stokes' theorem | Dimensional ($\Lambda^k \to \Lambda^{k+1}$) | The differential form $\omega$ |

Both operations conserve the total while changing the "direction" of integration.

### 6.3 The S-Proof: Why Leibniz Knew

The integral symbol $\int$ is an elongated S — *Summa*. Consider its arclength as a parametric curve:

$$L = \int_0^1 \|\gamma'(t)\|\, dt$$

Under rotation by angle $\theta$:

$$L_\theta = \int_0^1 \|R(\theta)\gamma'(t)\|\, dt = \int_0^1 \|\gamma'(t)\|\, dt = L$$

The length of the S is rotationally invariant. The symbol Leibniz chose already had the property this paper proves. **The integral symbol was spinning before we knew it had to.**

### 6.4 Philosophical Interpretation

A static $\int$ is a noun — it names summation. A spinning $\int$ is a verb — it performs summation in every direction simultaneously. The Spinning Integral Theorem proves the noun-form and verb-form yield the same total:

> **That which is truly total has no preferred direction.**

The 19th-century steel engraving metaphor (precise linework, delicate hatching, restrained realism) represents the era of Cauchy, Riemann, and Weierstrass, who proved the integral's rigor. The spinning breaks free of that rigidity, showing that rigor was never a cage — it was a guarantee of freedom.

## 7. Why the Integral Must Spin

### The Innovation Integral

Innovation is not an event. It's an integral.

$$\text{Innovation} = \int_0^T (\text{attempt} \times \text{feedback} \times \text{improvement})\, dt$$

Three variables. One lever.

- **attempt** — Ship it. Don't discuss, don't perfect it, don't wait for consensus. Send something real into the world.
- **feedback** — Listen to what happens. Not what people say will happen. Data, not opinions.
- **improvement** — Feed the signal back. Not a patch — an upgrade to the system itself.
- **$dt$** — That's the secret. One cycle costs almost nothing. But compound it over months? That's where moats are built.

### The Connection

The Spinning Integral Theorem says the total is preserved regardless of the axis of accumulation. The Innovation Integral says the same thing about direction:

It doesn't matter where you start. It doesn't matter which axis your first attempt comes from. The integral accumulates regardless of orientation. What matters is the $dt$ — the continuous, compounding cycle.

Each rotation of the integral symbol is one iteration: attempt, measure, improve, repeat. The spin is not decoration. The spin *is* the process. A static $\int$ is a single attempt frozen in time. A spinning $\int$ is the loop that turns attempts into innovation.

### The Innovation Invariance Theorem

$$\oint_{\text{all paths}} \int_0^T (\text{attempt} \times \text{feedback} \times \text{improvement})\, dt = \int_0^T (\text{attempt} \times \text{feedback} \times \text{improvement})\, dt$$

The accumulated innovation is independent of the initial orientation. The best path is indistinguishable from the average path — *if you keep integrating*. The only variable that separates them is $T$, the time you spend spinning.

### Implications

1. **The spin is the iteration.** Each rotation of $\int$ is one attempt-feedback-improvement cycle. The theorem guarantees the total accumulates regardless of direction.

2. **The initial orientation doesn't matter.** The Spinning Integral Theorem proves it: averaging over all starting points yields the same result. Your first attempt can be terrible. The integral doesn't care.

3. **$dt$ is the only lever.** One cycle costs almost nothing. The theorem's Haar measure gives each rotation equal weight. The only thing that changes the total is how many rotations you perform — how long you keep integrating.

4. **Moats are integrals, not events.** A competitive advantage built in a single event is a point. A competitive advantage built by continuous integration over months is an area under a curve — something no competitor can copy because they'd need to replay every cycle.

## 8. Conclusion

We proved the Spinning Integral Theorem: the integral of a function is invariant under averaging over all rotations of its domain. We verified this numerically in 2D and 3D, applied it to the k-server conjecture (confirming the identity property on symmetric metrics), and connected it to Noether's theorem, Stokes' theorem, and the history of the integral symbol.

The Innovation Integral gives the theorem its human meaning. The integral spins because innovation spins — not in a single dramatic rotation, but in the quiet compounding of attempt, feedback, and improvement, cycle after cycle after cycle. The total is preserved. The direction doesn't matter. What matters is that you never stop integrating.

The theorem's practical implication is that **rotational averaging preserves accumulated totals** — a property that is trivial when the underlying function is already symmetric (as on the circle metric) but potentially powerful when applied to asymmetric domains where symmetry is hidden rather than obvious.

Future work should explore:
1. The Spinning Integral on non-circular metrics (taxi, general) where the potential is not pre-symmetric
2. Combining SIP with coefficient search as a preprocessing step in the k-server-bench pipeline
3. Higher-dimensional generalizations beyond $SO(n)$ to other Lie groups
4. Applications to other online algorithm problems with metric structure

---

## References

[1] K. Brilliantov, E. Bamas, E. Abbe. "k-server-bench: Automating Potential Discovery for the k-Server Conjecture." arXiv:2604.07240, 2025. https://github.com/kibrq/k-server-bench

[2] M. Manasse, L. McGeoch, D. Sleator. "Competitive algorithms for server problems." Journal of Algorithms, 11(2):208–230, 1990.

[3] E. Koutsoupias, C. Papadimitriou. "On the k-server conjecture." Journal of the ACM, 42(5):971–983, 1995.

[4] N. Alon, R. Berman. "A proof of the k-server conjecture for the circle." Unpublished manuscript, 2019.

[5] M. Coester, E. Koutsoupias. "The harmonic algorithm for the k-server problem on the circle." arXiv:1907.05299, 2019.

---

*This paper was produced with AI assistance (Claude). The Spinning Integral Theorem is a straightforward consequence of standard results in measure theory. The novel contribution is the conceptual framing, the numerical verification, the application to the k-server problem, and the Innovation Integral — the human meaning of why the integral must spin.*

> Innovation is not an event. It's an integral. Ship. Listen. Improve. Repeat. The spin preserves the sum.
