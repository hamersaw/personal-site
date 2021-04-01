---
title: "Projects"
date: 2021-03-24T11:49:12-07:00
draft: false

disqus: ""
hideDate: true
---

# STIP

Effective analyses of satellite imagery faces several intrinsic challenges stemming from the volume and variety of data. STIP (SpatioTemporal Image Partitioner) spatially partitions input images and coordinates storage and analytics within a DHT, facilitating distributed evaluations with data locality. Image partitioning is performed with geocodes, which support varying levels of precision and spatial reference systems. STIP reconciles differences in image format, pixel resolution, band types, and spatial reference systems among others. Additionally, it rectifies occlusions, which may be attributed to atmospheric phenomena or instrumentation failure, with preclude accurate analysis. \[[Code Repository](https://github.com/hamersaw/stip)\]

# Atlas

Observational dataset formats seldom align with spatiotemporal access patterns, introducing challenges in identification and retrieval of specific spatiotemporal extents. AtlasFS addresses these inefficiencies with two approaches. Blocked data is reordered so observations are spatiotemporally contiguous, facilitating targeted, sequential disk access. Additionally, informed data replication reconciles the competing pulls of dispersion and locality to mitigate analytical data movements. AtlasFS provides an HDFS-compliant interface with extensions for URL-embedded spatiotemporal filtering predicates, providing efficient, interoperable data space reduction operations. \[[Code Repository](https://github.com/hamersaw/NahFS)\]

Effective spatiotemporal analytics must account for intrinsic data characteristics such as voluminous data and unique access patterns. AtlasSpark, an Apache Spark spatiotemporal extension, supports a rich collection of data wrangling functionality over native points, lines, and polygons including inspection, calculation, and transformation operations. AtlasSpark ensures performant analytics by injecting domain-specific rules into Spark's logical and physical plans. It mitigates data movements by scheduling analytics with data locality, aggregates spatiotemporal filtering, and prunes unnecessary abstract syntax tree branches. \[[Code Repository](https://github.com/hamersaw/NahSpark)\]

# Anamnesis

Memory access is several orders of magnitude faster than disk access. However, modern datasets quickly exceed the memory capacity of commodity clusters. Data sketches reduce dataset sizes by maintaining summary statistics and inter-feature relationships. Anamnesis is the first in-memory, sketch-aligned, HDFS-compliant distributed file system. It leverages data sketches to alleviate memory contention and reduce network I/O. Upon request, Anamnesis may generate full resolution data in myriad formats and presents an HDFS-compliant interface, achieving unprecedented compatibility with popular analytics engines. \[[Code Repository](https://github.com/hamersaw/anamnesis)\]

# Proddle

Near real-time detection of Internet outages is a difficult problem, where individual events may be short-lived, localized, or service specific. Proddle is a global service monitoring framework used to detect such outages. It supports script-driven monitor definitions facilitating diverse functionality including incrementally refined monitor precision. Monitors are evaluated periodically on globally diverse vantages providing wide-spread outage coverage. Additionally, Proddle manages the orchestration of monitor data collection and archiving, providing well-defined API's for community retrieval of raw data. Accompanying is a web UI with live monitoring information, including event-driven alerts with the perceived outage cause. \[[Code Repository](https://github.com/CSUNetSec/proddle)\]
