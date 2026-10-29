# Numerical verification and known limitations

This document distinguishes available plots and implementation features from independently verified numerical results. It is a **static review** of `PVL.py` and the current repository artifacts; the nonlinear simulation has **not** been rerun for this documentation update.

## Verification matrix

| Check | Expected evidence | Current status |
|---|---|---|
| Linear elastic junction displacement | Compare FEM and analytical $u_J=F/[E(A_1/L_1+A_2/L_2)]$ for a load below yield | Analytical reference available; code comparison pending |
| Reaction-force balance | Verify that the two end reactions balance the junction force for every converged increment | Numerical check pending |
| 1D element correctness | For an elastic element, compare element tangent to $EA/L\begin{bmatrix}1&-1\\-1&1\end{bmatrix}$ | Check pending |
| Local viscoplastic integration | Verify stress update and consistent tangent with an independent single-integration-point solver | Check pending |
| Material state/history | Track independent history at each element/integration point; commit history after global convergence | **Code correction recommended** |
| Global nonlinear equilibrium | Record residuals, correction norms, iteration counts and failed increments | Logging and failure handling pending |
| Time-step convergence | Compare selected response quantities under refinement of time increments | Study pending |
| Mesh convergence | Compare junction response at multiple bar discretizations | Archived figure exists; numerical tolerances/data provenance pending |

## Analytical elastic reference

Because both ends are fixed, the two bars act as two axial springs attached to the same junction. Their stiffnesses add:

$$
 k_1=EA_1/L_1=16{,}000\ \mathrm{N/mm},\qquad
 k_2=EA_2/L_2=16{,}000\ \mathrm{N/mm}.
$$

Therefore,

$$
 k_{\mathrm{junction}}=32{,}000\ \mathrm{N/mm},\qquad
 u_J(F=2100\ \mathrm N)=\frac{2100}{32000}=0.065625\ \mathrm{mm}.
$$

The forces carried by the two bars at this displacement have magnitudes $1{,}050\ \mathrm N$ each; the corresponding axial stress magnitudes are $131.25\ \mathrm{MPa}$ for Bar 1 and $65.625\ \mathrm{MPa}$ for Bar 2, both below the reference stress $160\ \mathrm{MPa}$. Thus this is a suitable **elastic** check when starting from zero plastic strain. Compare with actual solver output and set an error tolerance before declaring the check passed.

## Source-level issues to address before claiming quantitative validation

### 1. History variables are committed inside Newton iterations

`main()` stores just `ep_1` and `ep_2` for the two segments and updates them during element assembly. This means the state changes during Newton iterations, rather than treating the previous converged load step as a fixed history state while computing each trial iterate. In a nonlinear FEM solver, internal variables should generally be maintained **per material integration point**, recalculated from the *last converged* state at each Newton iterate, and committed only upon global convergence. Otherwise, the computed constitutive response can depend on the assembly order and number of Newton iterations.

### 2. Constitutive function does not use the supplied time increment

The signature is `material(strain, dt, ep)`, but the active-branch formulas currently use the global variable `h`. Replace this with the passed `dt` as part of a dedicated solver correction, and check all dependent expressions.

### 3. No explicit failed-increment handling

The Newton loop breaks when a residual or displacement-correction condition is satisfied, but the script does not emit a failure or reject/redo the increment if the maximum iteration count is exhausted. Convergence data and nonconvergence handling should be introduced before using responses as validated results.

### 4. Numerical outputs and reproducibility

`PVL.py` currently displays the response with `plt.show()`; the committed PNG figures are archived examples rather than automatically regenerated outputs from a documented, pinned environment. Record input configurations, package versions, commit SHA, plotting method and a numerical comparison against the elastic reference.

## Recommended acceptance sequence

1. Run and record the current elastic-only response without overwriting the archived results.
2. Compare the junction displacement at 2,100 N with the analytical value and document absolute/relative errors.
3. Implement integration-point state storage and *trial versus committed* constitutive updates; verify a one-element material test.
4. Add Newton iteration logs and explicit handling of failed load steps.
5. After corrections, regenerate the nonlinear plots and mesh/time-step sensitivity results.

**Terminology:** “Implementation exists” and “figure available” do not mean “numerically verified.” Mark a test as passed only when an actual run meets a specified acceptance criterion.
