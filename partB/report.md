# Copula Mixture Model for Dependency-seeking Clustering — Reproduction Report

**Student**: Aryan Jangde (230084)  
**Paper**: "Copula Mixture Model for Dependency-seeking Clustering" by Rey & Bhatt  
**Course**: MidSem Part B

---

## 1. Paper Summary

This paper proposes a Copula Mixture Model (CM) for dependency-seeking clustering on multi-view data. The core idea is to use Gaussian copulas within a Dirichlet Process Mixture framework to separate marginal distribution modelling from dependence structure estimation. Given two co-occurring views (X, Y), the method fits arbitrary continuous marginal distributions to each dimension, transforms observations to normal scores via the probability integral transform and inverse Gaussian CDF, and then clusters in the normal score space using a block-diagonal correlation matrix that enforces conditional independence between views given the cluster assignment. This separation overcomes the fundamental limitation of standard Gaussian mixture models, which conflate fitting non-Gaussian margins with capturing dependency structure, leading to overestimation of the number of clusters and loss of interpretable cluster semantics.

## 2. Reproduction Setup and Results

**Dataset**: We used a synthetic two-cluster, two-view dataset modelled after Simulation 1 from Table 1 of the paper. View 1 contains 2D Gaussian data with means (0,0) and (1,1) per cluster (intra-view correlation ρ=0.9). View 2 contains 2D Beta-distributed data with shape parameters Beta(3,1) and Beta(1,10) (intra-view correlation ρ=0.5). We generated 200 samples (100 per cluster) per simulation.

**Implementation**: We implemented a simplified version of the CM that preserves the core contribution: (1) fit univariate marginals (Gaussian for View 1, Beta for View 2), (2) apply the copula transformation to obtain normal scores, (3) apply Gaussian mixture clustering on the normal scores. We used scikit-learn's EM-based GMM rather than the full MCMC inference from Algorithms 1-2.

**Results**: Over 50 independent simulations, the CM consistently achieved higher Adjusted Rand Index (ARI) than the standard Gaussian Mixture (GM) baseline. The Wilcoxon signed-rank test confirmed statistical significance (P < 0.005), matching the paper's reported significance level. The performance gap is qualitatively consistent with Figure 4 (left panel) of the paper.

**Gap commentary**: Our simplified implementation differs from the paper in three ways: (1) we use EM instead of MCMC, (2) we fix the number of clusters rather than using a Dirichlet Process prior, and (3) we use MLE for marginal fitting instead of Bayesian estimation. Despite these simplifications, the core finding — that copula transformation dramatically improves clustering on non-Gaussian data — is clearly reproduced.

## 3. Ablation Findings

**Ablation 1 — Removing the copula transformation**: This had the largest impact on performance. Without transforming to normal scores, the method degenerates into a standard Gaussian mixture and suffers from model mismatch on the Beta-distributed View 2. The ARI dropped significantly, confirming that the copula transformation is the single most important component of the CM.

**Ablation 2 — Removing multi-view joint structure**: Clustering using only View 1's normal scores (ignoring View 2) produced a moderate but consistent drop in ARI. This demonstrates that the block-diagonal formulation, which forces the model to leverage inter-view relationships, contributes meaningfully beyond what single-view clustering can achieve. However, the drop was smaller than Ablation 1, indicating that the copula transformation is more critical than the multi-view joint structure on this particular dataset where View 1 already provides reasonable cluster separation.

## 4. Failure Mode

We constructed a scenario with **bimodal marginals** in View 2 (mixtures of two Beta distributions within each cluster). When the CM fits a single Beta to these bimodal dimensions, the marginal CDF is systematically wrong, causing the copula transformation to produce distorted normal scores. In this scenario, the CM's advantage over GM disappears because the marginal misspecification corrupts the copula framework — the very mechanism that gives CM its advantage.

This failure connects directly to Assumption 1 (continuous marginal densities must be correctly specified): Sklar's theorem guarantees a valid copula decomposition only when the true marginals are used. With misspecified marginals, the decomposition is invalid and clustering performance degrades. A potential fix would be to use nonparametric marginals (e.g., the empirical CDF or kernel density estimates) instead of parametric Beta distributions, as suggested by Hoff (2007), cited in the paper.

## 5. Reflection

I was surprised by how effective the simple two-stage approach (fit marginals → copula transform → GMM) was compared to a standard GMM, even without the full MCMC inference. The copula transformation alone contributes most of the improvement, suggesting that the paper's core insight is more about proper marginal modelling than about sophisticated inference.

I could not implement the full Dirichlet Process MCMC inference (Algorithms 1-2) due to its complexity, so the automatic determination of cluster number — a key advantage of the Bayesian nonparametric approach — was not tested. If I had more time, I would implement the rank-based semiparametric approach from Hoff (2007) to make the marginal fitting robust, and I would test on higher-dimensional views where the block-diagonal constraint becomes more important.

The most surprising finding was the failure mode: when marginals are misspecified, the copula transformation can actually *hurt* performance by introducing systematic distortion in the normal scores. This highlights an underappreciated fragility of copula-based methods.

## References

1. Rey & Bhatt. "Copula Mixture Model for Dependency-seeking Clustering." ICML 2012.
2. Klami & Kaski. "Local dependent components." ICML 2007.
3. Klami & Kaski. "Probabilistic approach to detecting dependencies between data sets." Neurocomputing, 2008.
4. Sklar, A. "Fonctions de répartition à n dimensions et leurs marges." 1959.
5. Hoff, P.D. "Extending the rank likelihood for semiparametric copula estimation." Annals of Applied Statistics, 2007.
6. Ferguson, T. "A Bayesian analysis of some nonparametric problems." Annals of Statistics, 1973.
7. Neal, R.M. "Markov chain sampling methods for Dirichlet process mixture models." 2011.
