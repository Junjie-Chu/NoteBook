# Unsupervised learning goals
1. Approximate the data distribution p(x)
2. Generate new samples (i.e. generative model 生成模型)
3. Learn useful features

# Differences to supervised learning
1. We aim to model pθ(xn) as opposed to pθ(xn | yn), implying that unsupervised learning is about estimating unconditional distributions, whereas supervised learning is concerned with conditional distributions.
2. We need to work with multivariate probability models, since xn is most often a vector, whereas in supervised learning the output yn is often a scalar. 

Unsupervised learning is more widely applicable compared to supervised learning, since it does not require a human expert to manually label the data.

Unsupervised learning methods needs to somehow find an underlying hidden structure in the data, e.g. interesting patterns, clusters, correlations of causal relationships

# PCA
https://zhuanlan.zhihu.com/p/77151308
