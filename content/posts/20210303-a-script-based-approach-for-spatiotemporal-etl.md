---
title: "A Script-Based Approach for Hierarchical ETL"
date: 2021-03-03T03:52:15-06:00
draft: false
tags: ["development", "project-sustain"]
---

[Project Sustain](http://urban-sustain.org) is an ongoing effort to provide diverse, spatiotemporal analytics to inform urban planning. It is an integral component in a well funded, multi-university and discipline project I'm currently involved in. My contributions center around the distributed storage and analyses of diverse data at scale.

The initial drive has left the project in some level of disarray. Data storage formats and documentation are disjoint and the development process has been widely mis-managed, fortunately not under malice rather a result of inexperience. Therefore, we are exploring solutions for standardizing data processing and subsequent analyses. Throughout this process, we hope to adopt a solution providing:

- Support for Diverse Datasets: A viable solution must cope with diversity in source data formats, dimensionality, resolutions, etc.

- Stable and Reproducible Development: User-facing front-end interfaces require development to be performed off the critical path for stability. Implementing a reproducible loading process promotes a well-defined testing pipeline, where users begin with a local implementation, transition to testing in a development cluster, and finally deploy in a user-facing production environment.

- Low Learning Curve: Being an academic project means contributors are active for only a few years (or a semester in some cases). With this level of worker churn, the learning curve must be minimized.

## ETL Processes and Ensuing Development Pipeline

ETL is an acronym for Extract, Transform, and Load. It describes the foundational framework for data warehousing applications. The crux of ETL tools is to provide hierarchical task definitions which, when executed, process a collection of raw data to populate a well-defined, queryable data store. 

The main advantage of defining an ETL process, as opposed to more elementary solutions, is providing reproducibility in the build system. Foremost, the solution enables an end-to-end solution which constructs a data store from raw data. An ancillary component is the ability to integration a specific development pipeline to adopt new datasets. An elementary workflow example may be (1) develop process on a local machine, (2) perform extensive testing on a distributed development cluster, and (3) deploy to a well-controlled production cluster. This proven pipeline mitigates unforeseen issues to front-facing, production level tooling.

## Python-Driven Task Management

There are many robust, well tested ETL frameworks. We did some exploratory implementation with toy ETL pipelines to get a feel for the various tooling. However, we concluded that between the extensive overhead in the learning curve and typical licensing issues these frameworks were ill-suited for our use-case. Additionally, this is an academic project -- if there ever was a place for innovation (or too often the lack thereof), it's probably here.

We managed to find a number of simple, generalized task manager implementations. [doit](https://pydoit.org/) and [invoke](http://www.pyinvoke.org/) are two python-based solutions which allow users to define tasks, with hierarchical dependencies, which integrate well into a script environment. Specifically, we see leveraging these tools for ETL as a solution facilitation:

- Script-Driven Tasks: While both frameworks are development in python, they interoperate well with shell commands.

- Hierarchical Task Dependencies: Higher-level tasks may define dependencies and the use of checkpointing simplifies processing.

- Reproducible Builds: Subsequent execution of a task will produce identical results. This promotes a well-defined development pipeline enabling the production environment to be restricted for well-tested solutions.

## Project Directory Structure

Our project directory layout lends itself to incremental addition of tasks.

    hamersaw@nightcrawler:~/development/sustain-etl$ tree -L 2
    .
    ├── config.py
    ├── docs
    │   ├── macav2.md
    │   ├── ncproj.md
    │   └── region.md
    ├── impl
    │   └── ncproj
    ├── README.md
    ├── task
    │   ├── all.py
    │   ├── macav2.py
    │   ├── ncproj.py
    │   └── region.py
    └── tasks.py

A brief description of the top-level components is as follows:

- config.py: A python configuration file, used specifically to set environment variables.

- docs/: Generalized documentation folder. Currently, the python task definition file and documentation are singularity related.

- impl/: Directory used to download and compile utility libraries and applications.

- README.md: Standard README file.

- task/: Directory where file defining specific tasks are located

- tasks.py: Top-level, invoke required task definition.

# Intermission

Below are static links to the relevant project files. I worry that linking to the repository will be useless, because this change.

[config.py](/posts/20210303-a-script-based-approach-for-spatiotemporal-etl/config.txt)
[task/all.py](/posts/20210303-a-script-based-approach-for-spatiotemporal-etl/all.txt)
[task/macav2.py](/posts/20210303-a-script-based-approach-for-spatiotemporal-etl/macav2.txt)
[task/ncproj.py](/posts/20210303-a-script-based-approach-for-spatiotemporal-etl/ncproj.txt)
[task/region.py](/posts/20210303-a-script-based-approach-for-spatiotemporal-etl/region.txt)
[tasks.py](/posts/20210303-a-script-based-approach-for-spatiotemporal-etl/tasks.txt)

For the sake of brevity I'm going to forego a tutorial on the invoke application. The project is concerningly well-documented. Take a look if your interested, the utility of generalized task managers is difficult to overstate.

# Task Definitions

Our solution partitions ETL tasks into a variety of categories. While these are mostly semantic, data storage identifiers coincide with the task names which populate them. Therefore, users may easily infer the source of a dataset. Our tasks categories are defined as follows:

- Compile: Download source code repositories and build tooling to assist in ETL process.

- Stage: Load raw data into the data store, these should not include significant pre-processing.

- Build: Compute complex dataset using existing staged data, potentially requiring aid of additional tooling.

- Clean: Remove intermediately cached data produced during the ETL process.

Our preliminary example was conceived to ensure this tooling satisfies our use-case. We show the ability to (1) process shapefile data defining county and state boundaries in the United States (2) compile external utilities in [ncproj](https://github.com/hamersaw/ncproj) which we covered in [another post](/posts/20210211-reprojecting-netcdf-data-using-shapefiles/) and (3) using the county definitions to aggregate netcdf macav2 data using ncproj. Below is a invoke displayed list of the tasks we defined.

    hamersaw@nightcrawler:~/development/sustain-etl$ invoke -l
    Available tasks:

      all.build             Execute all build tasks
      all.clean             Execute all clean tasks
      all.compile           Execute all compile tasks
      all.stage             Execute all stage tasks
      macav2.build          Build macav2 data
      macav2.clean          Delete cached macav2 data
      macav2.index          Compute grid index for macav2 data
      ncproj.compile        Compile the ncproj git repository
      ncproj.update         Update the ncproj git repository
      region.clean          Clean region data
      region.stage          Stage region data
      region.county.clean   Delete cached county data
      region.county.stage   Stage county data
      region.state.clean    Delete cached state data
      region.state.stage    Stage state data

We structured the invoke collection hierarchy to reflect task dependencies. For example, the "all.stage" task will execute the "region.stage" task, which in turn executes both "region.county.stage" and "region.state.stage". This enables users to execute at a variety of granularities. Below are a few examples of executing invoke tasks at varying levels.

    # stage both county and state region datasets
    hamersaw@nightcrawler:~/development/sustain-etl$ invoke region.stage

    # clone ncproj repository and compile project
    hamersaw@nightcrawler:~/development/sustain-etl$ invoke ncproj.compile

    # use ncproj to index and load macav2 data
    hamersaw@nightcrawler:~/development/sustain-etl$ invoke macav2.build
