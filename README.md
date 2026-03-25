# SINDy-PI: Discovering Double Pendulum Dynamics from Data

Python implementation of **SINDy-PI** (Sparse Identification of Nonlinear Dynamics with Parallel Implicit models) applied to a compound double pendulum with viscous friction.

SINDy-PI discovers governing equations directly from time-series data — including systems with **rational structure** that standard SINDy cannot handle. The double pendulum is a natural test case because its mass matrix coupling produces equations with angle-dependent denominators.

## Discovered Equations

From 10 seconds of simulation data, SINDy-PI recovers the dominant dynamics with 5 sparse terms per equation and <1% one-step prediction error:

$$\ddot\phi_1 = \frac{-29.87\sin\phi_1 - 8.15\sin(\phi_1 - 2\phi_2) - 0.22\dot\phi_1^2\sin(2\phi_1 - 2\phi_2) - 0.32\dot\phi_2^2\sin(\phi_1 - \phi_2)}{1 - 0.44\cos^2(\phi_1 - \phi_2)}$$

$$\ddot\phi_2 = \frac{25.99\sin(2\phi_1 - \phi_2) - 24.29\sin\phi_2 + 1.37\dot\phi_1^2\sin(\phi_1 - \phi_2) + 0.22\dot\phi_2^2\sin(2\phi_1 - 2\phi_2)}{1 - 0.44\cos^2(\phi_1 - \phi_2)}$$

The shared $\cos^2(\phi_1 - \phi_2)$ denominator correctly captures the mass matrix coupling from the Lagrangian mechanics.

## How It Works

1. **Simulate** the double pendulum to generate training, test, and validation datasets
2. **Build a candidate library** of 39 terms: trig functions, velocity products, and acceleration-coupled terms
3. **Parallel implicit regression**: each library term is tested as a candidate left-hand side, with STLS (Sequential Thresholded Least Squares) finding sparse coefficients
4. **Model selection**: sweep sparsification thresholds, score on held-out test data
5. **Refinement**: prune noise-level coefficients and re-fit
6. **Validation**: simulate the discovered ODE from an unseen initial condition

## Results

| Metric | phi1_ddot | phi2_ddot |
|--------|-----------|-----------|
| Active terms (of 39) | 5 | 5 |
| One-step prediction error | 0.53% | 0.78% |
| Trajectory error @ 1s | 0.46% | 1.47% |
| Trajectory error @ 3s | 14.9% | 11.5% |

Short-term trajectory agreement is excellent; divergence beyond ~3s is expected for chaotic systems (Lyapunov instability). The notebook includes energy conservation checks, phase portraits, and lambda sweep diagnostics.

## Quick Start

```bash
conda create -n sindy python numpy scipy matplotlib sympy pandas -y
conda activate sindy
jupyter notebook SINDy_PI_Double_Pendulum.ipynb
```

## References

- Kaheman, K., Kutz, J.N., & Brunton, S.L. (2020). [SINDy-PI: a robust algorithm for parallel implicit sparse identification of nonlinear dynamics.](https://royalsocietypublishing.org/doi/10.1098/rspa.2020.0279) *Proceedings of the Royal Society A.*
- Original MATLAB implementation: [github.com/dynamicslab/SINDy-PI](https://github.com/dynamicslab/SINDy-PI)
