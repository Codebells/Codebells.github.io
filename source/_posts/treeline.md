---
title: treeline
date: 2022-12-02 10:38:45
categories: 
  - [database]
  - [paper_read]
  - [storage]
tags: [paper_read,database,storage]
category_bar: true
---

# 摘要

LSM Tree将随机写变为顺序写，提高了写性能，但是它必须依赖于压缩以及布隆过滤器来维持读性能。但是NVMe SSD的出现，读写性能的Tradeoff就不需要再考虑了，在并发的情况下，在原地更新的方案也可以提供优异的读写性能，是一个可以替代LSM Tree的方案。

3个点

1. record caching for efficient point operation
2. page grouping for high-performance range scans
3. insert forecasting to reduce the reorganization costs of accommodating new records
