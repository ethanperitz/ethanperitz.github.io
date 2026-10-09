# Independent Scientific Computing Review Sample
## Estimating a Cooling Rate from Noisy Measurements

**Prepared by Ethan Peritz**

This exercise was created independently for an application portfolio. The initial solution is an intentionally constructed illustration, not an actual AI-generated response. No proprietary evaluation material is included.  The purpose of this project is to illustrate realistic AI evaluation techniques for mathematic and scientific problem-solving. The error was intentionally devised to be one that would plausibly be committed by an AI agent. 

## 1. Problem Statement

An object cools according to Newton's law of cooling:

$$
T(t)=T_{\text{ambient}}+(T_0-T_{\text{ambient}})e^{-kt}.
$$

Here:

- $t$ is elapsed time in minutes.
- $T(t)$ is the object's temperature in degrees Celsius.
- $T_{\text{ambient}}$ is the known, constant ambient temperature.
- $T_0>T_{\text{ambient}}$ is the known initial temperature.
- $k>0$ is the unknown cooling constant, in inverse minutes.

**Important:**  Observed temperatures contain additive measurement noise, adding a stochastic component to an otherwise mathematical modeling problem. 

$$
y_i=T(t_i)+\varepsilon_i.
$$

Assume the errors are independent, have mean zero, and have equal variance. Measurements may fall at or below ambient because of noise.

Write a Python function that estimates $k$ by minimizing the sum of squared temperature residuals:

$$
S(k)=\sum_i\left[T_{\text{ambient}}+(T_0-T_{\text{ambient}})e^{-kt_i}-y_i\right]^2.
$$

Also predict when the modeled temperature reaches a specified target.

### Inputs

- `times`: a nonempty, one-dimensional array of finite, nonnegative measurement times, with at least one positive time.
- `temperatures`: a matching one-dimensional array of finite observed temperatures.
- `ambient`: the known, finite ambient temperature.
- `initial`: the known, finite initial temperature, greater than `ambient`.
- `target`: a finite temperature strictly between `ambient` and `initial`.

### Outputs

- `k`: the estimated positive cooling constant.
- `target_time`: the predicted elapsed time to reach `target`.

Raise `ValueError` for invalid inputs. Use Python with NumPy and SciPy.

**Boundary behavior:** Valid measurements do not always identify a finite, strictly positive best-fit rate. Report a fitting failure if a positive rate cannot be resolved, rather than presenting a boundary estimate as a successful cooling fit.

## 2. Initial Solution

> **Illustrative only:** This intentionally flawed response was constructed for the review sample. It is not an actual AI-generated response.

Taking logarithms makes the noiseless cooling model linear:

$$
\log\left(\frac{T(t)-T_{\text{ambient}}}{T_0-T_{\text{ambient}}}\right)=-kt.
$$

The proposed solution estimates $k$ by fitting a line through the origin to transformed observations, then rearranges the cooling equation to calculate the target time.

```python
import numpy as np


def estimate_cooling(times, temperatures, ambient, initial, target):
    times = np.asarray(times, dtype=float)
    temperatures = np.asarray(temperatures, dtype=float)

    transformed = np.log(
        (temperatures - ambient) / (initial - ambient)
    )

    # Least-squares slope through the origin is -k.
    k = -np.dot(times, transformed) / np.dot(times, times)

    target_time = -np.log(
        (target - ambient) / (initial - ambient)
    ) / k

    return float(k), float(target_time)
```

The algebra is correct for noiseless model values. The review below focuses on applying that transformation to noisy measurements. Missing input validation is a separate defect.

## 3. Error Demonstration: Fitting the Wrong Noise Model

The initial solution applies a transformation that is exact for noiseless temperatures directly to noisy observations.

Substituting $y_i=T(t_i)+\varepsilon_i$ into the transformation gives:

$$
\log\left(\frac{y_i-T_{\text{ambient}}}{T_0-T_{\text{ambient}}}\right)
=-kt_i+\log\left(1+\frac{\varepsilon_i}{T(t_i)-T_{\text{ambient}}}\right).
$$

This expression is defined only when the observed temperature exceeds ambient. The extra term depends on both the measurement error and the temperature's distance from ambient. The same absolute temperature error has a much larger effect near ambient.

For small errors relative to the temperature excess, the transformed error is approximately:

$$
\frac{\varepsilon_i}{T(t_i)-T_{\text{ambient}}}.
$$

Its approximate variance therefore increases as the temperature excess decreases. The transformed errors also need not have mean zero. Equal-variance, zero-mean errors on the temperature scale do not retain that structure after transformation.

### Concrete Dataset

Let ambient temperature be $20^\circ\mathrm{C}$, initial temperature be $80^\circ\mathrm{C}$, and the true cooling constant be $k=0.1$.

| Time (minutes) | True temperature (°C) | Added error (°C) | Observed temperature (°C) |
|---:|---:|---:|---:|
| 5 | 56.392 | +1 | 57.392 |
| 10 | 42.073 | -1 | 41.073 |
| 20 | 28.120 | +1 | 29.120 |
| 30 | 22.987 | -2 | 20.987 |

These errors are an illustrative realization, not a claim that this four-element sample itself has exactly zero mean. Every observation is above ambient, so all logarithms are valid.

At 30 minutes, the true excess above ambient is approximately $2.987^\circ\mathrm{C}$, but the observed excess is only $0.987^\circ\mathrm{C}$. The logarithm makes this modest absolute error disproportionately influential.

Using the unrounded observations:

| Quantity | Initial log-linear solution | Fit to original temperature residuals |
|---|---:|---:|
| Estimated $k$ | 0.12191 | 0.10047 |
| Relative error against true $k=0.1$ | +21.9% | +0.47% |
| Sum of squared temperature residuals | 49.363 | 6.974 |
| Predicted time to reach $30^\circ\mathrm{C}$ | 14.70 minutes | 17.83 minutes |

The true target time is:

$$
t_{\text{target}}=\frac{\log(60/10)}{0.1}\approx17.92\text{ minutes}.
$$

The initial solution predicts arrival more than three minutes too early.

### Review Finding

**Defect location:** Mathematical method in the submitted solution.

**Defect:** The submission minimizes squared residuals in log-transformed temperature excess, while the specification requires squared residuals in temperature.

**Required correction:** Fit the original exponential model directly to the observations.

This example demonstrates the consequences of that mismatch. It does not imply that the corrected method recovers the true parameter more accurately for every noisy dataset, particularly one that violates the assumptions of independent variance. A transformed regression explicitly posed with a different noise model could be appropriate for a different problem.

## 4. Corrected Approach: Nonlinear Temperature Residuals

For each candidate $k$, define:

$$
r_i(k)=T_{\text{ambient}}+(T_0-T_{\text{ambient}})e^{-kt_i}-y_i.
$$

Estimate $k$ by minimizing $\sum_i r_i(k)^2$. This keeps observations on their original temperature scale and implements the specified objective.

### Accounting for Noise

Individual measurement errors are unknown; they cannot simply be subtracted. Instead, choose the model that minimizes the total squared discrepancy.

- With independent, zero-mean errors of equal variance, each observation receives equal weight.
- If the errors are also normally distributed, minimizing this objective is maximum-likelihood estimation.
- Observations near ambient are not amplified by a logarithmic transformation, a crucial change from the log-linear model.
- Observations at or below ambient remain valid data. The model stays above ambient, but noise may place individual measurements below it.

### Core Python Implementation

This implementation demonstrates the corrected fitting method **for valid inputs**. It is not a complete reference solution: the input validation required by Section 1 remains to be implemented, and optimizer success alone does not certify a global optimum or parameter identifiability.

```python
import numpy as np
from scipy.optimize import least_squares


def estimate_cooling(times, temperatures, ambient, initial, target):
    times = np.asarray(times, dtype=float)
    temperatures = np.asarray(temperatures, dtype=float)
    excess = initial - ambient

    def residuals(parameters):
        k = parameters[0]
        predicted = ambient + excess * np.exp(-k * times)
        return predicted - temperatures

    def jacobian(parameters):
        k = parameters[0]
        derivative = -excess * times * np.exp(-k * times)
        return derivative[:, None]

    # Initial guess: one characteristic decay over the observed time span.
    k_start = 1.0 / np.max(times)

    result = least_squares(
        residuals,
        x0=[k_start],
        jac=jacobian,
        bounds=([0.0], [np.inf]),
        loss="linear",  # Ordinary squared residuals.
    )

    if not result.success:
        raise RuntimeError(f"Cooling fit failed: {result.message}")

    k = float(result.x[0])

    # A boundary estimate does not establish positive cooling.
    if result.active_mask[0] == -1 or k <= 0:
        raise RuntimeError(
            "The fit reached k = 0; a positive cooling rate was not resolved."
        )

    target_time = np.log(excess / (target - ambient)) / k
    return k, float(target_time)
```

The analytical Jacobian is:

$$
\frac{\partial r_i}{\partial k}
=-(T_0-T_{\text{ambient}})t_i e^{-kt_i}.
$$

It tells the optimizer how the predictions change with $k$, avoiding finite-difference approximation of that derivative.

### Reproducing the Numerical Comparison

Run this block with the corrected `estimate_cooling` function above. Constructing observations from the model avoids discrepancies caused by rounding the displayed table.

```python
ambient = 20.0
initial = 80.0
true_k = 0.1
target = 30.0

times = np.array([5.0, 10.0, 20.0, 30.0])
true_temperatures = ambient + (initial - ambient) * np.exp(-true_k * times)
observed = true_temperatures + np.array([1.0, -1.0, 1.0, -2.0])

transformed = np.log((observed - ambient) / (initial - ambient))
log_k = -np.dot(times, transformed) / np.dot(times, times)
log_target_time = np.log((initial - ambient) / (target - ambient)) / log_k

fitted_k, fitted_target_time = estimate_cooling(
    times, observed, ambient, initial, target
)


def temperature_sse(k):
    predictions = ambient + (initial - ambient) * np.exp(-k * times)
    return np.sum((predictions - observed) ** 2)


print("Log-linear fit:", log_k, log_target_time, temperature_sse(log_k))
print("Nonlinear fit:", fitted_k, fitted_target_time, temperature_sse(fitted_k))

# Noiseless recovery check; exact model parameters provide an independent target.
noiseless_k, noiseless_target_time = estimate_cooling(
    times, true_temperatures, ambient, initial, target
)
expected_target_time = np.log(6.0) / true_k
assert np.isclose(noiseless_k, true_k, rtol=1e-6, atol=1e-9)
assert np.isclose(
    noiseless_target_time, expected_target_time, rtol=1e-6, atol=1e-8
)
```

The noisy comparison is a demonstration, not a universal accuracy test. The noiseless assertions check parameter recovery and the independently calculated target time; their tolerances allow small floating-point and optimizer differences while remaining substantially tighter than the displayed precision.

### Uncertainty and Further Validation

A fitted parameter is an estimate, not exact recovery of the underlying rate. Inspect residuals for systematic patterns and changing spread (i.e. heteroskedasticity); either may indicate that the cooling model or noise assumptions need revision. Multiple starting guesses or inspection of the one-dimensional objective can help assess whether the fitted minimum is stable.

If observations have known, different measurement standard deviations $\sigma_i$, minimize standardized residuals:

$$
\sum_i\left(\frac{r_i(k)}{\sigma_i}\right)^2.
$$

More precise measurements then receive greater weight. Equal variance is assumed in this exercise, so unweighted least squares is the appropriate correction.

Before presenting this as a complete reference implementation, add input validation and tests for mismatched array shapes, nonfinite inputs, negative times, all-zero times, invalid targets, observations below ambient, and unresolved boundary fits. Tests should accept different valid numerical implementations rather than requiring an identical optimizer trajectory.
