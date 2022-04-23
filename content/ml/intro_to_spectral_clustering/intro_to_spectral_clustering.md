---
title: __TODO__ The Basics of Spectral Clustering
description: A beginner's introduction to the intuitive math behind spectral clustering and how it is useful versus other clustering methods.
date: 2018-04-10
categories: [ "Machine Learning" ]
tags: [
    "machine learning"
]
mathjax : true
katex: true
header-includes:
  - \usepackage{algorithm}
  - \usepackage{algorithmic}
  - \usepackage[noend]{algpseudocode}
---

A beginner's introduction to the intuitive math behind spectral clustering and how it is useful versus other clustering methods.
<!--more-->
$$\sum_{x=1}^5 y^z$$

$$
\begin{algorithm}[H]
\begin{algorithmic}[H]
x
\end{algorithmic}
\end{algorithm}
$$

$$
\begin{algorithm}[H]
\caption{Spectral Clustering}\label{spectralalg}
\begin{algorithmic}[H]
\State \textbf{\textit{Input:}} Weighted adjacency matrix $W \in \mathbb{R}^{N \times N}$, number of clusters $k$
\State Compute degree matrix $D$
\State $L \gets D - W$
\State Compute first $k$ eigenvectors $v_1, ..., v_k$  of $L$ and stack them into $X \in \mathbb{R}^{N \times k}$
\State $y_i \gets$ normalized ith row of $X \quad$   (note: $y_i \in \mathbb{R}^{k}$)
\State Cluster $(y_i)_{i=1,...,n}$ into clusters $C_1, ..., C_k$ using k-means
\State \textbf{\textit{Output:}} Clusters $A_1,...,A_k$ where $A_i = \{j | y_j \in C_i \}$
\end{algorithmic}
\end{algorithm}
$$


### Introduction

Many of the clustering techniques we have learned throughout the ML sequence in the MSCF program only work well for convex or *blob* type data. When the data is non-convex, the performance of these techniques decreases substantially. This happens because these methods use elliptical metrics to form the clustered groups. Following the canonical example of concentric crescents shown in Figure 1 we can see that k-means where $k=3$ would result in a clearly incorrect clustering. This is a well known problem and much work has been done on methods that overcome this issue such as hierarchical clustering using single-linkages. Spectral clustering has become one of the most popular methods to cluster non-convex data.

<center>
<img src="https://raw.githubusercontent.com/sibbs-gambling/spectral-clust-crypto/master/docs/writeup/nonconvex_sets.png" style="width:200px;"/>
</center>

As noted in Wang and Zhang (2005), time series are not amenable to being clustered using traditional methods such as k-means. They can be of varying length depending on the number of observations and are often very high dimensional. The authors have proposed spectral clustering as a means to get past these problems. Of the most important reasons to use spectral clustering in the case of time series is that the dimensionality does not affect the computational costs of running the algorithm. When one has an appropriate similarity measure or distance matrix, this can be fed into the algorithm without extra computational burden for even very high dimensional time series. For example if you had hourly time series for ten currencies over a week versus over a decade, given that one has a distance matrix for both, the spectral clustering takes the same amount of time in either case. Different methods can be used to compute the distance matrix, in our case we will use the same as for Minimum Spanning Trees. Below we guide the reader through an introduction to spectral clustering starting from the basic building blocks progressing to interpretations of the algorithm's output.

### Basics
Given a positive similarity metric $$s_{ij} := \text{similarity}(x_i, x_j) \geq 0$$  between points in a set, $S = $ { $x_1,x_2,..., x_N$}, clustering can be thought of as attempting to form groups where the in-group similarity metrics are high and the out-group similarity is low. An often used similarity metric is that of the radial basis function kernel (RBF). There is an entire sub-field of mathematics devoted to studying problems of this type, graph theory. Spectral clustering uses much of the methodology from this area.The basic building blocks are as follows.

  - We represent vertices (or nodes) as belonging to the set $V = $ { $v_1, ..., v_n$}
  - Let $E$ be the set of edges or connections between vertices, where each edge is weighted with some similarity metric $$w_{ij} = s_{ij}  \forall \hspace{0.1cm} i \neq j$$, $w_{ii}$ is often taken to be zero though that ultimately won't change the results of the algorithm.
  - $G = (V, E)$ is thus a fully defined graph. The weights form a weighted-adjacency matrix
  $W = (w__{ij})\_{i,j=1,...,N}$. In our case the graph is undirected, meaning connections are independent of direction being traveled between nodes. This means that $w_{ij} = w_{ji}$, the adjacency matrix is symmetric. The degree of a vertex measures how strong the connections are that exist between it and other vertices,
$$
d_i = \\sum_{j=1}^N w_{ij}.
$$
Using the set of $\{d_1,..,d_N\}$ we form a diagonal matrix. This matrix, $D$, is the degree matrix of size $N \times N$. Please note that we use $D$ to represent the distance matrix later in the document but in this section we use it to represent the degree matrix.
\begin{figure}[H]
\centering
    \includegraphics[totalheight=2.5cm]{graph.pdf}
    \caption{Undirected Weighted Graph}
    \label{fig:graph1}
\end{figure}
Using the example in Figure \ref{fig:graph1} where we assume vertex 2 (not shown) has no connections, we would have the following for $W$ and $D$. Please note we use zero-based indexing here.
$$
W = \begin{pmatrix}
0 & 0 & 0 & 3 & 4 & 5 \newline
0 & 0 & 0 & 2 & 0 & 0 \newline
0 & 0 & 0 & 0 & 0 & 0 \newline
3 & 2 & 0 & 0 & 0 & 0 \newline
4 & 0 & 0 & 0 & 0 & 1 \newline
5 & 0 & 0 & 0 & 1 & 0 \newline
\end{pmatrix}
$$
$$ D = \begin{pmatrix} 12 & 0 & 0 & 0 & 0 & 0 \newline
0 & 2 & 0 & 0 & 0 & 0 \newline
0 & 0 & 0 & 0 & 0 & 0 \newline
0 & 0 & 0 & 5 & 0 & 0 \newline
0 & 0 & 0 & 0 & 5 & 0 \newline
0 & 0 & 0 & 0 & 0 & 6 \newline
\end{pmatrix} $$
\newline
The final basic building block one should know is the notion of connected components. If $A \subseteq V$, $A$ is said to be a connected component of $G$ if any two vertices $v_i, v_j \in A$ can be connected such that all intermediate vertices are also in $A$.

### Graph Laplacians
The Laplacian matrix is an extremely useful tool that can be used to find many properties of a graph. It is defined as,
$$
L = D - W.
$$
As noted above one can see that it won't matter what you make the self-to-self weighting for the vertices. Any self weighting will result in the exact same $L$.

_Properties of $L$_

- $L$ is symmetric and positive semi-definite
- All of the row and column sums of $L$ equal 0
- The smallest eigenvalue of $L$ is zero with eigenvector $v = (1, 1, ..., 1)$
- $L$ has $N$ non-negative eigenvalues 0 $=\lambda_1 \leq \lambda_2 \leq ... \leq \lambda_N$
- $\forall  f \in \mathbb{R}^N $

$ f'Lf = \frac{1}{2} \sum_{i,j = 1}^N w_{ij}(f_i - f_j)^2. $

One of the most interesting and useful properties of $L$ is that the the number of connected components in $G$ is equal to the algebraic multiplicity of eigenvalue 0. This is also equal to the dimension of the nullspace of $L$. To show this is true imagine a case where there is only one connected component, the whole graph. If the eigenvector pertaining to the eigenvalue 0 is $f$ then from the properties above we have,

$
0 = f' L f = \sum_{i,j=1}^N w_{ij}(f_i - f_j)^2.
$

Every term in the summation is non negative as the second term is squared and the weights in our graph are non-negative. This means that for the equality to hold every $w_{ij}(f_i - f_j)^2$ must equal zero. This graph is connected so all $w_{ij} > 0 \hspace{0.1cm} \forall \hspace{0.1cm} i \neq j$. This means that $f_i=f_j$, thus $f$ needs to be a vector where every element is equal. After normalizing, $f$ is then $(1, 1, ..., 1)$ or the unit vector. A similar argument holds for a graph with $n$ connected components. The matrices $W$ and thus $L$ will now be block diagonal if we order them according to which connected component they belong to. We call each block $L^i$. Equation \ref{eq} still needs to hold and knowing that each block $L^i$ is a connected component it has $f^i = (1, 1, ..., 1)$ thus $f$ for the entire $L$ is this vector padded with zeros where the other blocks are. There will precisely $n$ of these vectors. For example, if each block was size $2 \times 2$ and there were $n=3$ blocks, meaning $N=6$, the 3 normalized eigenvectors pertaining to the eigenvalue zero would look like, $v_1 = (1, 1, 0, 0, 0, 0)$, $v_2 = (0 ,0, 1, 1, 0, 0)$, and $v_3 = (0, 0, 0, 0, 1, 1)$. In reality we will not observe completely disconnected sub-graphs but some perturbation of $L$. This perturbation \textit{will not} substantially affect the resultant vectors. As show in Sun and Stewart (1990) \cite{perturb} the stability of the eigenvectors is determined by the eigengap betwen $\lambda_k$ and $\lambda_{k+1}$.
In a number of experiments run in Ng et al. (2001) \cite{ng}, this eigengap is large for most any reasonable perturbation. As such they provide a reasonable matrix (when stacked) on which to cluster the rows. We discuss this in detail later in this section.
\end{multicols}

### Algorithm
\begin{algorithm}[H]
\caption{Spectral Clustering}\label{spectralalg}
\begin{algorithmic}[H]
\State \textbf{\textit{Input:}} Weighted adjacency matrix $W \in \mathbb{R}^{N \times N}$, number of clusters $k$
\State Compute degree matrix $D$
\State $L \gets D - W$
\State Compute first $k$ eigenvectors $v_1, ..., v_k$  of $L$ and stack them into $X \in \mathbb{R}^{N \times k}$
\State $y_i \gets$ normalized ith row of $X \quad$   (note: $y_i \in \mathbb{R}^{k}$)
\State Cluster $(y_i)_{i=1,...,n}$ into clusters $C_1, ..., C_k$ using k-means
\State \textbf{\textit{Output:}} Clusters $A_1,...,A_k$ where $A_i = \{j | y_j \in C_i \}$
\end{algorithmic}
\end{algorithm}

\begin{multicols}{2}
\subsection{Intuition and interpretation}
\begin{figure}[H]
    \centering
    \subfloat[Non-convex original data]{{\includegraphics[width=3.5cm]{two_circles.png} }}
    \quad
    \subfloat[$y_1$ and $y_2$ (rows in $X$)]{{\includegraphics[width=3.5cm]{rowsy.png} }}
    \caption{Why k-means works on $y_i$'s}%
    \label{fig:yrowsex}%
\end{figure}

After reading Algorithm \ref{spectralalg}, it may not seem obvious why spectral clustering works at all. Namely, one may wonder why we use k-means after all of the fuss about it performing so poorly for time series. The reason for this seeming inconsistency is that in using the algorithm above we transform into a space that is extremely amenable to being clustered using traditional methods. The choice of k-means is somewhat arbitrary and even simple methods like linear classifiers work well on the rows $y_i$. As can be seen in Figure \ref{fig:yrowsex} from \cite{ng} separation of the $y_i$ vectors in the $k=2$ dimensional space is trivial. One should also be aware that the smallest eigenvalues will not be identically zero but are interpreted as such when small enough, this happens because we are not in the ideal *noiseless* case.
Care should also be taken in choosing $k$, the number of clusters the algorithm will group $y_i$'s into. Some authors have chosen $k$ to be the $k$-th eigenvalue directly prior to a jump in the eigenvalues (when sorted). This method has been controversial as there is little theoretical backing for doing so and is heavily dependent on specific qualities of the data. Zelnik-Manor and Perona (2005) propose a self-tuning process that chooses $k$ from the eigenvectors by maximizing sparseness in the matrix of $k$ eigenvectors. At the most basic level one could use qualitative knowledge, in our case about the geographic or legal aspects of the currency market that would lead to some specific number of clusters.
