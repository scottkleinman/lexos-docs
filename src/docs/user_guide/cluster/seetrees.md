# Hierarchical Agglomerative Clustering with SeeTrees

!!! warning

    `seetrees` is currently in beta mode. Some of the visualisation methods do not return figures that can be stored to variables. Likewise, there are currently no methods to save the plot output. For the moment, it is best called as a script or run in a Jupyter notebook where the figures can be displayed by default when the method is called.

## Overview

The `seetrees` module provides stylometric analysis and visualization tools for document-term-style data. It is especially useful for comparing document profiles, computing distance matrices, visualizing document relationships, and exploring feature importance across clusters.

SeeTrees is adapted from the R ['see' package](https://github.com/perechen/seetrees){target="_blank"} by Artjoms Šeļa.

The `SeeTrees` class accepts frequency tables, distance matrices, and Lexos `DTM` objects (recommended) as input. It can compute stylometric distance matrices using several metrics, visualize relationships using Multidimensional Scaling (MDS) or Principal Components Analysis (PCA), compare documents by z-score profiles, and render dendrograms with feature importance scores.

## Basic Usage

``` python
# Sample method calls
st = SeeTrees(frequencies=frequencies)
st.compute_distances(metric="delta")
st.view_distances(method="MDS")
st.view_scores(target_text="doc_a", top=15)
st.view_tree(k=2)
```

``` python
from lexos.cluster.seetrees import SeeTrees
from lexos.dtm import DTM

# Assuming dtm is already built and contains document labels
st = SeeTrees(dtm=my_dtm)
st.compute_distances(metric="cosine_delta")
st.view_tree(k=3)
```

## Initializing SeeTrees

Although the recommended input for initializing `SeeTrees` is a Lexos `DTM`, `SeeTrees` supports precomputed distance matrices, lists of features names, dicts of features and pandas DataFrames. See the [API documentation](../../api/cluster/seetrees.md) for details on the constructor signature and parameters.

## Methods

### Statistical Concepts Used by SeeTrees

`SeeTrees` uses a few standard statistical concepts in its visualizations and cluster analysis.

- **Z-score**: A z-score measures how many standard deviations a value is from the mean of its feature distribution. In `SeeTrees`, z-scores are computed across documents for each feature, so feature usage can be compared on a standardized scale.
- **Eta**: Eta (`η`) is a measure of association between a categorical grouping (such as cluster membership) and a numeric variable (such as feature frequency). It is related to how much of the variance in the feature is explained by the grouping.
- **Eta-squared (`η²`)**: Eta-squared is the squared value of eta and represents the proportion of total variance in a feature that is explained by cluster assignment. In `SeeTrees.view_tree()`, the top eta-squared features are the terms most strongly associated with the clusters defined by the dendrogram.

### `compute_distances`

This method computes a pairwise distance matrix from the loaded frequency data. It accepts the `metric` parameter with one of the following values:

- `"delta"` — Burrows' Delta using Manhattan distance over z-scores.
- `"eder_delta"` — Eder's Delta using rank-weighted Manhattan distance over z-scores.
- `"cosine_delta"` — Cosine distance computed on z-scored frequencies.
- `"manhattan"` — Standard Manhattan distance on raw frequencies.
- `"cosine"` — Standard cosine distance on raw frequencies.

The method returns a pandas DataFrame containing the pairwise distance matrix, indexed by document labels.

### `view_distances`

This method outputs a visualization of document relationships in two-dimensional space, using the specified projection method.

It accepts the following parameters:

- `method`: The projection method, either `"MDS"` or `"PCA"`
- `metric`: Optional metric name used to compute a distance matrix before projection. If provided, `compute_distances(metric)` is called.
- `random_state`: Optional random seed integer for reproducibility.

!!! note
    - When `method="MDS"`, `distance_table` must be available either through initialization or via the `metric` parameter.
    - When `method="PCA"`, frequency data is z-scored before projection.

### `view_scores`

This method outputs a visualization of the most distinctive features for a single target document. It accepts a `target_text` parameter to specify the document label and a `top` parameter to specify how many of the top absolute z-score features to display (default is 20).

### `compare_scores`

This method displays a visualization comparing two documents by z-score profiles or direct feature differences. It accepts the following parameters:

- `source_text`: Reference document label.
- `target_text`: Document label to compare.
- `top_diff`: Number of top differing features to display.
- `view_type`: View type. Valid values:
    - `"profile"`: Overlay of z-score profiles for both documents.
    - `"difference"`: Bar chart of z-score differences (`target - source`).

### `view_tree`

Renders a dendrogram and reports top eta-squared features that distinguish clusters. It accepts the following parameters:

- `k`: An integer indicating the number of clusters to cut the dendrogram into.
- `right_margin`: An integer indicating the margin for the dendrogram's right-hand side in inches.

## Troubleshooting

- If you see `ValueError: Frequency data is required`, make sure your input includes a valid frequency table or DTM.
- If you see `ValueError: Distance table is required`, run `compute_distances(...)` before calling `view_tree()`.
