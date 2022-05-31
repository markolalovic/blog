---
layout: post
title:  "Lasso dual"
subtitle: "Derivation of the Lasso solution from dual formulation using projection operator and some non-obvious properties that follow from the geometry of the dual formulation."
author: Marko Lalovic
date:   2021-08-01 10:34:39 +0100
keywords: LASSO, duality
---
<div class="images">
  <img src="/blog/assets/posts/lasso-dual/transformation-3d.svg">
  <div class="label">
    <strong>Figure 1:</strong> Illustration when $n=p=3$ of primal and dual admissible sets $C$ and $D$. Solution to (1) can be determined by a projection of $y$. Furthermore, if $A^{-1}$ exists, then the primal solution can be expressed as a function of $y$.
  </div>
</div>

Let $A: \mathbb{R}^{p} \rightarrow \mathbb{R}^{n}$ and $y \in \mathbb{R}^{p}$. Consider the Tikhonov regularization

$$
\newcommand{\norm}[1]{\left\lVert#1\right\rVert}
\begin{equation}
\min_{x \in \mathbb{R}^{n}}
\frac{1}{2} \norm{Ax - y}_{2}^{2} + \alpha \norm{x}_{1}.
\label{eq:min-problem}
\end{equation}
$$

This type of penalized regression is called Lasso; see Tibshirani's original paper [<a href="http://statweb.stanford.edu/~tibs/lasso/lasso.pdf" target="_blank">1</a>].

In this post, we first derive the dual problem, then show that the solution $x^{\*}$ can be determined with the help of a projection operator. Under the assumption that $A^{-1}$ exists, we can further express the solution $x^{\*}$ with the help of a mapping $\left(A^{T}\right)^{-1}$  from the dual space as a function of $y$; see **Figure 1**.

Some nice and non-obvious properties of the solution $x^{\*}$ follow from the geometry of the dual formulation. For example that the Lasso solution is non-expansive as a function of $y$. This is not obvious and would probably be hard to show without the dual formulation.

<!-- Section 1 -->
## Formulation of the dual problem

To derive the dual problem, we can introduce a dummy variable $z \in \mathbb{R}^{n}$

$$
\begin{equation}\nonumber
z = Ax
\end{equation}
$$

and reformulate the minimization problem in \eqref{eq:min-problem} as a constrained problem

$$
\begin{align*}
\label{eq:primal-problem}
\tag{P}
&\underset{z \in \mathbb{R}^{n}, x \in \mathbb{R}^{p}}{\text{minimize}} \quad
\frac{1}{2} \norm{y - z}_{2}^{2} + \alpha \norm{x}_{1} \\
&\text{subject to} \quad z = Ax
\end{align*}
$$

Then we can construct the Lagrangian by introducing the dual variable $p \in \mathbb{R}^{n}$ (containing $n$ Lagrange multipliers)

$$
\begin{equation}\nonumber
L(x, z, p) = \frac{1}{2} \norm{y - z}_{2}^{2}
+ \alpha \norm{x}_{1} + p^{T} (z - Ax)
\end{equation}
$$

The dual objective function is

$$
\begin{equation}\nonumber
g(p) = \min_{z \in \mathbb{R}^{n}, x \in \mathbb{R}^{p}}
\left\lbrace
\frac{1}{2} \norm{y - z}_{2}^{2}
+ \alpha \norm{x}_{1} + p^{T} (z - Ax)
\right\rbrace
\end{equation}
$$

We can split the terms depending on $z$ and $x$ and minimize each part separately

$$
\begin{align}\nonumber
g(p)
&= \min_{z \in \mathbb{R}^{n}, x \in \mathbb{R}^{p}}
\left\lbrace
\frac{1}{2} \norm{y}_{2}^{2}
- y^{T}z
+ \frac{1}{2} \norm{z}_{2}^{2}
+ p^{T}z
+ \alpha \norm{x}_{1}
- p^{T}Ax
\right\rbrace \\[.3em]
&= \min_{z \in \mathbb{R}^{n}}\nonumber
\left\lbrace
\frac{1}{2} \norm{y}_{2}^{2}
- (y - p)^{T}z
+ \frac{1}{2} \norm{z}_{2}^{2}
\right\rbrace
+ \max_{x \in \mathbb{R}^{p}}
\left\lbrace
\alpha \norm{x}_{1}
- (A^{T}p)^{T}x
\right\rbrace
\end{align}
$$

We can use the stationarity condition, which says that at the optimal point, the subgradient of $L(x, z, p)$ with respect to $x$ and $z$ must contain 0.

For the first part, since $L(x, z, p)$ is differentiable in $z$, the subgradient with respect to $z$  equals the gradient. By taking
$\frac{\partial}{\partial z} L(x, z, p)$ and setting it to $0$, we get the stationarity condition

$$
\begin{equation}\nonumber
z = y - p^{*}
\end{equation}
$$

Plugging this into the first part, we get

$$
\begin{align}\nonumber
&\frac{1}{2}\norm{y}_{2}^{2} - (y - p^{*})^{T}(y - p^{*})
+ \frac{1}{2}\norm{y - p^{*}}_{2}^{2} \\[.3em]
&=
\frac{1}{2}\norm{y}_{2}^{2}\nonumber
- \frac{1}{2}\norm{y - p^{*}}_{2}^{2}
\end{align}
$$

For the second part, because $\alpha \norm{x}_1$ is a non-differentiable function of $x$, we need to compute the subdifferential $\partial (\alpha \norm{x}_1)$.

By using the rules for the subgradient of the maximum we can derive, see [<a href="https://see.stanford.edu/materials/lsocoee364b/01-subgradients_notes.pdf" target="_blank">2</a>], that the $\partial \norm{x}_{1}$ can be expressed as

$$
\begin{equation}\nonumber
\partial (\norm{x}_{1})
= \lbrace g : \norm{g}_{\infty} \leq 1, g^{T} x = \norm{x}_{1}
\rbrace
\end{equation}
$$

and using the rule for scalar multiplication

$$
\begin{equation}\nonumber
\partial (\alpha\norm{x}_{1})
= \lbrace g : \norm{g}_{\infty} \leq \alpha, g^{T} x = \alpha\norm{x}_{1}
\rbrace
\end{equation}
$$

Thus, we get the stationarity condition

$$
\begin{equation}\nonumber
g = A^{T}p \in \partial \alpha \norm{x}_{1}
\end{equation}
$$

when

$$
\begin{align} \nonumber
\norm{A^{T}p}_{\infty} \leq \alpha \\[.3em]
\alpha \norm{x}_{1} - (A^{T}p)^{T}x = 0\nonumber
\end{align}
$$


Therefore, the dual problem is

$$
\begin{align*}
\label{eq:dual-problem}
\tag{D}
&\max_{p \in \mathbb{R}^{n}}
\frac{1}{2}\norm{y}_{2}^{2}
- \frac{1}{2}\norm{y - p}_{2}^{2} \\[.3em]
&\text{subject to} \norm{A^{T}p}_{\infty} \leq \alpha
\end{align*}
$$


<!-- Section 2 -->
## Solution of the dual problem
The solution $p^{\*}$ of the dual problem can be determined with the help of a projection operator.

Looking at the dual problem formulation \eqref{eq:dual-problem}, we see that we can omit $\frac{1}{2}\norm{y}_{2}^{2}$, since this is constant. Then if we multiply it by 2 and flip the sign, we get an equivalent dual problem

$$
\begin{align*}
\label{eq:dual-problem-prime}
\tag{D'}
&\min_{p \in \mathbb{R}^{n}}
\norm{y - p}_{2}^{2} \\[.3em]
&\text{subject to} \norm{A^{T}p}_{\infty} \leq \alpha
\end{align*}
$$

A *projection operator* $P_{C}$ can be defined as

$$
\DeclareMathOperator*{\argmin}{arg\,min}
\begin{align}\nonumber
P_{C} \text{: } & \mathbb{R}^{n} \rightarrow C \\[.3em]
& y \mapsto P_{C}(y) := \argmin_{p \in C} \norm{y - p}_{2} \nonumber
\end{align}
$$

where $C \subset \mathbb{R}^{n}$ is closed and convex set. Looking at \eqref{eq:dual-problem-prime}, we see that $p^{\*}$ is a projection

$$
\begin{equation}\nonumber
p^{*} = P_{C}(y)
\end{equation}
$$

where $C$ is equal to

$$
\begin{equation}\nonumber
C = \lbrace p \in \mathbb{R}^{n} :
\norm{A^{T}p}_{\infty} \leq \alpha
\rbrace
\end{equation}
$$

We notice that $C$ is indeed closed and convex. For example if $n=p=2$, and let

$$
\begin{equation}\nonumber
A^{T} = \begin{pmatrix}
a_{11} & a_{21}\\
a_{12} & a_{22}
\end{pmatrix}
\end{equation}
$$

then

$$
\begin{equation}\nonumber
C = \lbrace -\alpha \leq a_{11} p_{1} + a_{12} p_{2} \leq \alpha \rbrace
\cap
\lbrace -\alpha \leq a_{21} p_{1} + a_{22} p_{2} \leq \alpha \rbrace
\end{equation}
$$

Or we can express $C$ as

$$
\begin{equation}\nonumber
C = (A^{T})^{-1}(D)
\end{equation}
$$

where $D$ is equal to

$$
\begin{equation}\nonumber
D = \lbrace d \in \mathbb{R}^{p} : \norm{d}_{\infty} \leq \alpha \rbrace
\end{equation}
$$

Again for $p=2$

$$
\begin{equation}\nonumber
D = \lbrace -\alpha \leq d_{1} \leq \alpha \rbrace
\cap
\lbrace -\alpha \leq d_{2} \leq \alpha \rbrace
\end{equation}
$$

<div class="images">
  <img src="/blog/assets/posts/lasso-dual/transformation-2d.svg">
  <div class="label">
    <strong>Figure 2:</strong> Illustration when $n=p=2$ of primal and dual admissible sets $C$ and $D$.
  </div>
</div>

<!-- Section 3 -->
## Solution of the primal problem
From optimality condition of the dual problem, we can derive the primal solution under the assumption that $A^{-1}$ exists.

During the formulation of the dual problem, we introduced the dummy variable $z = Ax$ and derived the stationary condition $z = y - p^{\*}$.  From this, we get that every solution $x^{\*}$ of \eqref{eq:primal-problem} should satisfy

$$
\begin{equation}\nonumber
A x^{*} = y - p^{*}
\end{equation}
$$

where $p^{\*}$ is a solution of the dual problem \eqref{eq:dual-problem}. Therefore, if $A^{-1}$ exists, the primal solution is

$$
\begin{equation}\nonumber
x^{*} = A^{-1} \left( y - P_{C}(y) \right)
\end{equation}
$$

We notice that, since $P_{C}$ is a projection onto convex set $C$, it follows that $x^{\*}$ is also non-expansive as a function of $y$; see **Figure 3(a)** in contrast to **Figure 3(b)** showing projection onto non-convex set $N$. This is not obvious and would probably be hard to show without the dual formulation.

| ![convex.svg][convex] | ![bean.svg][bean] |
|:--:|:--:|
| <span class="subfig">(a) Convex set $C$.</span> | <span class="subfig">(b) Non-convex set $N$.</span> |

<div class="images">
  <div class="label">
    <strong>Figure 3:</strong> Lasso solution $x^{*}$ is non-expansive as a function of $y$.
  </div>
</div>


## References
[1] Tibshirani, R. (1996). Regression shrinkage and selection via the lasso. J. Royal. Statist. Soc B., Vol. 58, No. 1, pages 267-288).
<a target="_blank" href="https://webdoc.agsci.colostate.edu/koontz/arec-econ535/papers/Tibshirani%20(JRSS-B%201996).pdf</a>

[2] S. Boyd and L. Vandenberghe, “Subgradients Notes”, (2008), Stanford University
<a target="_blank" href="https://see.stanford.edu/materials/lsocoee364b/01-subgradients_notes.pdf">https://see.stanford.edu/materials/lsocoee364b/01-subgradients_notes.pdf</a>

[convex]: /blog/assets/posts/lasso-dual/convex.svg
[bean]: /blog/assets/posts/lasso-dual/bean.svg
