---
layout: post
author: Marko Lalovic
title:  "Layout optimization for a solar tower power plant"
subtitle: "Finding the optimal arrangement of heliostats to maximize the energy production of a solar tower power plant."
date: "2022-07-12"
nav_order: 8
keywords: layout optimization, concentrated solar, heliostat field
refs: layout-optimization
---
A solar power tower plant uses flat, rotating mirrors called ***heliostats*** to reflect and concentrate solar radiation onto a tower-mounted receiver. <a href="#figure1">Figure 1</a> shows heliostats reflecting sunlight during solar noon, when the sun is at its zenith in the sky.

<div id="figure1" class="imgcap">
  <img src="/blog/assets/posts/layout-optimization/solar-power-tower.svg" alt="Schematic of a solar power tower plant" width="80%">
  <div class="thecap">
    Figure 1: Schematic of a solar power tower plant.
  </div>
</div>
<!-- A heliostat, positioned on a pylon, redirects sunlight onto a tower-mounted receiver.  -->
<!-- a day the position of the sun varies and each -->
Throughout the day, each heliostat rotates to ensure that the sunlight remains focused on the receiver by using a tracking system as shown in <a href="#figure2">Figure 2</a>.

<div class="mono-italic">
Click to animate the diagram:
</div>
<button class="btn btn-blue" title="Run" id="runButton" onclick="run()">RUN</button>
{% include posts/layout-optimization/heliostat-rotation.html %} 

Note that the position of the sun varies, and individual heliostats can be blocked or shaded by neighboring heliostats, which affects the efficiency of the power plant. In addition, there are other factors that affect the power plant's efficiency as described in <a href="#table1">Table 1</a>.

{% include posts/layout-optimization/effects-table.html %}

Given the specifications of the power plant, including the tower height and receiver dimensions, the task is to determine the optimal locations of heliostats that maximize the plant's efficiency. This optimal arrangement of heliostats is referred to as ***optimal layout***.

Following a brief introduction, a simplified model is presented for determining the optimal layout. The task of finding an optimal layout for a single heliostat is solved using only geometry. In the case of multiple heliostats, discretization and ray tracing are used to estimate the efficiency of a solar power plant taking into account shading and blocking effects. Then, the layout is optimized using Sequential Quadratic Programming (SQP).

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

## Introduction
Concentrating solar power systems such as solar tower power plants concentrate sunlight to produce electricity, offering a renewable energy production. However, the cost of electricity from these systems remain high compared to other renewable sources, as indicated in <a href="#figure3">Figure 3</a>.

<div id="figure3" class="imgcap-nbd">
  <img src="/blog/assets/posts/layout-optimization/energy-costs.svg" alt="Global weighted average cost of electricity" width="80%" class="invertImg">
  <div class="thecap">
    Figure 3: Global weighted average cost of electricity. Data from: <a href="https://www.irena.org/Data/View-data-by-topic/Costs/Global-Trends">IRENA (2023)</a>.
  </div>
</div>

Efforts in research and development are focused on improving efficiency, cutting construction and operation expenses, and ultimately reducing electricity production costs. <a href="#figure4">Figure 4</a> shows an experimental solar power tower plant at the German Aerospace Center.

<div id="figure4" class="imgcap-nbd">
  <img src="/blog/assets/posts/layout-optimization/DLR.jpg" alt="Experimental solar power plant" width="80%">
  <div class="thecap">
    Figure 4: Experimental solar power plant near Jülich in Germany. Source: <a href="https://www.dlr.de/sf/en/desktopdefault.aspx/tabid-8560/15527_read-44867/">DLR</a> (CC-BY 3.0).
  </div>
</div>

## Optical model for a solar tower power plant
We begin by examining a solar power tower plant employing a single heliostat and determining its optimal position. Although actual plants employ thousands of heliostats, focusing on a single one makes it easier to explain the model. Furthermore, the derived solution for a single heliostat allows us to verify the accuracy of the optimization methods to find the optimal layout for multiple heliostats. The model introduced here operates in two dimensions to simplify complexity, but a more realistic approach would involve replacing all definitions with their three-dimensional counterparts.

The normal $n(t)$ of heliostat at time $t$ is given by

$$
\newcommand{\norm}[1]{\left\lVert#1\right\rVert}
n(t) = \frac{r + s(t)}{\norm{r + s(t)}}.
$$

Here, $r$ denotes the direction from the center $H$ of the heliostat to the center of the receiver $R$, while $s(t)$ is the normalized solar vector at time $t$, as illustrated in <a href="#figure5">Figure 5</a>.

<div id="figure5" class="imgcap-nbd">
  <img src="/blog/assets/posts/layout-optimization/drawings/heliostat-normal.svg" width="80%" class="invertImg">
  <div class="thecap">
    Figure 5: Vectors $r, s(t)$, and normal $n(t)$ of a heliostat.
  </div>
</div>

## Cosine effect
The primary energy loss in a solar power plant occurs when sunlight does not strike the heliostat perpendicularly. The effective reflection area of the heliostat is reduced by the cosine of one-half of the angle between vectors $r$ and $s$. This phenomenon is known as the ***cosine effect***: 

$$
\eta_{cos} = \cos{\left( \frac{\gamma}{2} \right)}
$$

where

$$
\gamma = \angle (r, s).
$$

As depicted in <a href="#figure6">Figure 6A</a>, an $\eta_{cos}$ value approximately 0 indicates minimal power output, while an $\eta_{cos}$ value close to 1 signifies maximum power output (<a href="#figure6">Figure 6B</a>).

<div id="figure6" class="imgcap-nbd">
  <img src="/blog/assets/posts/layout-optimization/drawings/cosine-effect.svg" width="80%" class="invertImg">
  <div class="thecap">
    Figure 6: Cosine effect on the power output. No power (<strong>A</strong>) when $\eta_{cos} \approx 0$. Maximum power (<strong>B</strong>) when $\eta_{cos} \approx 1.$
  </div>
</div>

In order to determine the optimal position of the heliostat to maximize the power output, we will express the cosine effect $\eta_{cos}$ in terms of the heliostat's center $H$. Let $\rho$ denote the angle between a line passing through the receiver's center $R$ and the heliostat's center $H$, and the horizontal line passing through $R$. Additionally, let $\phi$ denote the angle between the direction of sunlight and the ground's horizontal plane, as shown in <a href="#figure7">Figure 7</a>.

<div id="figure7" class="imgcap-nbd">
  <img src="/blog/assets/posts/layout-optimization/drawings/angles.svg" width="80%" class="invertImg">
  <div class="thecap">
    Figure 7: Angles $\gamma$, $\rho$ and $\phi.$
  </div>
</div>

In this simplified model, as the sun's position changes throughout the day, the solar angle $\phi$ ranges from 0 to $\pi$. The angle $\gamma = \angle (r, s)$ can be expressed in terms of $\phi$ and $\rho$ as:

$$
\phi = |\pi - \phi - \rho|
$$

and ***cumulative daily cosine effect*** can be expressed in terms of $\rho$ as:

$$
\begin{equation}
\eta_{cos}(\rho) = \int_{0}^{\pi} \cos \left( \frac{|\pi - \phi - \rho |}{2} \right) d\phi
= 2 \left( \sin \left( \frac{\rho}{2} \right)
+ \cos \left(\frac{\rho}{2}\right)  \right).
\label{eq: cosine-effect}
\end{equation}
$$

## Spillage effect
Solely focusing on the cumulative daily cosine effect, the optimal position of the heliostat to maximize power output is directly underneath the receiver. This corresponds to $\rho = \pi/2$, as indicated by equation \ref{eq: cosine-effect}. 

However, in this position, many reflected rays from the heliostat fail to reach the receiver at various times throughout the day as illustrated in <a href="#figure8">Figure 8A</a>.

<div id="figure8" class="imgcap-nbd">
  <img src="/blog/assets/posts/layout-optimization/drawings/spillage-effect.svg" width="80%" class="invertImg">
  <div class="thecap">
    Figure 8: Spillage effect on the power output. No power (<strong>A</strong>) when $\eta_{spill} \approx 0$. Maximum power (<strong>B</strong>) when $\eta_{spill} = 1.$
  </div>
</div>

This effect on the energy loss is known as ***spillage effect*** and can be defined as follows:

$$
\eta_{spill} = \text{the proportion of rays that reach the receiver.}
$$

To derive the spillage effect $\eta_{spill}$, let $\tau$ be the angle between the line passing through the receiver's surface and the ground's horizontal plane, as shown in <a href="#figure9">Figure 9</a>.

<div id="figure9" class="imgcap-nbd">
  <img src="/blog/assets/posts/layout-optimization/drawings/spillage-derivation.svg" width="80%" class="invertImg">
  <div class="thecap">
    Figure 9: Notation in the derivation of the spillage effect.
  </div>
</div>

Note that,

$$
\begin{align*}
\alpha &= \pi - \tau - \rho \\[.5em]
\beta &= (\phi + \rho) / 2.
\end{align*}
$$

By similarity and sine rule,

$$
\begin{align*}
\frac{|AQ|}{|AR|} &= \frac{|AP|}{|AH|} \\[.5em]
\frac{|AR|}{\sin{\alpha}} &= \frac{|AH|}{\sin{\beta}}
\end{align*}
$$

we can derive the spillage effect in terms of angles $\rho, \tau$ and $\phi$ as

$$
\begin{equation}
\eta_{spill} = \min \left( 1, \frac{|HP|}{|RQ|} \right)
= \min \left( 1, \frac{\sin{(\pi - \tau - \rho)} }{ \sin{\left((\phi + \rho)/2\right)} } \right).
\label{eq: spillage-effect}
\end{equation}
$$

Note that, in our model, the angle $\tau \approx 80^{\circ}$ and it is a fixed constant because it is part of plant's specification. Therefore $\eta_{spill}$ only varies with respect to $\phi$ and $\rho$.

## Atmospheric attenuation
Another loss of energy is due to atmospheric attenuation. This loss depends on the distance $d$ between a heliostat and the receiver and assumes a visibility of 40 km. See {% cite Richter2017 --file {{ page.refs }} %} for more details.

Denote by $d$ the distance in meters from the center $H$ of the heliostat to the center of the receiver $R$:

$$
\newcommand{\norm}[1]{\left\lVert#1\right\rVert}
d = \norm{H - R}
$$

The ***atmospheric attenuation effect***, shown in <a>Figure 10</a>, is given by the formula:

$$
\eta_{aa}(d) = \begin{cases}
0.99321 - 1.176 \cdot 10^{-4} d + 1.97 \cdot 10^{-8} d^{2}, \quad d \leq 1000 \, m\\
\exp(-1.106 \cdot 10^{-4} d), \quad d > 1000 \, m
\end{cases}
$$

<div id="figure10" class="imgcap-nbd">
  <img src="/blog/assets/posts/layout-optimization/drawings/AA-crop.png" width="50%" class="invertImg">
  <div class="thecap">
    Figure 10: Atmospheric attenuation effect.
  </div>
</div>

## Optimal position for a single heliostat
To determine the optimal position for the center $H(\rho, d)$ of a single heliostat we define the following objective function called ***daily energy production*** of a power plant:

{: .note-title }
>  Daily Energy Production (DEP)
> 
> $$
> \text{DEP}(\rho, d) = 
> \frac{1}{\pi} \,
> \eta_{aa}(d) \, 
> \int_{0}^{\pi} \eta_{cos}(\rho, \phi) \, \eta_{spill}(\rho, \phi) \, d\phi
> $$

Note:

* The normalizing constant is $1/\pi$ because when there are no losses, all effects are 1 and $\int_{0}^{\pi} d\phi = \pi$. 

* In this simplified model, as the sun's position changes throughout the day, the solar angle $\phi$ ranges from 0 to $\pi$. This is why we integrate over $\phi$ from 0 to $\pi$.

To determine the optimal position for $H(\rho, d)$ that maximizes DEP, we need to find the optimal distance $d^{*}$ and optimal angle $\rho^{\*}$. 

Since $\eta_{aa}(d)$ is a monotonically decreasing function,

$$
\DeclareMathOperator*{\argmin}{arg\,min}
d^{*} = \argmin_{d} \norm{ H - R }
\quad
\text{ subject to: }
\norm{ H - R } > 4 \, m.
$$

In our model, the heliostat and receiver are both 4 meters in size and the rotation of heliostats adds a constraint $\norm{ H - R } > 4 \, m.$ (See <a href="#figure11">Figure 11B</a>.) Therefore, the optimal distance is $d^{*} = 4 \, m$.

Plugging into DEP the derived equations \ref{eq: cosine-effect} and \ref{eq: spillage-effect} for $\eta_{cos}$ and $\eta_{spill}$ and integrating, we get DEP with respect to $\rho$ shown in <a href="#figure11">Figure 11A</a>. The optimal angle $\rho$ is approximately $30^{\circ}$.

The optimal position for a single heliostat is shown in <a href="#figure11">Figure 11B</a>.


<div id="figure11" class="imgcap-nbd">
  <img src="/blog/assets/posts/layout-optimization/drawings/solution-singleton.svg" width="80%" class="invertImg">
  <div class="thecap">
    Figure 11: <strong>(A)</strong> Optimal angle $\rho$ and <strong>(B)</strong> optimal position for a single heliostat.
  </div>
</div>

## Multiple heliostats

### Shading and blocking effects
In case of multiple heliostats, individual heliostats can be blocked or shaded by neighboring heliostats, which affects the efficiency of the power plant (<a href="#figure12">Figure 12</a>). The effects of shading and blocking are defined as

$$
\begin{align*}
\eta_{shade} &= \text{ proportion of incoming rays not shaded by neighboring heliostats} \\[.5em]
\eta_{block} &= \text{ proportion of reflected rays not blocked by neighboring heliostats}.
\end{align*}
$$

<div id="figure12" class="imgcap-nbd">
  <img src="/blog/assets/posts/layout-optimization/drawings/shading-and-blocking-effects.svg" width="80%" class="invertImg">
  <div class="thecap">
    Figure 12: The effects of shading (<strong>A</strong>) and blocking (<strong>B</strong>).
  </div>
</div>

### Discretization and ray tracing
Finding the optimal layout using only geometry is challenging when there are multiple heliostats.Therefore, discretization and ray tracing are used to estimate the reflected power of each heliostat that is hitting the receiver.

For each heliostat, we divide the surface of the heliostat into equal-area regions. We then trace a ray from each region in the direction of the sun and in the direction of the receiver. <a href="#figure13">Figure 13</a> shows ray tracing for $m = 5$ rays.

<div id="figure13" class="imgcap-nbd">
  <img src="/blog/assets/posts/layout-optimization/drawings/ray-tracing.svg" width="80%" class="invertImg">
  <div class="thecap">
    Figure 13: Tracing $m = 5$ rays from the surface of the heliostat towards the sun and towards the receiver.
  </div>
</div>

Denote by $m$ the number of rays per heliostat and by $\eta_{ray, i}$ the proportion of rays that are received at the surface of heliostat $i$ and hit the surface of the receiver.


### The layout optimization problem
For a given plant with $n$ heliostats with centers:

$$ H_{1}, H_{2}, \dots, H_{n} $$

we approximate the daily energy production by

$$
\widehat{\text{DEP}}(H_{1}, H_{2}, \dots, H_{n}) = 
\frac{1}{n \, m}
\sum_{i = 1}^{n} \eta_{aa, i} \, 
\sum_{k = 1}^{m} \eta_{cos, i}(\phi_{k}) \, \eta_{ray, i}.
$$

We can now define the layout optimization problem in a more formal manner.

{: .note-title }
> Layout optimization problem
>
> Determine the locations $H_{i}$ of heliostats $i = 1, 2, \dots, n$, that maximize the $\widehat{\text{DEP}}$ subject to:
> 
> * all $H_{i}$ are in the specified field area,
> * all $H_{i}, H_{j}$ where $i \neq j$ are at least $4$ meters appart,
> * $H_{i}, R$ are at least $4$ meters appart.

An example of a valid layout for $n = 5$ heliostats is in <a href="#figure14">Figure 14</a>. Here the field area is specified as follows. Maximum distance from receiver's center to a heliostat's center is 35 meters and maximum height of heliostat's center is 10 meters. 

<div id="figure14" class="imgcap-nbd">
  <img src="/blog/assets/posts/layout-optimization/drawings/valid-layout.svg" width="60%" class="invertImg">
  <div class="thecap">
    Figure 14: Field area and a valid layout for $n = 5$ heliostats.
  </div>
</div>

### Solving the layout optimization problem
Note that the objective function $\widehat{\text{DEP}}$ is not convex as indicated by <a href="#figure15">Figure 15</a>. It shows the plot of $\widehat{\text{DEP}}$ as we vary the location of $H_{3}$ in the layout shown in <a href="#figure14">Figure 14</a>.

<div id="figure15" class="imgcap-nbd">
  <img src="/blog/assets/posts/layout-optimization/drawings/landscapes-min.png" width="100%" class="invertImg">
  <div class="thecap">
    Figure 15: Plot of $\widehat{\text{DEP}}(H_{3})$ as we vary the location of $H_{3}$.
  </div>
</div>

Starting far from the global optimum and running a gradient ascent can lead us to a sub-optimal solution. Similarly with higher order methods. Despite this, we can still observe some interesting results.

#### Example
Running Sequential Quadratic Programming (SQP) method from a class of Lagrange-Newton methods starting from some initial layout with 3 heliostats, the method converges to the layout show in <a href="#figure16">Figure 16</a>. See the [Source code](#source-code) for more details.

<div id="figure16" class="imgcap-nbd">
  <img src="/blog/assets/posts/layout-optimization/drawings/sqp-solution.svg" width="60%" class="invertImg">
  <div class="thecap">
    Figure 16: Converged layout using SQP for $n = 3$ heliostats.
  </div>
</div>

It places the heliostats some distance appart on the parabola with focal point $R$:

$$
y = \frac{1}{48} x^{2}.
$$

The layout we get for $n=5$ heliostats is shown in <a href="#figure1">Figure 1</a>. Intuitively this seems to be the best option. Generally speaking, the heliostats do not need to rotate so much to reflect the rays towards the receiver. Furthermore, the heliostats should be far apart in order to minimize the effects of shading and blocking. In particular, the effects of shading and blocking are most noticable when the cosine effect is minimized, that is when $\phi$ is roughly 90 degrees. In <a href="#figure17"> Figure 17</a> is sketch of the shadow from heliostat $H_{j}$ considering the values of $\phi$ close to 90 degrees.

<div id="figure17" class="imgcap-nbd">
  <img src="/blog/assets/posts/layout-optimization/drawings/shadow.svg" width="60%" class="invertImg">
  <div class="thecap">
    Figure 17: Sketch of the shadow from heliostat $H_{j}$ for values of $\phi$ close to 90 degrees.
  </div>
</div>

## Source code
The repository containing all Python scripts is available here:

<div class="mono-italic">
<a href="https://github.com/markolalovic/math-mods-camp">https://github.com/markolalovic/math-mods-camp</a>
</div>

The code for [Example](#example) above: 
```python
import sys
import numpy as np
from scipy import optimize

# load the specifications of the plant
hypo_plant = utils.load("../data/plants/tiny-plant.json")

# set the initial layout
basic_layout = np.array([[10, 10], [20, 5], [30, 10]])
x0 = basic_layout.flatten()

# objective function
def f(x):
    plant.layout = x.reshape((3, 2))
    plant.set_layout()
    return -utils.get_energy(plant)

# set the bounds and constraints
bounds = []
for i in range(3):
    bounds.append((0, 35))
    bounds.append((0, 10))    

def g01(x):
    i, j = 0, 1
    layout = x.reshape(3, 2)
    distance = np.linalg.norm(layout[i] - layout[j])
    return distance - 4

def g02(x):
    i, j = 0, 2
    layout = x.reshape(3, 2)
    distance = np.linalg.norm(layout[i] - layout[j])
    return distance - 4

def g12(x):
    i, j = 1, 2
    layout = x.reshape(3, 2)
    distance = np.linalg.norm(layout[i] - layout[j])
    return distance - 4

constraints = [{"type": "ineq", "fun": g01},
               {"type": "ineq", "fun": g02},
               {"type": "ineq", "fun": g12}]

result = optimize.minimize(f, x0, 
                           method="SLSQP", 
                           bounds=bounds,
                           constraints=constraints,
                           options={'disp': True, 'maxiter': 100})
# Optimization terminated successfully    (Exit mode 0)
#             Current function value: -41.033956692960274
#             Iterations: 28
#             Function evaluations: 317
#             Gradient evaluations: 28
```

## References

{% bibliography --file {{ page.refs }} --cited %}