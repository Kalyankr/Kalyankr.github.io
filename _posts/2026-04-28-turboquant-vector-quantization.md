---
layout: post
title: "Reimplementing Google's TurboQuant: 4-bit vectors with no training"
subtitle: "A PyTorch and NumPy implementation of training free vector quantization"
tags: [vector quantization, embeddings, pytorch, information retrieval, kv cache]
author: Kalyan Reddy Katla
---

TurboQuant is my implementation of Google's TurboQuant algorithm (Zandieh et al., 2025)
for compressing embeddings and KV caches. The headline is hard to believe at first:
compress vectors to 4 bits with recall close to OPQ, and zero training time.

I built it because I kept hitting the cost of vector quantization in retrieval systems.
PQ and OPQ are great until you have to retrain the codebook every time the data shifts. A
training free method that still matches OPQ sounded too good to ignore, so I reimplemented
the paper to see for myself.

Classic Product Quantization (PQ and OPQ) learns a codebook with k means, which is
expensive and has to be redone whenever the data distribution shifts. TurboQuant skips
training entirely. It applies a random orthogonal rotation (so information spreads
evenly across dimensions), scalar quantizes each coordinate with precomputed Lloyd-Max
centroids, and adds a one bit Quantized Johnson-Lindenstrauss correction to keep inner
product estimates unbiased.

![TurboQuant in four steps: rotate, quantize, correct, pack](/assets/img/posts/turboquant-method.svg)

### Challenges I faced

1. Getting the math right. The rotation, the Lloyd-Max centroids for a Gaussian, and the
   QJL sign correction each had to be exactly right, and a small mistake quietly tanks
   recall instead of throwing an error.
2. Bit packing from 1 to 8 bits efficiently, then making search run on the GPU with an
   optional fp16 path for throughput.
3. Honest benchmarking. I built reproducible scripts against FAISS PQ, OPQ, IVF, and SQ
   on real embeddings (GloVe), because it is easy to fool yourself with a friendly test
   set.

### Results

On GloVe-100d (10,000 vectors, 200 queries, recall@10 against exact inner product):

| Method | Build time | Size | Recall@10 |
| --- | --- | --- | --- |
| TurboQuant 4-bit | 38 ms | 0.52 MB | 0.889 |
| TurboQuant 6-bit | 123 ms | 0.77 MB | 0.973 |
| FAISS-PQ (m=50) | 1,187 ms | 0.50 MB | 0.913 |
| FAISS-OPQ (m=50) | 20,518 ms | 0.50 MB | 0.922 |
| FAISS-PQ (m=100) | 3,288 ms | 1.00 MB | 0.976 |

TurboQuant matches OPQ-level recall at the same memory footprint while building 150 to
500 times faster, with no codebook training and no dataset dependency.

Using it is a few lines:

```python
from tqtorch import TurboQuantIndex

index = TurboQuantIndex(dim=384, bits=4, metric="ip")  # no training step
index.add(embeddings)
scores, ids = index.search(queries, k=10)
```

### What I learned

A clever, data independent transform can match a trained method. Reimplementing a paper
end to end, including a NumPy reference and a production PyTorch package with LangChain
support, taught me the algorithm far better than reading it ever could. It also reminded
me that benchmarking rigor is a feature, not an afterthought.

Code: [github.com/Kalyankr/turboquant](https://github.com/Kalyankr/turboquant)
