---
title: "Exercise C1 — Superposition, Fields, and Potentials"
subtitle: "Computational companion to Jackson Ch. 1–2"
author: "R. A. de Souza"
date: "August 2026"
geometry: "left=2.5cm,right=2.5cm,top=2.5cm,bottom=2.5cm"
header-includes:
  - \usepackage{amsmath}
  - \usepackage{amssymb}
---

# Exercise C1 — Superposition, Fields, and Potentials

**Companion to:** Jackson §§1.1–1.7, 2.1–2.6
**Estimated effort:** 3–5 hours, splittable into parts
**Prerequisites:** none beyond Jackson Ch. 1. Parts (f) and (h) use `np.gradient`.
**Deliverable:** one Python module plus a short notes file answering the questions in *italics*.

---

## Statement

Build a tool that computes and visualises the electrostatic field and potential of an arbitrary set of point charges in a plane, and use it to verify — numerically — the structural properties of electrostatics that Jackson Chapter 1 asserts analytically.

The point is not the plots. The point is that every claim in Chapter 1 ($\mathbf{E} = -\nabla V$, $\nabla\times\mathbf{E} = 0$, $\nabla\cdot\mathbf{E} = 0$ away from sources, superposition) becomes something you can *check* rather than something you accept. Write the code so that each check is a function returning a number, not an impression.

Work in Gaussian-like units with $k = 1$ throughout. Physical constants add nothing here and obscure the checks.

---

## Part (a) — Baseline: one charge

1. Set up a square grid on $[-L, L]^2$ using `meshgrid`. Keep the resolution modest at first (say $101\times101$); you will want it coarse for arrows and fine for contours, so make it a parameter.
2. Write a function that returns $(E_x, E_y)$ for a single charge $q$ at position $(x_0, y_0)$, evaluated on the whole grid at once. No Python loops over grid points.
3. Plot it with `quiver`.

**Trap to avoid.** $\mathbf{E} = kq\,\hat{\mathbf{r}}/r^2$ and $\hat{\mathbf{r}} = \mathbf{d}/r$, so the expression you code is $kq\,\mathbf{d}/r^3$ — cubed, with the *unnormalised* displacement in the numerator. Normalising separately and then dividing by $r^2$ works but is slower and is where sign errors hide.

**Checkpoint.** Evaluate your field at a point a distance $d$ from the charge along the $x$-axis and confirm by hand arithmetic that you get $q/d^2$ pointing the right way. Do this for both signs of $q$. If this fails, nothing downstream will work.

---

## Part (b) — Superposition

1. Generalise to a list of charges: `charges = [(q1, x1, y1), (q2, x2, y2), ...]`. The function should accumulate contributions in a loop over *charges* — never over grid points.
2. Produce three configurations:
   - **Dipole:** $+q$ at $(-a, 0)$, $-q$ at $(+a, 0)$
   - **Like pair:** $+q$ at both $(\pm a, 0)$
   - **Asymmetric:** $+2q$ at $(-a,0)$, $-q$ at $(+a, 0)$

**Before you run anything:** for each of the three configurations, predict by hand the field at the origin — magnitude and direction, or zero. Write your predictions down first. Then check them numerically.

*In words: your code contains no logic specific to "two charges." Why not? Which property of Maxwell's equations are you relying on, and where in Chapter 1 does Jackson state it?*

---

## Part (c) — The singularity

Your plots from (b) are almost certainly unreadable: one or two grid cells near each charge have a field many orders of magnitude larger than everywhere else, and the arrow scaling collapses.

Implement **two** independent mitigations and compare them:

1. **Softening.** Replace $r^2 \to r^2 + \epsilon^2$ in the denominator, with $\epsilon$ a small length. Make $\epsilon$ a parameter.
2. **Masking.** Set the field to `nan` at grid points within some radius of a charge, and let matplotlib skip them.

**Checkpoint.** With softening on, evaluate the field at a distance $d \gg \epsilon$ from an isolated charge and confirm the answer is unchanged from part (a) to within your tolerance. Then evaluate at $d \sim \epsilon$ and watch it depart. Find the value of $d/\epsilon$ at which the softened field differs from the true field by 1%.

*In words: softening does not merely suppress a numerical annoyance — the softened expression is the exact field of a particular (non-point) charge distribution. What sort of distribution? You do not need to derive it; describe qualitatively what changes. (You will meet this idea again as regularisation.)*

---

## Part (d) — The potential

1. Write a function returning the scalar potential $V = \sum_i q_i / |\mathbf{x} - \mathbf{x}_i|$ on the grid. Note that this is much easier than the field — no vector bookkeeping, and it's a sum of scalars.
2. Plot it as a filled contour or heatmap. Use a symmetric colour scale centred on zero for the dipole case.

**Trap.** $V$ spans a huge dynamic range and changes sign. A linear colour scale will show you a white field with two dots. Use `np.sign(V) * np.log1p(np.abs(V))` or clip the range, and say in a comment which you chose and why.

**Checkpoint.** For the dipole configuration, find the locus of points where $V = 0$. Predict it analytically first — it is a simple geometric object — then confirm your contour plot shows it.

*In words: the field at the origin for the dipole is nonzero, but the potential there is zero. Explain why there is no contradiction.*

---

## Part (e) — The central check: $\mathbf{E} = -\nabla V$

This is the part that makes the exercise worth doing.

1. Take the potential array from (d) and compute $-\nabla V$ numerically with `np.gradient`. Mind the argument order — `np.gradient` returns derivatives along array axes, and with `meshgrid` the axis order may not be the one you expect.
2. Compare component-by-component against the analytic $(E_x, E_y)$ from (b).
3. Report a quantitative agreement measure: the maximum relative error over all grid points more than some distance $R$ from any charge, as a function of $R$ and of grid spacing $h$.

**Checkpoint.** The error should scale as $h^2$ for a centred difference. Halve the grid spacing and confirm the error drops by roughly a factor of four. **If it doesn't, you have a bug — most likely an axis-order mix-up or an off-by-one in the mask.** This convergence test is the single most valuable habit in the whole exercise; it distinguishes "the plot looks plausible" from "the code is correct."

*In words: you have now verified $\mathbf{E} = -\nabla V$ numerically. What would have to be true of the physics for this relation to fail? Name a situation, later in Jackson, where it does.*

---

## Part (f) — Field lines and equipotentials together

1. Overlay `streamplot` of $(E_x, E_y)$ on `contour` of $V$, choosing contour levels so they're roughly evenly spaced in the visible region.
2. Do this for all three configurations from (b).

**Checkpoint.** Field lines should cross equipotentials at right angles, everywhere, in all three plots. Verify this *numerically* rather than by eye: at a sample of grid points, compute the angle between $\mathbf{E}$ and the local tangent to the equipotential (equivalently, check that $\mathbf{E}$ is parallel to $\nabla V$ up to sign) and report the maximum deviation from $90°$.

*In words: why is this orthogonality guaranteed rather than coincidental? One sentence, from the definition of the gradient.*

*Second question: where the contour lines bunch together, what is the field doing? Relate this to why a "field line density" picture is a legitimate visualisation and not just an artistic convention.*

---

## Part (g) — Two integral checks

1. **Circulation.** Pick a closed rectangular loop in a region containing no charges. Numerically evaluate $\oint \mathbf{E}\cdot d\boldsymbol\ell$ around it. Repeat for a loop that *encloses* a charge.
2. **Divergence.** Compute $\nabla\cdot\mathbf{E}$ numerically over the grid (two applications of `np.gradient`).

**Checkpoint.** State what you expect for each *before* computing, then check. For the circulation, both loops should give the same answer, and the answer should be one specific number. For the divergence, you should get approximately zero everywhere except in the immediate neighbourhood of the charges — where, with softening on, you will see a smooth blob rather than a spike.

*In words: the divergence result is a numerical picture of $\nabla\cdot\mathbf{E} = 4\pi\rho$. Given that you fed in point charges, what does the blob's finite width represent? Connect this back to part (c). Then: integrate the numerical $\nabla\cdot\mathbf{E}$ over a small region around one charge and see what you get.*

---

## Part (h) — The dipole limit

1. Fix the product $p = 2qa$ and take a sequence of configurations with $a \to 0$, $q \to \infty$.
2. For each, measure the field magnitude along the axis at several distances $r \gg a$ and fit the power law $|\mathbf{E}| \propto r^{-n}$.

**Checkpoint.** Two things to observe. First, $n$ should converge to a specific integer as $a$ shrinks — one different from the single-charge case. Second, at *fixed* $a$, the exponent you measure should depend on $r$: near-field behaves one way, far-field another. Find the crossover distance and see how it scales with $a$.

3. Compare the on-axis and equatorial field magnitudes at the same $r$. There is a simple integer ratio. Find it numerically, then derive it by hand.

*In words: you have just discovered the first non-trivial term of the multipole expansion empirically. Predict what the next configuration in the sequence (a quadrupole: four charges, no net charge, no net dipole moment) will give for $n$, and then build one and check.*

---

## Stretch (optional)

**S1 — The Green function refactor.** Rewrite your potential function as
```
V(x) = sum over i of  q_i * G(x, x_i)
```
with `G` a separate function. You have not changed the physics, but you have changed the *structure*: `G` is now the free-space Dirichlet Green function of the Laplacian.

Now write a second Green function, `G_plane(x, x')`, implementing the grounded-conducting-plane case via the image construction (Jackson §2.1). Swap it in. Everything else in your code stays the same.

**Checkpoint.** With `G_plane` at $z=0$ and a single positive charge above it, confirm numerically that $V = 0$ everywhere on the plane, and that the field is perpendicular to the plane there.

*In words: your code now has a function whose signature is `G(x, x')` and which fully encodes the boundary conditions. Everything in Jackson Chapters 1–3 is the project of constructing this function for progressively harder geometries. Write two sentences on why that is a more powerful idea than solving each problem from scratch.*

**S2 — Grid-independence.** Replace the analytic field entirely: solve $\nabla^2 V = -4\pi\rho$ on the grid by Gauss–Seidel relaxation with $V=0$ on the boundary, then differentiate to get $\mathbf{E}$. Compare against the analytic answer. This is a genuinely different method — no superposition, no Green function, just the PDE — and getting the same answer two ways is the strongest validation available.

---

## Self-assessment

You have got what you came for if you can answer these without the code in front of you:

1. Why does the two-charge case require no new physics beyond the one-charge case?
2. What is the numerical signature of a correct centred-difference gradient, and how do you test for it?
3. Why are field lines perpendicular to equipotentials?
4. What does softening actually change, physically?
5. What is the object `G(x, x')` and why is constructing it the central task of Jackson Chapters 1–3?

---

## Notes

- Keep the code in one module with functions, not a script. You will extend it in Weeks 2, 7, and 9 of the Jackson plan, and again when you get to multipoles.
- Every checkpoint above should be a function returning a number. Resist the temptation to check things by looking at plots — the whole methodological point is that "it looks right" and "it is right" are different claims.
- Part (e)'s convergence test is the one to internalise. It generalises to every numerical method you will write for the rest of this project.
