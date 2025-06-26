This ipynb notebook provides a reproduceable implementation of the Hierarchical Risk Parity (HRP) portfolio optimization method, based on the work of Marcos López de Prado and adapted for practical use in Python. The notebook includes steps for data preparation, hierarchical clustering, intra-cluster risk parity allocation, and capital allocation across clusters, ensuring a comprehensive understanding of the HRP methodology.

Chatgpt propmpt used in production, created by NotebookLM:
HRP is a novel portfolio optimization method developed by Marcos López de Prado, designed to address issues like instability, concentration, and out-of-sample underperformance often seen in traditional quadratic optimizers such as Markowitz's Critical Line Algorithm (CLA). PyPortfolioOpt's HRPOpt class implements HRP with permission from Marcos López de Prado.
I. Practical Steps for HRP using PyPortfolioOpt (Re-evaluated)
1.
Import Necessary Modules: To perform HRP, you would typically import pandas for data handling, numpy for numerical operations, and specifically HRPOpt from pypfopt.hierarchical_portfolio. You would also use expected_returns from pypfopt to prepare your data.
2.
Prepare Your Data: HRP requires a DataFrame of historical or simulated asset returns. If you have historical prices, expected_returns.returns_from_prices() can convert them into a DataFrame of historical returns, which is an input for HRPOpt.
3.
While HRPOpt can optionally take a covariance matrix, its primary input is the returns DataFrame.
4.
Instantiate HRPOpt: An instance of the HRPOpt class is created by passing your historical returns DataFrame.
5.
Perform Optimization: The HRPOpt class contains a single optimize() method that calculates the HRP weights. You can specify the linkage_method, with 'single' being a common and default choice. This aligns with the "single" linkage method discussed in the HRP paper for hierarchical clustering.
6.
Retrieve and Clean Weights: After optimization, you can retrieve the raw_weights and use the clean_weights() method, which rounds the weights and clips near-zero values for practical usability.
7.
Evaluate Portfolio Performance: The portfolio_performance() method, available through HRPOpt (which inherits from BaseOptimizer), calculates and can optionally print the expected return, volatility, and Sharpe ratio of the optimized HRP portfolio.
II. Behind the Scenes: HRP's Three Core Stages (as Implemented by  and Detailed in Source)
The HRPOpt.optimize() method encapsulates the three main stages of the Hierarchical Risk Parity algorithm, as described by Marcos López de Prado.
1.
Stage 1: Tree Clustering:
◦
Compute Correlation and Distance Matrices: This stage begins by computing an N-by-N correlation matrix from the input asset returns. A distance measure (d_{i,j} = \sqrt{0.5 * (1 - \rho_{i,j})}) is then applied to the correlation matrix to transform it into a proper metric space (distance matrix D). Subsequently, the Euclidean distance ((\tilde{d})) between any two column-vectors of this distance matrix D is computed, representing a "distance of distances".
◦
Hierarchical Clustering: Assets are clustered hierarchically by iteratively grouping the pair of columns with the minimum (\tilde{d}). The "linkage criterion" defines how the distance between a newly formed cluster and other items is measured; for example, min (nearest point algorithm) can be used. This process is applied recursively, creating a hierarchy that results in a linkage matrix. PyPortfolioOpt leverages scipy.cluster.hierarchy.linkage for this, allowing specification of the linkage_method.
2.
Stage 2: Quasi-Diagonalization:
◦
This stage reorganizes the rows and columns of the covariance matrix (or correlation matrix), such that the largest values lie along the diagonal [62, 72 (Exhibit 6)]. This reordering places similar investments closer together and dissimilar investments farther apart, without requiring a change of basis. The algorithm achieves this by recursively processing the linkage matrix from Stage 1 to produce a sorted list of original item indices. This is implemented by the getQuasiDiag function.
3.
Stage 3: Recursive Bisection:
◦
This final stage assigns portfolio weights. It starts by assigning a unit weight to all assets.
◦
The algorithm then recursively bisects clusters (based on the ordering from Stage 2). For each bisection (L_i), it splits into two subsets, (L_i^{(1)}) and (L_i^{(2)}).
◦
The variance of each sub-cluster (L_i^{(j)}) is computed using an Inverse-Variance Portfolio (IVP) allocation for its constituents. The IVP allocation is optimal when the covariance matrix is diagonal.
◦
A split factor (\alpha_i) is calculated based on the inverse proportion of these sub-cluster variances: (\alpha_i = 1 - \tilde{\sigma}_i^{(1)} / (\tilde{\sigma}_i^{(1)} + \tilde{\sigma}_i^{(2)})).
◦
The weights of the assets in the first sub-cluster are then re-scaled by (\alpha_i), and those in the second sub-cluster are re-scaled by ((1 - \alpha_i)). This recursive process ensures non-negative weights that sum to one. This stage is implemented by the getRecBipart function.
III. Key Advantages of HRP and Why it's Reproducible (Re-evaluated)
•
Robustness and Stability: HRP addresses major concerns of quadratic optimizers, including instability and concentration. Unlike traditional methods, HRP does not require the inversion of the covariance matrix, which is a significant source of instability, particularly when dealing with numerically ill-conditioned, ill-degenerated, or singular covariance matrices. This is crucial because, as "Markowitz's curse" highlights, the more correlated investments are, the greater the need for diversification, yet the more unstable traditional solutions become due to estimation errors. HRP's approach avoids this instability.
•
Superior Out-of-Sample Performance: Monte Carlo experiments demonstrate that HRP consistently delivers lower out-of-sample variance compared to CLA (which aims for minimum variance in-sample) and traditional Inverse-Variance Portfolios (IVP). For instance, HRP has shown a 72.47% lower out-of-sample variance than CLA, potentially improving the out-of-sample Sharpe ratio by about 31.3%. This outperformance becomes even more substantial with larger investment universes, stronger correlation structures, or when rebalancing costs are considered. HRP provides better protection against both common and idiosyncratic shocks.
•
Diversification and Intuitive Structure: HRP produces more diversified portfolios than CLA. It finds a compromise between broad diversification across all investments and diversification across hierarchical clusters. Its hierarchical structure ensures stable and intuitive weight rebalancing, as weights rebalance among peers at various hierarchical levels and are distributed top-down, consistent with how many asset managers build portfolios (e.g., from asset class to sectors to individual securities). CLA allocations, in contrast, can respond erratically to shocks.
•
Computational Efficiency: The HRP algorithm is computationally efficient, converging in deterministic logarithmic time, (T(n) = O(\log_2 n)).
•
Reproducibility: PyPortfolioOpt explicitly states that its HRPOpt class's code is "reproduced with permission from Marcos Lopez de Prado (2016)". The HRP_marcos.pdf source itself includes the Python code snippets for getIVP, getClusterVar, getQuasiDiag, and getRecBipart, which are the fundamental building blocks of the algorithm. This direct translation ensures that anyone using PyPortfolioOpt with the same input data will arrive at the same HRP portfolio, making the process highly reproducible.