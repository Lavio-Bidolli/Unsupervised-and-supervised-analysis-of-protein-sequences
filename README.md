# Unsupervised and Supervised Analysis of Protein Sequences

> One-hot encoding · PCA · t-SNE · K-Medoids clustering · Random Forest classification · artificial sequence generation — all from raw FASTA data.

---

## Table of Contents

1. [Overview](#overview)
2. [Pipeline](#pipeline)
3. [1 · Data & Encoding](#1--data--encoding)
4. [2 · Dimensionality Reduction (PCA)](#2--dimensionality-reduction-pca)
5. [3 · Unsupervised Clustering](#3--unsupervised-clustering)
6. [4 · Supervised Classification](#4--supervised-classification)
7. [5 · Artificial Sequence Generation](#5--artificial-sequence-generation)
8. [Requirements](#requirements)
9. [Usage](#usage)
10. [Repository Structure](#repository-structure)

---

## Overview

This project implements a full ML pipeline on protein sequences stored in **FASTA format**.
Starting from raw amino-acid strings, each sequence is numerically encoded and then explored
through both unsupervised (clustering) and supervised (classification) methods.
A generative module is also included to produce novel artificial sequences that statistically
resemble the training distribution.

---

## Pipeline

```
FASTA file
    │
    ▼
One-Hot Encoding (OHE)          ← 20 amino acids × sequence length
    │
    ▼
PCA  ──► t-SNE                  ← dimensionality reduction
    │
    ├──► K-Medoids clustering   ← unsupervised (elbow method for k selection)
    │
    ├──► Random Forest          ← supervised functionality prediction
    │
    └──► Sequence generation    ← sampling from learned distribution
```

---

## 1 · Data & Encoding

Input: protein sequences in standard **FASTA** format.  
Each sequence is encoded with **One-Hot Encoding (OHE)**: every position in the sequence
becomes a binary vector of length 20 (one dimension per canonical amino acid), producing a
sparse but information-preserving matrix representation.

<!-- INSERT: amino acid frequency distribution plot or sequence length histogram -->
<!-- Example: ![Sequence length distribution](plots/your_length_hist.png) -->

---

## 2 · Dimensionality Reduction (PCA)

PCA is applied to the OHE matrix to reduce the high-dimensional encoding to a compact
set of principal components. Explained variance is analysed to select a suitable number
of components before clustering.

### Explained variance

| First PCs | Full spectrum | Cumulative |
|-----------|--------------|------------|
| ![PCA explained variance](plots/pca_explained_variance.jpg) | ![Full explained variance](plots/pca_full_explained_variance.jpg) | ![Cumulative explained variance](plots/pca_full_cumulative_explained_variance.jpg) |

### PC projections (2D)

| PC1 vs PC2 | PC2 vs PC3 | PC3 vs PC4 |
|-----------|-----------|-----------|
| ![PC1 vs PC2](plots/pc1_vs_pc2.png) | ![PC2 vs PC3](plots/pc2_vs_pc3.png) | ![PC3 vs PC4](plots/pc3_vs_pc4.png) |

| PC3 vs PC7 | PC3 vs PC8 |
|-----------|-----------|
| ![PC3 vs PC7](plots/pc3_vs_pc7.png) | ![PC3 vs PC8](plots/pc3_vs_pc8.png) |

### PC loadings

| PC1 – PC2 loadings | PC3 – PC8 loadings |
|--------------------|--------------------|
| ![Loadings PC1–PC2](plots/loadings_pc1_pc2.png) | ![Loadings PC3–PC8](plots/loadings_pc3_pc8.png) |

> An interactive 3-D PCA plot is available at [`pca_3d_interactive.html`](pca_3d_interactive.html).

---

## 3 · Unsupervised Clustering

**K-Medoids** is used because, unlike K-Means, it is robust to outliers and works directly
with distance matrices (useful when sequences have varying lengths or non-Euclidean
dissimilarities).

### Choosing *k* — elbow method

| OHE space | PCA (50 PCs) | All PCs |
|-----------|-------------|---------|
| ![Elbow curve](plots/elbow_curve.png) | ![Elbow 50 PCs](plots/elbow_curve_50pc.png) | ![Elbow full PCs](plots/elbow_curve_full_pc.png) |

| Elbow (test set, OHE) | Elbow (test set, combined) |
|-----------------------|---------------------------|
| ![Elbow kmedoids test](plots/clustering_test_kmedoids_score_elbow.png) | ![Elbow kmedoids combined test](plots/clustering_test_kmedoids_combined_score_elbow.png) |

### Clustering results — PCA & t-SNE projections

#### Standard encoding

| PCA | t-SNE | Contingency |
|-----|-------|-------------|
| ![Clustering PCA](plots/clustering_kmedoids_pca.png) | ![Clustering t-SNE](plots/clustering_kmedoids_tsne.png) | ![Contingency](plots/clustering_kmedoids_contingency.png) |

#### Test-set exploration (first PCs)

| PCA | t-SNE | Contingency |
|-----|-------|-------------|
| ![Test PCA](plots/clustering_test_firstpcs.png) | ![Test t-SNE](plots/clustering_test_tsne_firstpcs.png) | ![Test contingency first PCs](plots/clustering_test_contingency_firstpcs.png) |

| Full test PCA | Full test t-SNE | Full test contingency |
|---------------|----------------|----------------------|
| ![Test full PCA](plots/clustering_test.png) | ![Test full t-SNE](plots/clustering_test_tsne.png) | ![Test contingency](plots/clustering_test_contingency.png) |

#### Combined features — k = 10 and k = 28

| k | PCA | t-SNE | Contingency |
|---|-----|-------|-------------|
| 10 | ![Combined PCA k10](plots/clustering_kmedoids_combined_pca_10.png) | ![Combined t-SNE k10](plots/clustering_kmedoids_combined_tsne_10.png) | ![Combined contingency k10](plots/clustering_kmedoids_combined_contingency_10.png) |
| 28 | ![Combined PCA k28](plots/clustering_kmedoids_combined_pca_28.png) | ![Combined t-SNE k28](plots/clustering_kmedoids_combined_tsne_28.png) | ![Combined contingency k28](plots/clustering_kmedoids_combined_contingency_28.png) |

---

## 4 · Supervised Classification

A **Random Forest** classifier is trained to predict protein **functionality** (class label)
from the encoded sequences. Feature importance is evaluated both with the standard
impurity-based criterion and with **permutation importance** (more robust to high-cardinality
features).

### Confusion matrices

| Training set | Test set |
|-------------|---------|
| ![Confusion matrix – training](plots/confusion_matrix_training_set.png) | ![Confusion matrix – test](plots/confusion_matrix_test.png) |

### Feature importance

| Impurity-based | Permutation-based |
|---------------|------------------|
| ![Feature importance](plots/top_predictive_features_random_forest.png) | ![Permutation importance](plots/top_predictive_features_random_forest_premutations.png) |

<!-- INSERT: learning curve or ROC/AUC curve if generated -->
<!-- Example: ![ROC curve](plots/roc_curve.png) -->

---

## 5 · Artificial Sequence Generation

Artificial protein sequences are generated using a deep generative approach (such as a Variational Autoencoder, VAE) capable of learning the continuous latent representation of the training dataset. The model optimizes a joint loss function combining reconstruction fidelity and a regularization term (KL Divergence) to ensure a smooth latent space.

The quality and biological plausibility of the generated sequences are then evaluated by projecting them back into the low-dimensional spaces (PCA, t-SNE) and analyzing their clustering behavior compared to natural sequences.

### Model Training & Convergence

The training progress demonstrates a stable optimization trajectory: the reconstruction loss decreases steadily while the KL divergence ($D_{KL}$) stabilizes, preventing latent space collapse and ensuring good generative capabilities.

| Loss vs Epochs |
|:---:|
| ![Training Loss](plots/gen_loss_vs_epochs.png) |

### Latent Space Mapping (Natural vs. Sampled)

To ensure the model has captured the underlying biological distribution, the sampled sequences are projected onto the PCA space. The generated data successfully covers the topology of the natural sequence space and accurately reflects the distribution of functional and non-functional classes.

| PCA Comparison (Natural vs. Sampled) | Sampled PCA with Predicted Labels |
|--------------------------------------|-----------------------------------|
| ![PCA Comparison](plots/gen_smp_pc1_vs_pc2) | ![Sampled PCA Labels](plots/gen_2-Components_PCA_Projection_of_sampled_sequences) |

### Topology and Clustering of Generated Sequences

A t-SNE analysis shows how natural (`nat`), artificial (`art`), and sampled (`smp`) sequences smoothly intertwine within the discovered clusters, indicating that the generated sequences are not just copies, but realistic interpolations. The clustering structure of the generated space is validated using the elbow method and silhouette score analysis.

| t-SNE with Cluster & Origin Labels | Elbow & Silhouette Analysis |
|------------------------------------|----------------------------|
| ![t-SNE Clusters](plots/gen_t-SNE_2_components_with_cluster_labels) | ![Sampled Elbow](plots/gen_smp_clustering_kmedoids_score_elbow.png) |

## Requirements

```
python >= 3.11
numpy
pandas
scikit-learn
scikit-learn-extra   # for KMedoids
scipy
matplotlib
seaborn
plotly               # for pca_3d_interactive.html
biopython            # for FASTA parsing
```

Install all dependencies:

```bash
pip install numpy pandas scikit-learn scikit-learn-extra scipy matplotlib seaborn plotly biopython
```

---

## Usage

```bash
# Clone the repository
git clone https://github.com/Lavio-Bidolli/Unsupervised-and-supervised-analysis-of-protein-sequences.git
cd Unsupervised-and-supervised-analysis-of-protein-sequences

# Open the notebook
jupyter notebook func_prot.ipynb
```

The notebook is self-contained and runs end-to-end:
1. Load FASTA file
2. One-hot encode sequences
3. Run PCA and t-SNE
4. K-Medoids clustering with elbow selection
5. Random Forest training and evaluation
6. Artificial sequence generation

---

## Repository Structure

```
.
├── func_prot.ipynb                         # Main analysis notebook
├── configurations/
├── plots/                       # Config files (hyperparameters, paths)
│
├── pca_explained_variance.jpg              # PCA variance (first PCs)
├── pca_full_explained_variance.jpg         # PCA variance (full spectrum)
├── pca_full_cumulative_explained_variance.jpg
├── pc1_vs_pc2.png  ...  pc3_vs_pc8.png    # 2-D PC scatter plots
├── loadings_pc1_pc2.png
├── loadings_pc3_pc8.png
├── pca_3d_interactive.html                 # Interactive 3-D PCA (Plotly)
│
├── elbow_curve.png  ...                    # Elbow curves for k selection
├── clustering_*.png                        # K-Medoids results (PCA / t-SNE / contingency)
│
├── confusion_matrix_*.png                  # RF confusion matrices
└── top_predictive_features_*.png          # RF feature importances
│   ├── gen_loss_vs_epochs.png                  # Generative model training loss
│   ├── gen_smp_pc1_vs_pc2.jpg                  # PCA comparison of natural vs sampled data
│   ├── gen_2-Components_PCA_Projection_...     # Sampled PCA with predicted functionality
│   ├── gen_t-SNE_2_components_with_...         # t-SNE projection showing origins (nat/art/smp)
│   └── gen_smp_clustering_kmedoids_...         # Elbow and Silhouette curves for sampled data
```

---

*Project developed as part of a course in computational biophysics / bioinformatics.*
