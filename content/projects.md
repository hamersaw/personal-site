---
title: "Projects"
date: 2021-03-24T11:49:12-07:00
draft: false

disqus: ""
hideDate: true
---

# Sustain

[Sustain](https://urban-sustain.org)

TODO

# Atlas

Observational dataset reporting seldom aligns with spatiotemporal access patterns. This introduces significant challenges for spatiotemporal analytics, which typically evaluate on a specific extent. AtlasFS addresses these inefficiencies at two levels. Blocked data is reordered so observations are spatiotemporally contiguous, facilitating targeted, sequential disk access during retrieval. Additionally, informed data replication reconciles the competing pulls of dispersion and locality ensuring each spatiotemporal extent is simultaneously distributed and collocated within the cluster to mitigate analytical data movements. AtlasFS provides seamless interoperation with analytic engines by providing an HDFS-compliant interface. Furthermore, it supports URL-embedded spatiotemporal filtering predicates for efficient data space reduction operations. \[[Code Repository](https://github.com/hamersaw/NahFS)\]

Effective spatiotemporal analytics must account for intrinsic data characteristics including voluminous data and access patterns. AtlasSpark provides a language agnostic, interoperable, spatiotemporal extension to ApacheSpark. It supports a rich collection of data wrangling functionality operating over native points, lines, and polygons. These include inspection (ex. intersects, disjoin), calculation (ex. area, distances), and transformation (ex. normalize, envelop) operations. AtlasSpark ensures performat analytics by injecting domain-specific rules into Spark's logical and physical plans. It mitigates data movements by scheduling analytics with data locality, aggregates spatiotemporal filtering (i.e., combine filters into a single, more precise predicate), and prunes unnecessary abstract syntax tree branches (e.x., sequential filtering over disjoint spatial bounds). \[[Code Repository](https://github.com/hamersaw/NahSpark)\]

# Anamnesis

[Anamnesis](https://github.com/hamersaw/anamnesis)

TODO

# Proddle

TODO
