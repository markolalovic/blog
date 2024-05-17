---
layout: post
author: Marko Lalovic
title:  "Some math behind water towers"
subtitle: "Finding the optimal arrangement of metal rings to reinforce the cylindrical structure of a water tank."
nav_order: 5
date: "2020-06-15"
keywords: layout optimization
---
The structure of a water tower reveals some math behind how they are build. <a href="#figure1">Figure 1</a> shows some water towers reinforced with metal rings to hold the cylindrical structure of the water tanks together. But what is the optimal arrangement of metal rings?

<div id="figure1" class="imgcap-nbd">
  <img src="/blog/assets/posts/water-towers/water-towers.jpg" width="70%">
  <div class="thecap">
    Figure 1: Water towers in New York. Image credit: © <a target="_blank" href="https://www.flickr.com/people/38782010@N00">takomabibelot</a> licensed <a target="_blank" href="https://creativecommons.org/licenses/by/2.0/">CC-BY 2.0</a>.
  </div>
</div>

Since I didn't find a derivation anywhere on the internet, I will describe my solution in this blog post.

#### Example
Suppose we have a wooden water tower that we want to reinforce with $K=5$ equal metal rings so that the wood does not give in to the water pressure. Let $d=3$ be the depth of the water tank. Denote by $h_{k}$ for $k=1, …, 5​​$ the height of each metal ring from the bottom of the tank. Two examples of arrangements are shown in <a href="#figure2">Figure 2</a>.

<div id="figure2" class="imgcap">
  <img src="/blog/assets/posts/water-towers/drawing.svg" width="70%"  class="invertImg">
  <div class="thecap">
    Figure 2: Two examples of arrangements of metal rings.
  </div>
</div>

<a href="#figure2">Figure 2A</a> shows a naïve approach, where the metal rings are spaced evenly at heights from the bottom of the tank:

$$
h_{1} = 0.5, \quad h_{2} = 1, \quad h_{3} = 1.5, \quad h_{4} = 2,
\quad h_{5} = 2.5.
$$

The pressure that the water exerts increases with depth and the metal rings at the bottom of the tank have to hold a lot more pressure than the metal rings at the top of the tank. Therefore the structural strength of the lower part of the tank is lower than that of the upper part of the tank. Hence, this arrangement of metal rings is a poor choice.

In <a href="#figure2">Figure 2B</a>, the metal rings are closer to each other at the bottom of the tank where the pressure that the water exerts on the tank is the highest. 

## Optimal arrangement
To derive the optimal arrangement of the metal rings shown in <a href="#figure2">Figure 2B</a>, let's assume that the water temperature in the tank is constant and the same everywhere. Also, the influence of depth on water density is negligible. Then the water pressure $p$ at depth $x$ is equal to:

$$
p(x) = p_{0} + \rho \cdot g \cdot x
$$

where $p_{0}$ is the pressure at the surface, $\rho$ is the density of water and $g$ is the gravitational acceleration. The important part to notice for this derivation, is that the pressure exerted by a static liquid increases linearly with depth

$$
p(x) \propto C \cdot x
$$

for some constant $C > 0$. Choose the constant $C = 2/9$ and define the density

$$
f_{X}(x) = \frac{2}{9} \cdot x
$$

of some continuously distributed random variable $X$ that represents the pressure in the tank defined on the interval $[0, 3]$. The corresponding distribution function is then:

$$
F_{X}(x) = \frac{1}{9} \cdot x^{2}.
$$

Define the limits of the interval $[0, 3]$ to be

$$
x_{1} = 0 \quad \text{ and } \quad x_{6} = 3
$$

and divide the interval $[0, 3]$ into 5 subintervals with boundaries:

$$
x_{2} = x_{0.2}, \quad x_{3} = x_{0.4}, \quad x_{4} = x_{0.6}, \quad
x_{5} = x_{0.8}
$$

where $x_{q}$ are quantiles, that is

$$
F_{X}(x_{q}) = q
$$

The boundaries are marked as dashed red lines in <a href="#figure2">Figure 2B</a>. At each subinterval, the integral of the density is equal to 0.2. This method, of dividing a continuous distribution into subintervals with equal density, is called *equal frequency discretization*.

Finally, we arrange the metal rings so that each ring is exactly in the centroid $h_{k}$​ of density $f_{X}$ at the subinterval $k$

$$
h_{k} = \frac{\int_{x_{k}}^{x_{k+1}} x \cdot f(x) dx}{\int_{x_{k}}^{x_{k+1}} f(x) dx}
$$

This way, each metal ring needs to hold the same amount of pressure. Therefore the structural strength is the same everywhere and this arrangement of metal rings is optimal.

In the optimal arrangement, the calculated heights of the metal rings from the bottom of the tank are approximately

$$
h_{1} = 0.16, \quad h_{2} = 0.49, \quad h_{3} = 0.88, \quad
h_{4} = 1.36, \quad h_{5} = 2.11.
$$

Using this procedure, it is possible to calculate the optimal arrangement for any number of metal rings. For example, in <a href="#figure1">Figure 1</a>, the water tower in the front uses 11 metal rings.