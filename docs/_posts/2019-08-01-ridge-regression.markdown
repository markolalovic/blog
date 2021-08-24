---
layout: post
title:  "Ridge Regression"
subtitle: "Derivation of ridge regression estimator and it's properties, the role of the penalty function and the analysis of regularization parameter."
author: Marko Lalovic
date:   2019-08-01 10:31:22 +0100
image: assets/posts/ridge-regression/problem-resized.jpg
keywords: Penalized regression, regularization, correlated variables
rtime: 5 min read
---
<div class="images">
  <img src="/blog/assets/posts/ridge-regression/problem.svg">
  <div class="label">
    <strong>Figure 1:</strong> As the correlation between regressors increases the OLS method becomes unstable while the ridge regression method produces stable estimates regardless of the given data in $X$.
  </div>
</div>

The ordinary least squares (OLS) method is not suitable to estimate the unknown parameters $\beta$ in the case of highly correlated regressors. As the correlation between regressors in $X$ increases the OLS method becomes unstable. In the limit $\|corr(x_{i}, x_{j})\| \rightarrow 1$, the OLS objective function is no longer strictly convex and there are infinitely many solutions of OLS problem. The matrix $X$ becomes singular and both the variance of the estimator and the distance of the estimator to the actual $\beta$ go to infinity. See my previous post TODO: link.

Here, after introducing penalized regression, we derive the ridge regression estimator. Ridge regression is an effective approach to solve such problems. We show that, regardless of data $X$, unique solution to ridge regression always exists. By adding the ridge (vector of $\alpha$'s) on the diagonal of $X$, this method produces stable estimates of the coefficients in $\beta$. See **Figure 1** for illustration.

We illustrate the method on a simple example in R, explain the role of the penalty function and finish with analysis of regularization parameter $\alpha$.

## Penalized regression

In *penalized regression*, for $n > p$ and given $X: \mathbb{R}^{p} \rightarrow \mathbb{R}^{n}$ and $y \in \mathbb{R}^{n}$, we minimize the functional

$$
\begin{equation}
\newcommand{\norm}[1]{\left\lVert#1\right\rVert}
J_{\alpha}(\beta) = \norm{ y - X\beta }_{2}^{2} + \alpha P(\beta)
\end{equation}
$$

over $\beta \in \mathbb{R}^{p}$, where

* $J_{\alpha}: \mathbb{R}^{p} \rightarrow \mathbb{R}$ is the objective function;
* $P: \mathbb{R}^{p} \rightarrow \mathbb{R}$ is a *penalty function* that penalizes unrealistic values in $\beta$;
* Parameter $\alpha > 0$ controls the trade-off between the penalty and the fit of the loss function.

The main idea that determines the choice of the penalty function is that we would prefer a simple model to a more complex one.

There are many different possibilities for penalty function $P$. For example, if we want a smoother fit, then $P$ is a measure of the curvature.

In the case of correlated regressors, the estimated coefficients can become too large and $P$ is a measure of the distance of the coefficients from the origin. In this case, the main penalty function to consider is

$$
\begin{equation}
P(\beta) = \norm{\beta}_{2}^{2}
\end{equation}
$$

This type of penalized regression is called *Ridge regression*; see original paper [<a href="https://www.nrs.fs.fed.us/pubs/rn/rn_ne236.pdf" target="_blank">1</a>].


## Derivation of ridge regression estimator

Here, in order to simplify the derivation, we will assume that $X: \mathbb{R}^{p} \rightarrow \mathbb{R}^{n}$ is linear and continuous with full column rank $p$.

The objective function we want to minimize, written in a matrix form, is

$$
\begin{align}
\norm{ y - X \beta }_{2}^{2} + \alpha \norm{\beta}_{2}^2
&= (y - X \beta)^{T} (y - X \beta) + \alpha \beta^{T} \beta \nonumber \\[1em]
&= y^{T}y - 2 y^{T} X \beta + \beta^{T} X^{T}X \beta + \alpha \beta^{T} \beta
\end{align}
$$

By taking a partial derivative with respect to $\beta$ and setting it to zero
\begin{equation}
-2 X^{T} y + 2 X^{T} X \hat{\beta} + 2 \alpha \hat{\beta} = 0
\end{equation}

we get a regularized normal equation
\begin{equation}
(X^{T}X + \alpha I) \hat{\beta} = X^{T}y
\end{equation}

we can express $\hat{\beta}$ as
\begin{equation}
\hat{\beta} = (X^{T}X + \alpha I)^{-1} X^{T} y
\end{equation}

and since $\text{rank}(X) = p$
\begin{equation}
X z \neq 0 \quad \text{for each} \quad z \neq 0
\end{equation}

for the Hessian
\begin{equation}
2X^{T}X + 2 \alpha
\end{equation}

it holds that

$$
\begin{align}
2 z^{T} X^{T} X z + 2 \alpha z^{T} z
&= 2 (Xz)^{T} (Xz) + 2 \alpha z^{T} z \nonumber \\[1em]
&= 2 \norm{Xz}_{2}^{2} + 2 \alpha \norm{z}_{2}^{2} > 0
\quad \text{for each} \quad z \neq 0
\label{eq: positive-definite}
\end{align}
$$

therefore, the expressed $\hat{\beta}$ is really an estimator

\begin{equation}
\hat{\beta}_{\text{RR}} = (X^{T}X + \alpha I)^{-1} X^{T} y.
\label{eq: rr}
\end{equation}

### Example

We can illustrate the ridge regression method to estimate the unknown parameters $\beta$ in the case of correlated regressors on a simple example in R.

Suppose we have a model $y \sim \beta_{0} + \beta_{1} x_{1} + \beta_{2} x_{2}$, more specifically, let

\begin{equation}
\beta_{0} = 3, \quad \beta_{1} = \beta_{2} = 1.
\end{equation}

and let the sample contain 100 elements

```r
n <- 100
```

We then introduce some highly correlated regressors
```r
set.seed(42)
x1  <- rnorm(n)
x2  <- rnorm(n, mean = x1, sd = 0.01)
```

with correlation coefficient almost 1
```r
cor(x1, x2)
[1] 0.999962365268769
```

into the model
```r
y <- rnorm(n, mean = 3 + x1 + x2, sd = 1)
```

and calculate the estimate $\hat{\beta}_{\text{RR}}$ for $\alpha = 0.3$
```r
alpha <- 0.3
x <- as.matrix(cbind(int = 1, x1, x2))

beta.ridge <- function(alpha, x, y) {
  xx <- solve(t(x) %*% x + alpha * diag(3))
  return(as.vector(xx %*% t(x) %*% y))
}

beta.ridge(alpha, x, y)
[1] 2.98537494896842 0.815120466450887 1.04146900239714
```

### Properties of ridge regression estimator

* The unique solution \ref{eq: rr} of ridge regression $\hat{\beta}_{\text{RR}}$ always exists, since $X^{T}X + \alpha I$ is always rank $p$.

* We can derive a relationship between ridge and OLS estimators for the case when the matrix $X$ is orthogonal. Using $X^{T}X = I$ twice and since $\hat{\beta}_{\text{OLS}} = (X^{T}X)^{-1} X^{T} y$, we get the relation

$$
\begin{align}
\hat{\beta}_{\text{RR}} &= (X^{T}X + \alpha I)^{-1} X^{T} y \nonumber \\[1em]
&= (I + \alpha I)^{-1} X^{T} y \nonumber \\[1em]
&= (1 + \alpha)^{-1} I X^{T} y \nonumber \\[1em]
&= (1 + \alpha)^{-1} (X^{T}X)^{-1} X^{T} y \nonumber \\[1em]
&= (1 + \alpha)^{-1} \hat{\beta}_{\text{OLS}}
\end{align}
$$

* Ridge regression estimator $\hat{\beta}_{\text{RR}}$ is biased since, for each value of $\alpha > 0$, its expected value is not equal to $\beta$

$$
\begin{align}
\mathbb{E}[\hat{\beta}_{ridge}]
&= \mathbb{E}[(X^{T}X + \alpha I)^{-1} X^{T} y] \nonumber \\[1em]
&= \mathbb{E}[(X^{T}X + \alpha I)^{-1} (X^{T}X) (X^{T}X)^{-1} X^{T} y] \nonumber \\[1em]
&= \mathbb{E}[(X^{T}X + \alpha I)^{-1} (X^{T}X) \hat{\beta}_{\text{OLS}}] \nonumber \\[1em]
&= (X^{T}X + \alpha I)^{-1} (X^{T}X) \mathbb{E}[\hat{\beta}_{\text{OLS}}] \nonumber \\[1em]
&= (X^{T}X + \alpha I)^{-1} (X^{T}X) \beta.
\end{align}
$$

* Also, as $\alpha \rightarrow 0$, ridge estimator tends to OLS estimator, which can easiliy be seen from

$$
\begin{align}
\lim_{\alpha \to 0} \hat{\beta}_{\text{RR}}
&= \lim_{\alpha \to 0} (X^{T}X + \alpha I)^{-1} (X^{T}X) \hat{\beta}_{\text{OLS}} \nonumber \\[1em]
&= (X^{T}X)^{-1} (X^{T}X) \hat{\beta}_{\text{OLS}} \nonumber \\[1em]
&= \hat{\beta}_{\text{OLS}}
\end{align}
$$

## The role of the penalty function
The role of the penalty function can be shown conveniently with the help of singular value decomposition. Let

\begin{equation}
X = U \Sigma V^{T}
\end{equation}

be the singular value decomposition of $X$ where $\Sigma$ contains all the singular values

\begin{equation}
\sigma_{1} \geq \sigma_{2} \geq \dots \geq \sigma_{p} > 0
\end{equation}

The regularized normal equation
$$
\begin{align}
( X^{T} X + \alpha I ) \hat{\beta} = X^{T} y
\end{align}
$$

can be rewritten as

$$
\begin{align}
(V \Sigma^{T} U^{T}U \Sigma V^{T} + \alpha I) \hat{\beta}
= V \Sigma^{T} U^{T} y
\end{align}
$$

Then, since $U^{T}U = I$ and $V^{T}V = I$, we get

$$
\begin{align}
(V \Sigma^{T} \Sigma V^{T} + \alpha V^{T}V) \hat{\beta}
&= V \Sigma^{T} U^{T} y \nonumber \\[1em]
V (\Sigma^{T} \Sigma + \alpha I) V^{T} \hat{\beta}
&= V \Sigma^{T} U^{T} y
\end{align}
$$

Furthermore, multiplying it by $V^{T}$ from the left and setting $z := V^{T} \hat{\beta}$, we get
\begin{equation}
(\Sigma^{T} \Sigma + \alpha I) z = \Sigma^{T} U^{T} y
\end{equation}

Therefore

\begin{equation}
z_{i} = \frac{\sigma_{i} (u_{i}^{T} y)}{\sigma_{i}^{2} + \alpha} \quad \text{for} \quad i = 1, \dots, p
\end{equation}

And, for minimum norm solution, let
\begin{equation}
z_{i} = 0 \quad \text{for} \quad i = p + 1, \dots, n
\end{equation}


Finally, from $\hat{\beta} = V z$ and since $V$ is orthogonal

$$
\begin{equation}
\norm{\hat{\beta}} = \norm{VV^{T} \hat{\beta}}
= \norm{V^{T}\hat{\beta}}
= \norm{z}
\end{equation}
$$

we get

$$
\begin{equation}
\hat{\beta}_{i}
= \frac{\sigma_{i} (u_{i}^{T} y)}{\sigma_{i}^{2} + \alpha} v_{i}
\label{eq: beta_i}
\end{equation}
$$

And from

$$
\begin{equation}
\hat{\beta}_{i}
\approx
\begin{cases}
  0, & \text{if } \sigma_{i} << \alpha \\
  \frac{u_{i}^{T} y}{\sigma_{i}}v_{i}, & \text{if } \sigma_{i} >> \alpha
\end{cases}
\end{equation}
$$

we can see that the penalty function $\alpha \norm{\beta}_{2}^{2}$ acts as a filter since the contributions

* from $\sigma_{i}$ that is small relative to the regularization parameter $\alpha$ are almost eliminated;
* from $\sigma_{i}$ that is large relative to the regularization parameter $\alpha$ are left almost unchanged.

By defining a filter

$$
\begin{equation}
F_{\alpha}(\xi) = \frac{1}{\xi + \alpha}
\end{equation}
$$

the solution of ridge regression can be further expressed as

$$
\begin{equation}
\hat{\beta}_{\text{RR}} = F_{\alpha}(X^{T}X) X^{T}y
\end{equation}
$$


## Regularization parameter $\alpha$

First, we notice that the solution of ridge regression is monotonically decreasing in $\alpha$.

To show this, let

$$
\begin{equation}
\psi(\alpha) = \norm{\hat{\beta}_{\text{RR}}}_{2}^{2}
\end{equation}
$$

Then, from derived equation for $\hat{\beta}_{i}$ in \ref{eq: beta_i}, we have that

$$
\begin{equation}
\psi(\alpha) =
\sum_{i = 1}^{p} \frac{\sigma_{i}^{2} (u_{i}^{T} y)^{2}}{ (\sigma_{i}^{2} + \alpha)^{2} } v_{i}^{2}
\end{equation}
$$

and taking derivative on $\alpha$

$$
\begin{equation}
\psi'(\alpha) =
-2 \sum_{i = 1}^{p} \frac{\sigma_{i}^{2} (u_{i}^{T} y)^{2}}{ (\sigma_{i}^{2} + \alpha)^{3} } v_{i}^{2} < 0
\end{equation}
$$

Second, as $\alpha \rightarrow \infty$ the solution of ridge regression goes to $\boldsymbol{0}$, since

$$
\begin{equation}
\lim_{\alpha \rightarrow \infty} \psi(\alpha) =
\lim_{\alpha \rightarrow \infty} \sum_{i = 1}^{p} \frac{\sigma_{i}^{2} (u_{i}^{T} y)^{2}}{ (\sigma_{i}^{2} + \alpha)^{2} } v_{i}^{2} = 0
\end{equation}
$$

In the limit $\alpha \rightarrow 0$, as shown before, the solution of ridge regression goes to ordinary least squares solution. Furthermore, if $\sigma_{p} \rightarrow 0$ where $X$ is no longer full column rank, then $\psi(\alpha) \rightarrow \infty$.

We can plot how the estimates $\beta_{0}, \beta_{1}, \beta_{2}$ change depending on the value of parameter $\alpha$ for the data of the [Example](#example); shown in **Figure 2** below
```r
alphas <- exp(seq(-5, 10, 0.1))
betas <- sapply(alphas, function(alpha) {
    beta.ridge(alpha, x, y)
})

library(latex2exp) # for annotation
plot(log(alphas), betas[1, ],
     type="l", lty=1, lwd=3, col="red",
     xlab=TeX(r'($\log(\alpha)$)'),
     ylab=TeX(r'($\hat{\beta}$)'),
     cex.lab=1.5, cex.axis=1.5, cex.main=1.5, cex.sub=1.5)
lines(log(alphas), betas[2, ],
      type="l", lty=2, lwd=3, col="blue")
lines(log(alphas), betas[3, ],
      type="l", lty=3, lwd=3, col="black")
legend(7.73, 3.12,
       legend=c(
           TeX(r'($\hat{\beta}_{1}(\alpha)$)'),
           TeX(r'($\hat{\beta}_{2}(\alpha)$)'),
           TeX(r'($\hat{\beta}_{3}(\alpha)$)')),
       col=c("red", "blue", "black"),
       lwd=rep(3,3), lty=1:3, cex=1.5)
```

<div class="images">
  <img src="/blog/assets/posts/ridge-regression/ridge-solution-path.svg">
  <div class="label">
    <strong>Figure 2:</strong> The solution of ridge regression as a function of the regularization parameter $\alpha$.
  </div>
</div>

The selection of $\alpha$ is usually done by cross-validation. This means that we randomly partition the data into $K$ equally sized sets. For some value of $\alpha$ we then build a model (calculate estimates for the coefficients) on the data from $K - 1$ sets (learning set) and test it on the rest of the data (test set). Of course, we are interested in the mean square error (MSE). We then repeat the process for the remaining values of $\alpha$ and select the value of $\alpha$ where this average is the smallest. Typical values for $K$ are $5, 10$, and $n$ (sample size).

Let’s find the optimal value of parameter $\alpha$ for the data of the [Example](#example) using 10-fold cross-validation
```r
K <- 10
folds <- cut(seq(1, nrow(x)), breaks=K, labels=FALSE)
cv.matrix <- matrix(NA, nrow=K, ncol=length(alphas))

mse <- function(b, x, y) {
    return(1/length(y) * sum((y - x %*% b)^2))
}

for (k in 1:K) {
    test.i <- which(folds == k)
    for (j in 1:length(alphas)) {
        br <- beta.ridge(alphas[j], x[-test.i, ], y[-test.i])
        cv.matrix[k, j] <- mse(br, x[test.i, ], y[test.i])
    }
}

avgs <- apply(cv.matrix, 2, mean)
best.alpha <- alphas[avgs == min(avgs)]
best.alpha
[1] 0.246596963941606
```

## References
[1] Hilt, Donald E.; Seegrist, Donald W. 1977. Ridge: a computer program for calculating ridge regression estimates. Research Note NE-236. Upper Darby, PA: U.S. Department of Agriculture, Forest Service, Northeastern Forest Experiment Station. 7p.
<a target="_blank" href="https://www.nrs.fs.fed.us/pubs/rn/rn_ne236.pdf">https://www.nrs.fs.fed.us/pubs/rn/rn_ne236.pdf</a>
