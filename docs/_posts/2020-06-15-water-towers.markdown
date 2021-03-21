---
layout: post
title:  "Some Math Behind Water Towers"
subtitle: "Finding the optimal arrangement of metal rings to reinforce the cylindrical structure of a water tank."
author: Marko Lalović
date:   2020-06-15 21:26:36 +0100
image: assets/posts/water-towers/water-towers-small.jpg
categories: Physics NYC Architecture
rtime: 2 min read
---
<div class="images">
  <img src="assets/posts/water-towers/water-towers.jpg">
  <div class="label">
    <strong>Figure 1:</strong> Water towers in New York. (Source:  <a target="_blank" href="https://www.flickr.com/people/38782010@N00">takomabibelot</a>,
<a target="_blank" href="https://creativecommons.org/licenses/by/2.0/">CC-BY 2.0</a>, edited by author.)
  </div>
</div>

The water towers are interesting to me because their structure reveals some math behind how they are built. In **Figure 1**, we see that the water tanks are reinforced with metal rings to hold the cylindrical structure together. We also notice that the metal rings are closer to each other at the bottom of the tank where the pressure is greatest. But what is the optimal arrangement?

Since I did not find a derivation anywhere on the internet, I will describe my solution here with a simple example.

## Example

Say we have a wooden water tank that we want to reinforce with 5 equal metal rings so that the wood does not give in to the water pressure. How would you arrange the metal rings?

<div class="images">
  <img src="assets/posts/water-towers/drawing.svg">
  <div class="label">
    <strong>Figure 2:</strong> Two examples of arrangements of metal rings.
  </div>
</div>

Two examples of arrangements are shown in **Figure 2**. Let $d=3$ be the depth of the tank. Denote by $h_{k}$ for $k=1, …, 5​​$ the height of each metal ring from the bottom of the tank.

**Figure 2A** shows a naïve straightforward approach, where the metal rings are spaced evenly at heights from the bottom of the tank:

$$
h_{1} = 0.5, \quad h_{2} = 1, \quad h_{3} = 1.5, \quad h_{4} = 2,
\quad h_{5} = 2.5.
$$

The pressure that the water exerts increases with depth and the metal rings at the bottom of the tank have to hold a lot more pressure than the metal rings at the top of the tank. Therefore the structural strength of the lower part of the tank is lower than that of the upper part of the tank. Hence, this arrangement of metal rings is a poor choice.

### Optimal Arrangement

We derive the optimal arrangement of the metal rings shown in **Figure 2B**. The influence of depth on water density is negligible. We also assume that the water temperature in the tank is the same and constant everywhere. The water pressure $p$ at depth $x$ is equal to:

$$
p(x) = p_{0} + \rho \cdot g \cdot x
$$

where $p_{0}$ is the pressure at the surface, $\rho$ is the density of water and $g$ is the gravitational acceleration. The important part to notice for this derivation, is that the pressure exerted by a static liquid increases linearly with increasing depth:

$$
p(x) \propto C \cdot x
$$

for some constant $C > 0$. We choose the constant $C = 2/9$ and define the density:

$$
f_{X}(x) = \frac{2}{9} \cdot x
$$

of some continuously distributed random variable $X$ that represents the pressure in the tank, defined on the interval $[0, 3]$. The corresponding distribution function is then:

$$
F_{X}(x) = \frac{1}{9} \cdot x^{2}.
$$

Define the limits of the interval $[0, 3]$:

$$
x_{1} = 0, \quad x_{6} = 3
$$

and divide the interval $[0, 3]$ into 5 subintervals with boundaries:

$$
x_{2} = x_{0.2}, \quad x_{3} = x_{0.4}, \quad x_{4} = x_{0.6}, \quad
x_{5} = x_{0.8}
$$

where $x_{q}$ are quantiles, that is:

$$
F_{X}(x_{q}) = q.
$$

The boundaries are marked in dotted red in **Figure 2B**. At each subinterval, the integral of the density is equal to 0.2. This method, of dividing a continuous distribution into subintervals with equal density, is called *Equal Frequency Discretization*.

Finally, we arrange the metal rings so that each ring is exactly in the centroid $h_{k}$​ of density $f_{X}$ at the subinterval $k$:

$$
h_{k} = \frac{\int_{x_{k}}^{x_{k+1}} x f(x) dx}{\int_{x_{k}}^{x_{k+1}} f(x) dx}.
$$

This way, each metal ring needs to hold the same amount of pressure. Therefore the structural strength is the same everywhere and this arrangement of metal rings is optimal.

In the optimal arrangement, the calculated heights of the metal rings from the bottom of the tank are approximately:

$$
h_{1} = 0.16, \quad h_{2} = 0.49, \quad h_{3} = 0.88, \quad
h_{4} = 1.36, \quad h_{5} = 2.11.
$$

The solution is shown in **Figure 2b**.

<!-- The website content is licensed <a href="https://creativecommons.org/licenses/by/4.0/" target="_blank">CC-BY 4.0</a>. -->


[mayavi-lib]: https://docs.enthought.com/mayavi/mayavi/
[1_original]: assets/posts/tda-digits/1_original-image.png
[2_binary]: assets/posts/tda-digits/2_binary-image.png
[3_skeleton]: assets/posts/tda-digits/3_skeleton.png
[4_points]: assets/posts/tda-digits/4_points.png
[5_embedded-graph]: assets/posts/tda-digits/5_embedded-graph.png
[6_simplices]: assets/posts/tda-digits/6_simplices.png
