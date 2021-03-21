---
layout: post
title:  "Topological Features Applied to the MNIST Data Set"
subtitle: "Extraction of topological features that can be used as an input to standard algorithms to obtain qualitative geometric information."
author: Marko Lalović
date:   2020-09-25 21:26:36 +0100
image: assets/posts/tda-digits/intro-figure.jpg
keywords: Topological-Data-Analysis Machine-Learning
rtime: 5 min read
---
<div class="images">
  <img src="/blog/assets/posts/tda-digits/intro-figure.jpg">
  <div class="label">
    <strong>Figure 1:</strong> Illustration of the main ideas. (Torus was drawn using <a target="_blank" href="https://docs.enthought.com/mayavi/mayavi/">Mayavi library</a>.)
  </div>
</div>

<!-- 8 min read -->
This is part of the project I made for Summer School on Computational Topology and Topological Data Analysis held in Ljubljana. I made publicly available all <a href="https://github.com/markolalovic/tda-digits" target="_blank">scripts</a> that I wrote for this tutorial including a processed version of the dataset. I am also using freely available computational topology package called <a href="https://github.com/mrzv/dionysus" target="_blank">Dionysus 2</a> for the computation of persistent homology.

## Preface

Topology applied to real-world data sets using persistent homology has begun to look for applications in machine learning, including deep learning [<a href="https://arxiv.org/abs/1304.0530" target="_blank">2</a>]. Topology is mainly used in a pre-processing step to provide robust features for learning. Our data is often a finite set of noisy samples from some underlying space. The developed topological techniques, mostly deal with point clouds, i.e. finite sets of data points in space. Point clouds are typically captured by a variety of imaging devices, such as MRI or CT scanners. With the greater availability of such devices, this type of data is being generated at an increasing rate. The data sets are often also very noisy and contain a lot of missing information, especially biological data sets. Our ability to analyze this data, both in terms of the amount and the nature of the data, is clearly out of step with the data we generate [<a href="https://arxiv.org/pdf/1905.12200.pdf" target="_blank">3</a>]. Topology can be used to make a useful contribution to the analysis of such data sets and it is especially helpful in studying them qualitatively.

Persistent homology is a fascinating mathematical tool that continues to be studied, developed, and applied. The purpose of this article is to give a friendly introduction on how to use the persistent homology that does not require substantial knowledge of topological methods. To illustrate the use of persistent homology in machine learning we apply it to the MNIST data set of handwritten digits. A very similar approach can be applied to any point cloud data and it can easily be generalized to higher dimensions. **Figure 1** illustrates the main ideas of the proposed technique. It shows a sample of handwritten digits on the left and extracted graph structure on the right. Underneath is a picture of a torus in 3D and the extracted point cloud with 2 colored loops on the right.

First, we explain the extraction of topological features on a simple example
shown in **Figure 2**. Second, we present and evaluate the empirical classification results on a subset of the MNIST database. The aim is to demonstrate the classification potential of the technique and not to outperform the existing models for the classification of handwritten digits. For a more interesting example of using this technique on a clinical data set to classify hepatic lesions, see [<a href="https://arxiv.org/abs/1304.0530" target="_blank">1</a>].

<div class="images">
  <img src="/blog/assets/posts/tda-digits/anim-compressed.gif">
  <div class="label">
    <strong>Figure 2:</strong> An example of applying this technique on the image of a handwritten digit 8.
  </div>
</div>

### Terminology

A list of short definitions which we will expand later if necessary:

* *Topology* is a branch of mathematics that deals with qualitative geometric information. This includes the classification of loops and higher-dimensional surfaces.
* *Topological data analysis* and *computational topology* deal with the study of topology using a computer.
* *Persistent homology* is an algebraic method for discerning topological features of data.
* *Connected component* (or connected cluster of points) is a 0-dimensional feature and *cycle* (or *loop*) is a 1-dimensional feature.
* *Simplicial complex* is a set composed of points, line segments, triangles, and their n-dimensional counterparts.
* *Filtration* is the sequence of simplicial complexes, with an inclusion map from each simplicial complex to the next.
* *Barcode* is a visual representation of the persistence of the topological features. Longer bars represent significant features of the data. Shorter bars are due to irregularities or noise.

## Methods

The main problem we are trying to solve is how to extract the topological features that can be used as an input to standard machine learning algorithms. We will use a similar approach as described in [<a href="https://arxiv.org/abs/1304.0530" target="_blank">1</a>]. From each image, we first construct a graph, where pixels of the image correspond to vertices of the graph and we add edges between adjacent pixels. A pure topological classification cannot distinguish between individual numbers, as the numbers are topologically too similar. For example numbers 1, 2, 3 are topologically the same if we use this style for writing numbers. Persistent homology, however, gives us more information.

**Overview.** We define a filtration on the vertices of the graph corresponding to the image pixels, adding vertices and edges as we sweep across the image in the vertical or horizontal direction. This adds spatial information to the topological features. For example, though 6 and 9 both have a single loop, it will appear at different locations in the filtration. We then compute the persistent homology given the simplex stream from the filtration to get the so-called Betti barcodes. *Betti* $k$ *barcode* is a finite set of intervals. Each interval represents the first filtration level where the topological feature of dimension $k$ appears and the filtration level where it disappears. These times are called *birth* and *death* of the topological feature respectively. We extract 4 features from the $k$-dimensional barcode from the invariants discussed in [<a href="https://arxiv.org/abs/1304.0530" target="_blank">1</a>]. For each of 4 sweep directions: top, bottom, right, left and dimensions: 0 and 1, we compute 4 features. This gives us a total of 32 features per image. On the extracted features from a set of images, we then apply a support vector machine (SVM) to classify the images.


### Pre-Processing

The pre-processing steps used, shown in **Figure 3**, are the following:

1. Load the MNIST image of a handwritten digit.
2. Produce the binary image by thresholding.
3. Reduce the binary image to a skeleton of 1-pixel width to expose its topology using the popular <a href="https://github.com/linbojin/Skeletonization-by-Zhang-Suen-Thinning-Algorithm" target="_blank">Zhang-Suen thinning algorithm</a>.
4. Transform the pixels of the skeleton to points in the plane.
5. Construct an embedded graph G in the plane where we treat the points as vertices and add edges between adjacent points similar and then remove all cycles of length 3. Intuitively, we connect the points while trying not to create new topological features.
6. Construct a simplex stream for computing the persistent homology using the filtration on the vertices of the graph G. Filtration is the following. We are adding the vertices and edges of the embedded graph G as we sweep across the plane. In this example, we sweep across the plane to the top in a vertical direction.

| ![1_original-image.png][1_original] | ![2_binary-image.png][2_binary] | ![3_skeleton.png][3_skeleton] |
|:--:|:--:|:--:|
| <span class="subfig">(1) Original image</span> | <span class="subfig">(2) Binary image</span> | <span class="subfig">(3) Skeleton</span> |
| ![4_points.png][4_points] | ![5_embedded-graph.png][5_embedded-graph] | ![6_simplices.png][6_simplices] |
| <span class="subfig">(4) Points</span> | <span class="subfig">(5) Embedded graph</span> | <span class="subfig">(6) Simplices</span> |

<div class="images">
  <div class="label">
    <strong>Figure 3:</strong> Pre-processing steps.
  </div>
</div>


### Extraction of Topological Features

Given the simplex stream from the final pre-processing step, we compute the persistent homology to get the Betti barcodes shown in **Figure 4**. See also **Figure 2** which is using the same example.

<div class="images">
  <img src="/blog/assets/posts/tda-digits/betti-barcodes.png" width="500">
  <div class="label">
    <strong>Figure 4:</strong> Betti barcodes from the sweep to the top of the image of digit 8.
  </div>
</div>

Betti 0 barcode consists of one interval $[4.0, \infty)$, which clearly shows the single connected component with the birth time of 4 when the first point is detected. Betti 1 barcode consists of two intervals $[12, \infty)$ and $[20, \infty)$, with birth times 12 and 20 correspondingly to the births of 2 cycles (the value of y when the loop closes) in the drawing of digit 8 when we sweep to the top.

Denote the endpoints of Betti barcode intervals with:

$$
x_{1}, y_{1}, ..., x_{n}, y_{n}
$$

where $x_{i}$ represents the beginning and $y_{i}$ the end of each interval. From the endpoints we compute 4 features from the invariants discussed in [<a href="https://arxiv.org/abs/1304.0530" target="_blank">1</a>], that take into account all of the bars, lengths, and endpoints:

$$
\begin{align}
&\sum_{i} x_{i} (y_{i} - x_{i}) \\
&\sum_{i} (y_{\max} - y_{i})(y_{i} - x_{i}) \\
&\sum_{i} (y_{\max} - y_{i})^{2} (y_{i} - x_{i})^{4} \\
&\sum_{i} x_{i}^{2} (y_{i} - x_{i})^{4} \\
\end{align}
$$

For each of the 4 sweep directions: top, bottom, right, left and for each $k$-dimensional barcode, for $k = 0, 1$, we compute the defined 4 features. This gives us a total of $4 \cdot 2 \cdot 4 = 32$ features per image.


### Results

Data set consisted of extracted topological features of 10000 images of handwritten digits from MNIST database. Data set was split 50:50 in train and test set so that each had 5000 examples. SVM with RBF kernel was used for classification of the images based on the extracted topological features. The empirical classification results are as follows. Accuracy on the train set using 10-fold cross-validation was $0.88 (\pm 0.05)$. Accuracy on the test set was 0.89.

We examine the common misclassifications.

<div class="images">
  <img src="/blog/assets/posts/tda-digits/miss-2.png" width="500">
  <div class="label">
    <strong>Figure 5:</strong> Examples of number 2 being mistaken for number 0.
  </div>
</div>

There were 3 examples of the number 2 being mistaken for number 0, shown in **Figure 5**. The reason is that the number 2 was written with a loop that appears in the region that is close to the loop in number 0.

<div class="images">
  <img src="/blog/assets/posts/tda-digits/miss-5.png" width="500">
  <div class="label">
    <strong>Figure 6:</strong> Examples of number 5 being mistaken for number 2.
  </div>
</div>

For number 5 we got the lowest F1 score of 0.75. It was misclassified as number 2 in 32 examples in the test set. The first three examples are shown in **Figure 6**. This was expected since these two numbers are topologically the same with no topological features (e.g. loops) appearing in different regions.

<div class="images">
  <img src="/blog/assets/posts/tda-digits/miss-8.png" width="500">
  <div class="label">
    <strong>Figure 7:</strong> Examples of number 8 being mistaken for number 4.
  </div>
</div>

Number 8 was misclassified as number 4 in 21 examples from the test set. The first three examples are shown in **Figure 7**. We see the stylistic problems that caused the misclassifications. The top loop of number 8 was not closed which made it topologically more similar to number 4 written with a loop.


## Source Code

The repository containing all Python scripts that I wrote for this tutorial including the processed version of the data set is available <a href="https://github.com/markolalovic/tda-digits" target="_blank">here</a>.

It’s dependencies are:

* Python (2 or 3);
* <a href="https://github.com/mrzv/dionysus" target="_blank">Dionysus 2</a> for computing persistent homology;
* Boost version 1.55 or higher for Dionysus 2;
* Matplotlib for plotting the data;
* NetworkX for plotting the graphs;
* NumPy for loading data and computing;
* Scikit-learn for machine learning algorithms;
* Scikit-image for image pre-processing;
* <a target="_blank" href="https://docs.enthought.com/mayavi/mayavi/">Mayavi library</a> if you want to draw the torus from Figure 1 in 3D.

To generate the figures in the example of topological features extraction, run:

```bash
cd scripts
./tda_digits
```
For details on how to use the functions and classes see the Jupyter notebooks: `Example.ipynb` and `Classification.ipynb` from `scripts` directory.

## Acknowledgments
This is part of the project I made for Summer School on Computational Topology and Topological Data Analysis held in Ljubljana. The project was presented by Primoz Skraba.

## References
[1] A. Adcock, E. Carlsson, and G. Carlsson, “The Ring of Algebraic Functions on Persistence Bar Codes”, (2013), Homology, Homotopy and Applications
<a target="_blank" href="https://arxiv.org/abs/1304.0530">https://arxiv.org/abs/1304.0530</a>

[2] R. Bruel-Gabrielsson, B. J. Nelson, A. Dwaraknath, P. Skraba, L. J. Guibas, and G. Carlsson, “A Topology Layer for Machine Learning”, (2020), Proceedings of Machine Learning Research
<a target="_blank" href="https://arxiv.org/pdf/1905.12200.pdf">https://arxiv.org/pdf/1905.12200.pdf</a>

[3] G. Carlsson, “Topology and data”, (2009), Bulletin of the American Mathematical Society
<a target="_blank" href="https://www.ams.org/journals/bull/2009-46-02/S0273-0979-09-01249-X/S0273-0979-09-01249-X.pdf">https://www.ams.org/journals/bull/2009-46-02/S0273-0979-09-01249-X/S0273-0979-09-01249-X.pdf</a>

[mayavi-lib]: https://docs.enthought.com/mayavi/mayavi/
[1_original]: /blog/assets/posts/tda-digits/1_original-image.png
[2_binary]: /blog/assets/posts/tda-digits/2_binary-image.png
[3_skeleton]: /blog/assets/posts/tda-digits/3_skeleton.png
[4_points]: /blog/assets/posts/tda-digits/4_points.png
[5_embedded-graph]: /blog/assets/posts/tda-digits/5_embedded-graph.png
[6_simplices]: /blog/assets/posts/tda-digits/6_simplices.png
