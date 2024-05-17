---
layout: post
author: Marko Lalovic
title:  "On Storey's direct approach to false discovery rates"
subtitle: "Properties of Storey's estimator and duality with Benjamini-Hochberg procedure."
date: "2019-09-14"
nav_order: 3
keywords: multiple testing, multiple comparisons, false discovery rate
refs: direct-approach-to-fdr
---
Storey {% cite Storey --file {{ page.refs }} %} shed a new light on the multiple-testing problem by defining a positive false discovery rate measure and providing a new perspective with a direct approach to false discovery rates.

**Overview.** After an introduction to multiple-testing and Storey's direct approach the duality between Storey's direct approach and Benjamini-Hochberg (BH) procedure {% cite BH --file {{ page.refs }} %} is explored. It follows that the estimate of the proportion of null hypotheses plays a key role in the multiple-testing problem, and the approach we take doesn't matter much. This is confirmed by simulations where performances were practically identical for both methods.

When the distance between distributions under the null hypothesis and alternative hypotheses is small, the Storey's estimator can have a strong upward bias. By increasing the tuning parameter value, the upward bias in the simulations was reduced. However, this can result in underestimation, as already observed in {% cite Black --file {{ page.refs }} %}.

In case of dependence, Storey suggested the same approach {% cite Dependence --file {{ page.refs }} %}. From the simulation, it can be observed that dependence can lead to a very high overestimation of the false discovery rate.

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

# Introduction

## Single-hypothesis testing
Following the scientific method, researchers usually want to establish the truth of a statement by demonstrating that the opposite appears to be false. A *hypothesis* is a proposed explanation for a phenomenon that can be tested. In statistics, hypotheses involve a parameter $\theta$ whose value is unknown but must fall within a certain parameter space $\Omega$. Our assumption is that $\Omega$ can be partitioned into two disjoint subsets, $\Omega_0$ and $\Omega_1$, and define two hypotheses:

$$
\begin{align*}
H_{0}\text{: } \theta &\in \Omega_0, \\[.5em]
H_{1}\text{: } \theta &\in \Omega_1.
\end{align*}
$$

The hypothesis $H_{1}$ is the statement that confirms the theory and it is called an *alternative hypothesis*. The opposite $H_{0}$ is called a *null hypothesis*. The task is to decide which of the hypotheses appears to be true. A procedure for deciding this is called a *test*. Before deciding which hypothesis to choose, we assume that a random sample $\mathbf{X} = (X_{1}, ..., X_{n})$ is drawn from a distribution that involves the unknown parameter $\theta$. Let $T = r(\mathbf{X})$ be a statistic and let $\Gamma$ be a subset of the real line and suppose that the test procedure is of the form: reject $H_{0}$ if $T \in \Gamma$. Then $T$ is called a *test statistic* and $\Gamma$ a *rejection region*. A problem of this type is called *single-hypothesis testing*.

In single-hypothesis testing there are only two kinds of errors. A *type I error* occurs when rejecting the null $H_{0}$ when in fact $H_{0}$ is true. And *type II error* occurs when failing to reject the null when in fact the null hypothesis $H_{0}$ is false. The *significance level* $\alpha$ is the upper bound on the probability of type I error:

$$
\begin{equation} \label{eq: condition1}
	\text{Pr}(T \in \Gamma \mid H_{0} \text{ is true }) \leq \alpha.
\end{equation}
$$

The *power* $\pi$ of a test is the probability of rejecting $H_{0}$ when in fact $H_{0}$ is false. Practice is to choose the value of $\alpha$, say $0.01$, and find the rejection region $\Gamma_{\alpha}$ that maximizes the power and satisfies (\ref{eq: condition1}), that probability of type I error is at most $\alpha$. The detection power $\pi$ is a function of rejection region $\Gamma_{\alpha}$:

$$
	\pi(\Gamma_{\alpha}) = \text{Pr}(T \in \Gamma_{\alpha} \mid H_{0} \text{ is false }).
$$

Typically, rejection regions $\Gamma_{\alpha}$ and $\Gamma_{\alpha'}$ for different values $\alpha$ and $\alpha'$, are nested in the following sense:

$$
\begin{equation} \label{eq: nesting}
	\text{if} \quad \alpha < \alpha' \quad \text{then} \quad \Gamma_{\alpha} \subset \Gamma_{\alpha'}.
\end{equation}
$$

For the given observation $T = t$, the smallest significance level $\alpha$ at which a null hypothesis would be rejected, is called *$p$-value*:

$$
	p(t) = \inf \lbrace \alpha : t \in \Gamma_{\alpha} \rbrace.
$$

## Multiple-hypothesis testing
Now consider simultaneous testing a collection of $m$ hypotheses, called a *family of hypotheses* $\mathcal{F}$, where for $i = 1, \dots, m$, we test $H_{0i}$ versus $H_{1i}$ on the basis of test statistics given in the form of p-values $p_{i}$. Let $H_{1}, ...,H_{m}$ be random variables, where 

$$
	H_{i} = \begin{cases}
		0 & \text{if $H_{0i}$ is true,}\\
		1 & \text{if $H_{1i}$ is true.}
	\end{cases}
$$

A *multiple-testing procedure* is a decision function $\phi$ that given p-values $p_{1}, ..., p_{m}$ assigns decision values $D_{1}, ..., D_{m}$: 

$$
\phi(p_{i}) = D_{i}, \quad i = 1,..., m,
$$

where $D_{i} = 1$ if hypothesis $H_{0i}$ is rejected and $D_{i} = 0$ otherwise, for $i = 1, \dots, m$. <a href="#summary-table">Table 1</a> describes the possible outcomes from $m$ hypothesis tests, where $I_{0} \subset \lbrace 1, \dots, m \rbrace$ is the index of true null hypotheses, $m_{0} = \|I_{0}\|$ is the number of true null hypotheses, $V = \sum_{i=1}^{m} (1 - H_{i}) D_{i}$ is the number of true null hypotheses which are rejected (type I errors), $R = \sum_{i=1}^{m} D_{i}$ is the number of all rejected null hypotheses, etc. 

{% include posts/direct-approach-to-fdr/summary-table.html %}

If the decision function $\phi$ is based on rejection region of the form $[0, \gamma]$ for some threshold $\gamma > 0$:

$$
	\phi(p_{i}) = \mathbb{1}{\lbrace p_{i} \leq \gamma\rbrace},
$$

then the random variables $U, V, ..., R$ depend only on this threshold $\gamma$, for example $R(\gamma) = \sum_{i=1}^{m}{\mathbb{1}{\lbrace p_{i} \leq \gamma\rbrace}}$.

Good scientific practice requires the specification of certain type I error measure control to be done prior to the data analysis. The problem is to find a multiple-testing procedure which maximizes detection power and satisfies certain conditions involving type I error measures. A problem of this type is called a *multiple-testing problem*. It is described in {% cite RHeller --file {{ page.refs }} %} or in the context of assessing the feature significance in {% cite ESL --file {{ page.refs }} %}.

The multiple-testing procedure $\phi$ is said to have a *strong level $\alpha$ control* of some error measure EM, if it satisfies the condition:

$$
\text{EM}_{\phi} \leq \alpha
$$

for any configuration of true and false hypotheses in the family of $m$ hypotheses $\mathcal{F}$. The first type I error measure that was suggested is the *family-wise error rate (FWER)*. This is the probability that we make at least one type I error in the family $\mathcal{F}$ of $m$ hypotheses using a multiple-testing procedure $\phi$:

$$
	\text{FWER}_{\phi} = \text{Pr}(V \geq 1).
$$


<div class="mono-italic">
<p>I'm switching the theme of this blog and this post still needs some work. See the PDF version:</p>

<a href="https://github.com/markolalovic/direct-approach-to-fdr/blob/master/public/main.pdf">PDF version</a>
</div>

## Bonferroni correction

## Benjamini-Hochberg procedure

## Storey's approach

# References

{% bibliography --file {{ page.refs }} --cited %}