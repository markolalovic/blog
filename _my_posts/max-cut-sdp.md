---
layout: post
author: Marko Lalovic
title:  "Maximum cut and Goemans-Williamson"
subtitle: "The famous semidefinite programming relaxation and the duality with the Sum-of-Squares hierarchy framework."
date: "2022-08-31"
nav_order: 9
keywords: maximum cut, semidefinite programming, approximation algorithms
refs: max-cut-sdp
---
This is a short summary of approximation algorithms for the MaxCut problem of finding a maximum cut of a graph. After introducing the problem and trivial $\frac{1}{2}$-approximation, we summarize the famous semidefinite programming relaxation and hyperplane rounding technique from {% cite GW94 --file {{ page.refs}} %} that gives the best known approximation ratio for MaxCut. We then take a look at the dual problem and some previous results. Taking the dual approach further, using so-called Sum-of-Squares hierarchy framework with Gaussian rounding technique, we arrive at the same approximation ratio for MaxCut. Finally, we discuss some possible generalizations.

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

## Introduction
Let $G = (V, E)$ be a simple undirected graph with $V := \lbrace 1, \dots, n \rbrace$ and $|E| = m$. For a subset $S \subset V$, we define a $cut(S)$ as a subset of edges $E$ having one vertex in $S$ and the other one in $V \setminus S$. We denote the cut size by $f(S) = |cut(S)|$. The *MaxCut problem* is to find a subset $S \subseteq V$ that maximizes the cut size $f(S)$. MaxCut problem is NP-complete {% cite Karp72 --file {{ page.refs }} %}.

<a href="#figure1">Figure 1</a> shows a cycle graph $C_{5}$ where a subset $S = \lbrace{v_1, v_3\rbrace}$ gives a cut of maximum size 4.

<div id="figure1" class="imgcap-nbd">
  <img src="/blog/assets/posts/max-cut-sdp/mc-example.svg" width="30%" class="invertImg">
  <div class="thecap">
    Figure 1: Cycle graph $C_5$ and a maximum cut of size 4.
  </div>
</div>

Given a graph $G=(V,E)$, denote the maximum cut size by $mc(G)$. For a given algorithm that returns the subset $S \subseteq V$, we say that it is an $\alpha$-*approximation algorithm* for MaxCut if

$$
\begin{equation}
f(S) \geq \alpha \cdot mc(G)
\label{eq: alpha}
\end{equation}
$$

for all graphs $G = (V, E)$ and some approximation ratio $\alpha \in [0, 1]$. If algorithm employed is randomized, meaning $S$ that it returns is a random variable, then we say the same, if (\ref{eq: alpha}) holds with an expectation taken on the left-hand side. 

#### Example
An algorithm, that assigns each vertex to $S$ or $V \setminus S$ independently uniformly at random, is a $1/2-$approximation for MaxCut:

$$
\DeclareMathOperator{\Ex}{\mathbb{E}}
\DeclareMathOperator{\Prob}{\mathbb{P}}
\mathbb{E}[f(S)] = \sum_{i, j} \Prob[(i, j) \in \text{cut}(S)] = \frac{1}{2} \, m \geq \frac{1}{2} \cdot mc(G).
$$ 

This approximation was first proposed and analyzed by {% cite Erd67 --file {{ page.refs }} %}. 

In {% cite GW94 --file {{ page.refs }} %} they improved this by proposing $\alpha_{GW}$-approximation algorithm with $\alpha_{GW} \geq 0.878$ using semidefinite programming (SDP) and hyperplane rounding technique. The proposed algorithm is easy to implement using off-the-shelf SDP solver, see the [Source code](#source-code) for more details.

## Preliminaries
A real symmetric matrix $X$ is positive definite, denoted $X \succeq 0$ if the following equivalent conditions hold:

* All eigenvalues of $X$ are non-negative.
* Quadratic form $v^{T}Xv \geq 0$ for all $v \in \mathbb{R}^{n}$.
* There exists $Q \in \mathbb{R}^{n \times r}$ with 

$$ X = QQ^{T} = \sum_{i = 1}^{r} v_{i} v_{i}^{T} $$

where $v_{1}, \dots, v_{r}$ are columns of $Q$.

In the set of all symmetric matrices $S^{n}$, denote the convex cone of positive semidefinite matrices by

$$
S^{n}_{+}.
$$ 

A *spectrahedron* is the intersection of $S^{n}_{+}$ with an affine linear space $A \bullet X  = b$. *Semidefinite programming (SDP)* solves the problem (P) of maximizing (or minimizing) a linear objective function $C \bullet X$ over the spectrahedron:

$$
\begin{align*}
& \text{maximize} \ C \bullet X \\
& \text{subject to: } \tag{P}\\
& \qquad A_{i} \bullet X  = b_{i} \quad i = 1, \dots, m \\
& \qquad X \succeq 0 
\end{align*}
$$

The corresponding dual problem (D) is:

$$
\begin{align*}
& \text{minimize} \ b^{T}y \\
& \text{subject to: } \label{eq: P} \tag{D}\\
& \qquad \sum_{i=1}^{m} A_{i} y_{i} - C \succeq 0
\end{align*}
$$

Properties:

* Every convex polyhedron is a spectrahedron.

* Spectrahedron is a closed convex set.

* *Weak duality* always holds:

$$ C \bullet X \geq b^{T} y.$$

* Under primal and dual feasibility, also *strong duality* holds:

$$ C \bullet X = b^{T} y. $$

More on semidefinite programming can be found in Chapter 12 {% cite Sturmfels --file {{ page.refs}} %}.

## SDP relaxation
Cuts in the complete graph $K_n$ can be represented by a set of $2^{n - 1}$ matrices $x x^{T}$ of rank one with $x \in \lbrace -1, 1 \rbrace^{n}$. The $n \times n$ spectrahedron:

$$
\mathcal{E}_{n} := \lbrace X \in S^{n}_{+} : X_{ii} = 1, \, \, \forall i \rbrace
$$ 

is called an *elliptope*. It is a set of all $n \times n$ correlation matrices. 

It gives the following formulation of maximum cut problem:

$$
mc(G) = \max_{X \in \mathcal{E}_{n}, rk(X) = 1 } \sum_{i, j} \frac{1 - X_{ij}}{2} .
$$

Letting go of rank one constraint $rk(X) = 1$, we get an SDP problem:

$$
sdp(G) := \max_{X \in \mathcal{E}_{n}} \sum_{i, j} \frac{1 - X_{ij}}{2}.
$$

Since $\mathcal{E}_{n}$ includes all the matrices of rank one,

$$
mc(G) \leq sdp(G).
$$

Therefore $sdp(G)$ is a *relaxation*.


#### Example
Cuts in complete graph $K_{3}$ can be represented by a set of matrices of rank one:

$$
\lbrace x x^{T} : x \in \lbrace -1, 1 \rbrace^{3} \rbrace.
$$

These matrices are contained in $3 \times 3$ elliptope 

$$
\mathcal{E}_{3} = \lbrace 
(x_1, x_2, x_3)^{T} \in \mathbb{R}^{3} :
\begin{pmatrix}
1 & x_1 & x_2 \\
x_1 & 1 & x_3 \\
x_2 & x_3 & 1
\end{pmatrix} \succeq 0 
\rbrace.
$$

The boundary of $\mathcal{E}_{3}$ consists of all $(x_1, x_2, x_3)^{T} \in \mathbb{R}^{3}$ where determinant is 0:

$$
(x_2 x_3 - x_1) x_1 + (x_1  x_3 - x_2) x_2 - x_{3}^2 + 1 = 0.
$$

This is a cubic surface drawn in <a href="#figure2">Figure 2</a> using Python and SageMath:

```python
from sage.all import *
var('x,y,z')
f(x, y, z) = (y*z - x)*x + (x*z - y)*y - z^2 + 1
p = implicit_plot3d(f(x,y,z) == 0, (x,-1,1), (y,-1,1), (z,-1,1),
                    opacity=0.9, plot_points=50, smooth=True)
p
```

<div id="figure2" class="imgcap-nbd">
 <img src="/blog/assets/posts/max-cut-sdp/K3-and-elliptope.svg" width="80%" class="invertImg">
  <div class="thecap">
    Figure 2: Complete graph $K_3$ and surface of elliptope $\mathcal{E}_{3}$.
  </div>
</div>

## Hyperplane rounding
Solution $X$ that realizes the maximum $sdp(G)$ is a positive semidefinite matrix that can be decomposed as $X=\sum_{i=1} v_{i}v_{i}^T$ for some unit vectors $v_{i}$. To construct a subset $S \subset V$, select a random unit vector $r \in \mathbb{R}^{n}$ and let $S = \lbrace i \in V : v_{i}^{T}r \geq 0 \rbrace$. Denote the angle between vectors $v_i, v_j$ by $\theta_{i, j}$. 

Since $v_{i}$ are unit vectors $v_{i}^{T} v_{j} = \cos{\theta_{i, j}}$, we can bound the contribution of an edge $(i, j) \in E(G)$ by

$$
\Prob[(i, j) \in cut(S)] = 
\frac{\theta_{i, j}}{\pi}  \geq \alpha_{GW} \cdot
\frac{1 - v_{i}^{T} v_{j} }{2} 
$$

where 

$$
\alpha_{GW} = \min_{\theta_{i, j} \in [0, \pi]} \lbrace 
\frac{2}{\pi} \cdot \frac{\theta_{i, j}}{ { 1 - \cos(\theta_{i, j}) }} \rbrace \geq 0.878.
$$ 

Summing up the contributions over all edges, we arrive at the famous result

$$
\Ex[f(S)] = \sum_{i, j} \Prob[(i, j) \in cut(S)] \geq \alpha_{GW} \cdot sdp(G).
$$

## Dual problem
Let $L$ be the Laplacian matrix of a graph $G = (V, E)$. The dual problem of SDP relaxation is equivalent to finding maximum eigenvalue of $L$ with smallest added correction $u \in \mathbb{R}^{n}$, under constraint that $\sum_{i} u_{i} = 0$

$$
\newcommand{\vect}[1]{\boldsymbol{#1}}
\frac{n}{4} \min_{u: \vect{1}^{T} u = 0} \lambda_{\max}(L + diag(u)) 
$$

Setting $u = \vect{0}$ and by weak duality we get an upper bound

$$ mc(G) \leq sdp(G) \leq \frac{n}{4} \lambda_{\max}(L) $$

given earlier in {% cite MP90 --file {{ page.refs }} %}. For 5-cycle $C_{5}$, $\lambda_{\max} = \frac{1}{5}(5 + \sqrt{5})$, giving an upper bound

$$
\frac{1}{2}(5 + \sqrt{5})/4 \geq 0.9 \cdot mc(C_{5}).
$$

This bound was studied in {% cite DP93 --file {{ page.refs }} %}.

## SoS relaxation
Now let $x \in \lbrace 0, 1 \rbrace^{n}$ and let $\mathcal{P}_{n}$ be a set of real valued polynomials $p(x)$ of degree at most $d/2$. Let 

$$
\tilde{\Ex}: \mathcal{P}_{n} \rightarrow \mathbb{R}
$$ 

be some operator over the probability distribution on $x$. We formulate the following optimization problem 

$$
\begin{align*}
sos(G) := \ & \max_{\tilde{\Ex}} \ \tilde{\Ex}[ \sum_{i, j} (x_{i} - x_{j})^{2} ] \\
& \text{subject to: } \\[.4em]
& \qquad \text{(1) $\tilde{\Ex}$ is linear} \qquad \text{(2) $\tilde{\Ex}[1] = 1$} \\[.4em]
& \qquad \text{(3) $\tilde{\Ex}[p^2] \geq 0$} \qquad \, \, \, \text{(4) $\tilde{\Ex}[x_{i}^{2}p] = \tilde{\Ex}[x_{i}p]$} \\[.4em]
& \qquad \forall p \in \mathcal{P}_{n}
\end{align*}
$$

An operator $\tilde{\Ex}$ that realizes the maximum $sos(G)$ is called *pseudo-expectation*, because it only meets a subset of constraints (1)-(3) required to be an actual expectation. The fourth constraint requires $x$ to be a vector $x \in \lbrace 0, 1 \rbrace^{n}$. Given a subset $S$ with maximum cut size $mc(G)$, let $\vect{a}$ be the indicator vector of $S$ and set $\tilde{\Ex} = \vect{a}$. Then $\tilde{\Ex}$ satisfies the constraints (1)-(4) and achieves objective value $mc(G)$. In other words, given an expectation $a$, we can build a pseudo-expectation $\tilde{\Ex}$ just by setting $\tilde{\Ex} = \vect{a}$. Therefore $mc(G) \leq sos(G)$.

This framework is called *Sum-of-Squares hierarchy* introduced by {% cite Parillo00 --file {{ page.refs }} %} and {% cite Lasserre01 --file {{ page.refs }} %}. Increasing the degree $d$, increases the size of the corresponding SDP problem. For $d = n$, the relaxation is exact. We can choose $d=2$ and show that we can get the same $\alpha_{GW}$-approximation as before by rounding the solution of SoS relaxation.

## Gaussian rounding 
Let $\tilde{\Ex}$ be solution that realizes the maximum $sos(G)$. Select $\vect{y}$ to be a Gaussian vector with the mean $\mu = \tilde{\Ex}[x] = \frac{1}{2} \vect{1}$ and covariance matrix $\Sigma = \tilde{\Ex}[(x - \mu)(x - \mu)^{T}]$ and construct a subset $S = \lbrace i \in V : y_{i} \leq \frac{1}{2} \rbrace$. For each edge $(i, j) \in E(G)$, define 

$$
\rho_{i, j} = 4 \tilde{\Ex}[x_{i} x_{j}] - 1.
$$ 

From $(s, t) \overset{\text{i.i.d.}}{\sim} N(0, I)$, setting 

$$
u = \rho_{i, j} s + \sqrt{1 - \rho_{i, j}^{2}} t
$$ 

gives $\rho_{i, j} = \Ex[u s] = \cos(\theta_{i, j})$. Summing up the contributions, we can show that

$$
\Ex[f(S)] = \sum_{i, j}  \Prob[(i, j) \in cut(S)] \geq \alpha_{GW} \cdot 
sos(G)
$$

## Generalizations
Given a non-negative weight function on the edges, $w \in \mathbb{R}^{E}_{+}$, the cut size becomes

$$
f(S) = \sum_{e \in \text{cut}(S)} w_e
$$

with no effect on analysis of approximation algorithms presented here. Given arbitrary weight function $w \in \mathbb{R}^{E}$, the analysis has to be adjusted but it is possible to derive a generalization of guarantees presented here. 

Whether increasing degree from $d=2$ to $d=4$ of SoS Relaxation, also improves the approximation, still remains unresolved question.

## Source code
Here is a simple implementation of Goemans-Williamson algorithm in Python:

```python
import numpy as np
import cvxpy as cp
from scipy.linalg import sqrtm

def gw(n, edges):
    '''Goemans-Williamson algorithm for MaxCut:
    Given a graph G(V=[n], E=edges), returns a vector x \in {-1, 1}^n
    that corresponds to the chosen subset of vertices S of V. '''
    ## SDP Relaxation
    X = cp.Variable((n, n), symmetric=True)
    constraints = [X >> 0]
    constraints += [
        X[i, i] == 1 for i in range(n)
    ]
    objective = sum( 0.5*(1 - X[i, j]) for (i, j) in edges )
    prob = cp.Problem(cp.Maximize(objective), constraints)
    prob.solve()

    ## Hyperplane Rounding
    Q = sqrtm(X.value).real
    r = np.random.randn(n)
    x = np.sign(Q @ r)

    return x

def cut(x, edges):
    '''Given a vector x \in {-1, 1}^n and edges of a graph G(V=[n], E=edges),
    returns the edges in cut(S) for the subset of vertices S of V represented by x.'''
    xcut = []
    for i, j in edges:
        if np.sign(x[i]*x[j]) < 0:
            xcut.append((i, j))
    return xcut
```
To solve an SDP problem it uses CVXPY {% cite CVXPY --file {{ page.refs }} %}.

Algorithm is randomized and returns a random cut, but guarantees that on average: `cut size >= 0.878 * maximum cut size`. For the trivial example of a cycle on 5 vertices, the chosen subset and corresponding cut may vary, but the return cut will always be of size 4:
```python
## Define a cycle on 5 vertices
n = 5
edges = [(0, 1), (1, 2), (2, 3), (3, 4), (0, 4)]

## Run Goemans-Williamson algorithm
x = gw(n, edges)

## Find edges in the cut
xcut = cut(x, edges)

print('Chosen subset: %s' % np.where(x == 1))
print('Cut size: %i' % len(xcut) )
print('Edges of the cut: %s' % xcut )
# Chosen subset: [1 4]
# Cut size: 4
# Edges of the cut: [(0, 1), (1, 2), (3, 4), (0, 4)]
```
## References

{% bibliography --file {{ page.refs }} --cited %}