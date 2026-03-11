# Dataset Information

## Dataset Description

This project uses **synthetic (toy) datasets** generated using NumPy and SciPy. No external dataset files are required.

### Simulation 1 — Gaussian-Beta Data (used in Tasks 2.1–2.3, 3.1, 3.2)

Modelled after **Table 1, Simulation 1** from the paper:

- **View 1 (Gaussian)**: 2D multivariate normal per cluster
  - Cluster 1: μ = (0, 0), intra-view correlation ρ = 0.9
  - Cluster 2: μ = (1, 1), intra-view correlation ρ = 0.9
- **View 2 (Beta)**: 2D Beta-distributed per cluster
  - Cluster 1: Beta(3, 1) for both dimensions, intra-view correlation ρ = 0.5
  - Cluster 2: Beta(1, 10) for both dimensions, intra-view correlation ρ = 0.5
- **Samples**: 200 total (100 per cluster)
- **Ground truth labels**: Available (since data is synthetically generated)

### How to Generate

The datasets are generated inline in each notebook using `numpy` and `scipy.stats`. No separate download or data files are needed. See `task_21.ipynb` for the full data generation code.

### Inter-view Dependencies

The two views are coupled through cluster membership — points in the same cluster share both a specific Gaussian pattern (View 1) and a specific Beta pattern (View 2). The dependency between views is entirely captured by the cluster assignment, which is exactly the structure the Copula Mixture Model is designed to discover.
