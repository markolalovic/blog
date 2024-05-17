---
layout: post
author: Marko Lalovic
title:  "Topological features applied to the MNIST data set"
subtitle: "A tutorial on extracting topological features that can be used as input to standard algorithms to obtain qualitative geometric information."
date: "2020-09-25"
nav_order: 6
keywords: topological data analysis, machine learning
refs: tda-digits
---
The purpose of this tutorial is to give a friendly introduction on how to use the persistent homology that does not require substantial knowledge of topological methods. To illustrate the use of persistent homology in machine learning we apply it to the MNIST data set of handwritten digits. A very similar approach can be applied to any point cloud data and can be generalized to higher dimensions.

A pure topological classification cannot distinguish between individual numbers, as the numbers are topologically too similar. For example, numbers 6 and 9 are topologically the same if we use this style for writing numbers. Persistent homology, however, gives us more information. The process is described as follows and illustrated in <a href="#figure1">Figure 1</a>. Define a filtration on the vertices of the graph corresponding to the image pixels, adding vertices and edges as we sweep across the image (<a href="#figure1">Figure 1A,B</a>). This adds spatial information to the topological features. For example, though 6 and 9 both have a single loop, it will appear at different locations in the filtration. Then, compute the persistent homology given the simplex stream from the filtration to get the finite set of intervals called *Betti barcodes* (<a href="#figure1">Figure 1C</a>).

<div class="mono-italic">
Click to animate the diagram:
</div>
<button class="btn btn-blue" title="Run" id="runButton" onclick="run()">RUN</button>
{% include posts/tda-digits/tda-example.html %}

Finally, extract 4 features from the k-dimensional barcode from the invariants discussed in {% cite Adcock2013 --file {{ page.refs }} %}. For each of 4 sweep directions: top, bottom, right, left and dimensions: 0 and 1, compute 4 features. This gives a total of 32 features per image. On the extracted features from a set of images, we then apply a support vector machine (SVM) learning algorithm to classify the images.

This blog post starts by a short introduction to Topological Data Analysis and Computational Topology. Next, the extraction of topological features is explained. Finally, the empirical classification results on a subset of the MNIST database are presented including the evaluation. 

The aim is to demonstrate the classification potential of the technique and not to outperform the existing models. For a more interesting example of using this technique on a clinical data set to classify hepatic lesions, see {% cite Adcock2013 --file {{ page.refs }} %}.

All scripts are available on GitHub <a href="https://github.com/markolalovic/tda-digits">repository</a> including a processed version of the dataset. Also, freely available computational topology package <a href="https://github.com/mrzv/dionysus">Dionysus 2</a> is used for the computation of persistent homology.

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

## Introduction
Persistent homology is a fascinating mathematical tool that continues to be studied, developed, and applied. Topology applied to real-world data sets using persistent homology has begun to look for applications in machine learning, including deep learning {% cite Gabrielsson2020 --file {{ page.refs }} %}. Topology is mainly used in a pre-processing step to provide robust features for learning. The developed topological techniques, mostly deal with point clouds, i.e. finite sets of data points in space. Point clouds are typically captured by a variety of imaging devices, such as MRI or CT scanners. With the greater availability of such devices, this type of data is being generated at an increasing rate. Our data is often a finite set of noisy samples from some underlying space. The data sets are often very noisy and contain a lot of missing information, especially biological data sets. Our ability to analyze this data, both in terms of the amount and the nature of the data, is getting out of step with the data we generate {% cite Carlsson2009 --file {{ page.refs }} %}. Topology can be used to make a useful contribution to the analysis of such data sets and it is especially helpful in studying them qualitatively.

The problem we are trying to solve in this tutorial is how to extract the topological features that can be used as an input to standard machine learning algorithms. We will use a similar approach as described in {% cite Adcock2013 --file {{ page.refs }} %}. From each image, we first construct a graph, where pixels of the image correspond to vertices of the graph and we add edges between adjacent pixels (<a href="#figure2">Figure 2A,B</a>). We then extract 0- and 1-dimensional topological features called *Betti numbers* denoted by $\beta_{0}$ and $\beta_{1}$. For example, a torus has one connected component giving $\beta_{0} = 1$, and two cycles or loops, giving $\beta_{1} = 2$ (<a href="#figure2">Figure 2C</a>).

<div id="figure2" class="imgcap-nbd">
  <img src="/blog/assets/posts/tda-digits/intro-figure.png" width="60%" class="invertImg">
  <div class="thecap">
    Figure 2: (A) handwritten digits (B) preprocessing and (C) a torus with two cycles on its surface.
  </div>
</div>

## Terminology

* *Topology* is a branch of mathematics that deals with qualitative geometric information. This includes the classification of loops and higher-dimensional surfaces.
* *Topological data analysis* and *computational topology* deal with the study of topology using a computer.
* *Persistent homology* is an algebraic method for discerning topological features of data.
* *Connected component* (or connected cluster of points) is a 0-dimensional feature and *cycle* (or *loop*) is a 1-dimensional feature.
* *Simplicial complex* is a set composed of points, line segments, triangles, and their n-dimensional counterparts.
* *Filtration* is the sequence of simplicial complexes, with an inclusion map from each simplicial complex to the next.
* *Barcode* is a visual representation of the persistence of the topological features. Longer bars represent significant features of the data. Shorter bars are due to irregularities or noise.

## Extracting the graph structure
The pre-processing steps to expose the topology of the digits, shown in the image of a handwritten digit 8 in <a href="#figure3">Figure 3</a>, are as follows.

* Load the MNIST image of a handwritten digit.
* Produce a binary image by thresholding.
* Reduce the image to a skeleton of 1-pixel width using the popular <a href="https://github.com/linbojin/Skeletonization-by-Zhang-Suen-Thinning-Algorithm">Zhang-Suen</a> thinning algorithm.
* Construct a graph $G$ from the skeleton.

<div id="figure3" class="imgcap">
  <img src="/blog/assets/posts/tda-digits/1_original-image.png" alt="original image" style="width:23%" class="invertImg">
  <img src="/blog/assets/posts/tda-digits/2_binary-image.png" alt="binary image" style="width:23%" class="invertImg">
  <img src="/blog/assets/posts/tda-digits/3_skeleton.png" alt="skeleton" style="width:23%" class="invertImg">
  <img src="/blog/assets/posts/tda-digits/4_graph.svg" alt="graph" style="width:21.2%" class="invertImg">
  <div class="thecap">
    Figure 3: Pre-procesing steps: the original image, binary image, skeleton and extracted graph.
  </div>
</div>

To construct the graph $G$ from the 1-pixel width skeleton, we treat the pixels of the skeleton as vertices of $G$ and add the edges between adjacent vertices and then remove all created cycles of length 3. Intuitively, we connect the points while trying not to create new topological features. The result for the image of a handwritten digit 8 is a graph in <a href="#figure3">Figure 3</a>. The resulting graph in this example consists of one connected component with two cycles, so no new topological features were added.

## Extracting the topological features
Construct a simplex stream for computing the persistent homology using the following filtration on the graph $G$ embedded in the plane. Sweep across the plane while adding the vertices and edges of graph $G$. From the simplex stream, compute the persistent homology to get the Betti barcodes. The <a href="https://github.com/mrzv/dionysus">Dionysus 2</a> package can be used for the computation of persistent homology. See the package documentation {% cite Dionysus2 --file {{ page.refs }} %} for details.

<a href="#figure4">Figure 4</a> shows the result of this technique applied on the image of a handwritten digit 8. In this example, we sweep across the plane to the top in a vertical direction $y \in (-\infty, \infty)$.

<div id="figure4" class="imgcap-nbd">
  <img src="/blog/assets/posts/tda-digits/tda-example.svg" width="80%" class="invertImg">
  <div class="thecap">
   Figure 4: Result of applying this technique on (A) an image of a handwritten digit 8, (B) extracted embedded graph and (C) the resulting Betti barcodes at $y = 22$.
  </div>
</div>
<div class="mono-italic">
Interactive example:
</div>
<div class="mono-italic">
<a href="https://markolalovic.github.io/tda-digits">https://markolalovic.github.io/tda-digits</a>
</div>

Betti 0 barcode $\beta_{0}$ consists of one interval $[2, \infty)$, which clearly shows the single connected component with the birth time of 2 when the first vertex of $G$ is added.

Betti 1 barcode $\beta_{1}$ consists of two intervals $[10, \infty)$ and $[18, \infty)$, with birth times of 10 and 18 corresponding to the births of the two cycles. The birth of the cycle is the value of $y$ when the loop closes in the embedded graph $G$ as we sweep to the top.

Finally, extract the following features from each of the k-dimensional Betti barcodes in the following way. Denote the endpoints of Betti barcode intervals with:

$$
x_{1}, y_{1}, ..., x_{n}, y_{n}
$$

where $x_{i}$ represents the beginning and $y_{i}$ the end of each interval. From the endpoints compute 4 features from the invariants discussed in {% cite Adcock2013 --file {{ page.refs }} %} that take into account all of the bars lengths and endpoints:

$$
\begin{align*}
f_{1}&=\sum_{i} x_{i} (y_{i} - x_{i}) \\
f_{2}&=\sum_{i} (y_{\max} - y_{i})(y_{i} - x_{i}) \\
f_{3}&=\sum_{i} (y_{\max} - y_{i})^{2} (y_{i} - x_{i})^{4} \\
f_{4}&=\sum_{i} x_{i}^{2} (y_{i} - x_{i})^{4} \\
\end{align*}
$$

For each of the 4 sweep directions: top, bottom, right, left and for each $k$-dimensional Betti barcode, $k = 0, 1$, we compute the defined 4 features $f_{1}, \dots, f_{4}$. This gives us a total of $4 \cdot 2 \cdot 4 = 32$ features per image.

## Empirical results
Data set consisted of extracted topological features of 10000 images of handwritten digits from MNIST database. Data set was split 50:50 in train and test set so that each had 5000 examples. SVM with RBF kernel was used for classification of the images based on the extracted topological features. The empirical classification results are as follows. Accuracy on the train set using 10-fold cross-validation was $0.88 (\pm 0.05)$. Accuracy on the test set was 0.89.

We now examine the common misclassifications. There were only 3 examples of the number 2 being mistaken for number 0, they are shown in <a href="#figure5">Figure 5</a>. The reason is that the number 2 was written with a loop that appears in the region that is close to the loop in number 0.

<div id="figure5" class="imgcap-nbd">
  <img src="/blog/assets/posts/tda-digits/miss-2.png" width="60%" class="invertImg">
  <div class="thecap">
    Figure 5: Examples of number 2 being mistaken for number 0.
  </div>
</div>

For number 5 we got the lowest F1 score of 0.75. It was misclassified as number 2 in 32 examples in the test set. The first three examples are shown in <a href="#figure6">Figure 6</a>. This was expected since these two numbers are topologically the same with no topological features (e.g. loops) appearing in different regions.

<div id="figure6" class="imgcap-nbd">
  <img src="/blog/assets/posts/tda-digits/miss-5.png" width="60%" class="invertImg">
  <div class="thecap">
    Figure 6: Examples of number 5 being mistaken for number 2.
  </div>
</div>

Number 8 was misclassified as number 4 in 21 examples from the test set. The first three examples are shown in <a href="#figure7">Figure 7</a>. We see the stylistic problems that caused the misclassifications. The top loop of number 8 was not closed which made it topologically more similar to number 4 written with a loop.

<div id="figure7" class="imgcap-nbd">
  <img src="/blog/assets/posts/tda-digits/miss-8.png" width="60%" class="invertImg">
  <div class="thecap">
    Figure 7: Examples of number 8 being mistaken for number 4.
  </div>
</div>

## Source code
The repository containing all Python scripts for this tutorial including the processed version of the data set is available here:

<div class="mono-italic">
<a href="https://github.com/markolalovic/tda-digits">https://github.com/markolalovic/tda-digits</a>
</div>

## References

{% bibliography --file {{ page.refs }} --cited %}



[1_original]: /blog/assets/posts/tda-digits/1_original-image.png
[2_binary]: /blog/assets/posts/tda-digits/2_binary-image.png
[3_skeleton]: /blog/assets/posts/tda-digits/3_skeleton.png

