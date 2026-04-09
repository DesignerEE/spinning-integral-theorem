# Spinning Integral Theorem

**A new integral theorem with proof, numerical verification, and application to the k-server conjecture.**

**Live page:** [https://designeree.github.io/spinning-integral-theorem/](https://designeree.github.io/spinning-integral-theorem/)

## Overview

The Spinning Integral Theorem establishes a relationship between rotated coordinate systems and integral evaluation, with direct applications to competitive analysis of the k-server problem. This repository contains the complete proof, numerical verification, and k-server benchmark results.

## Files

| File | Description |
|------|-------------|
| [index.html](index.html) | Interactive HTML presentation of the theorem (GitHub Pages main page) |
| [paper.md](paper.md) | Full paper with mathematical exposition |
| [theorem.md](theorem.md) | Formal statement and proof of the theorem |
| [kserver.md](kserver.md) | Application to the k-server conjecture benchmark |
| [verify.py](verify.py) | Numerical verification script (Python) |

## References

- **arXiv Paper:** [2604.07240](https://arxiv.org/abs/2604.07240) — k-server conjecture benchmark framework
- **k-server-bench:** [DesignerEE/k-server-bench](https://github.com/DesignerEE/k-server-bench) — LLM-driven potential function search

## Quick Start

```bash
# Run numerical verification
pip install numpy scipy
python verify.py
```
