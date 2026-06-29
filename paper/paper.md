---
title: 'IICESEE-Deploy: Reproducible packaging, containerization, and browser-based execution for ensemble ice-sheet data assimilation and modeling'
# ICESEE Ecosystem: Portable deployment of ensemble data assimilation workflows for ice-sheet modeling
tags:
  - Python
  - ice-sheet modeling
  - data assimilation
  - reproducibility
  - scientific workflows
  - high-performance computing
  - containers
  - coupling MPI-applications
authors:
  - name: Brian Kyanjo
    orcid: 0000-0002-0995-1051
    equal-contrib: true
    affiliation: "1"
  - name: Alexander A. Robel
    orcid: 0000-0003-4520-0105
    equal-contrib: true
    affiliation: "2"
  # - name: Third Author
  #   orcid: 0000-0000-0000-0000
  #   affiliation:  "1, 2"
affiliations:
  - name: Georgia Institute of Technology, United States of America
    index: 1
  # - name: Another Institution, Country
  #   index: 2
date: July 15 2026
bibliography: paper.bib
---

# Summary

Modern ice-sheet modeling workflows are difficult to install, configure, and reproduce across HPC, cloud, and educational environments. The ICESEE software ecosystem addresses this challenge through an integrated distribution strategy based on package management, containerization, and browser-accessible deployment.

ICESEE-Deploy is a meta-repository that pins and documents three complementary deployment pathways for ICESEE and ice-sheet modeling workflows: ICESEE-Containers for rapid container-based execution, ICESEE-Spack for source-based installation, debugging, and extensibility, and CryoLauncher for browser-based model execution and data assimilation workflows.

ICESEE-Deploy is a reproducible software deployment ecosystem for the ICESEE (Ice Sheet State and Parameter Estimator) framework, designed to simplify installation, execution, and extension of ensemble data assimilation and ice-sheet modeling workflows across heterogeneous computing environments. Modern cryosphere workflows often require complex dependency management, high-performance computing (HPC) configurations, and model-specific compilation pipelines, creating barriers to reproducibility and accessibility.

ICESEE-Deploy addresses these challenges by integrating three complementary deployment pathways: ICESEE-Spack for source-based package management and extensibility, ICESEE-Containers for portable and preconfigured runtime environments, and CryoLauncher for browser-based interactive execution and cloud-enabled workflows. Together, these components provide a unified software ecosystem that supports local development, HPC deployment, educational use, and cloud-native execution for scalable ice-sheet modeling and data assimilation.

# Statement of need

ICESEE-Deploy is a reproducible software deployment ecosystem for the ICESEE (Ice Sheet State and Parameter Estimator) framework, designed to simplify installation, execution, and extension of ensemble data assimilation and ice-sheet modeling workflows across heterogeneous computing environments. Modern cryosphere workflows often require complex dependency management, high-performance computing (HPC) configurations, and model-specific compilation pipelines, creating barriers to reproducibility and accessibility.

ICESEE-Deploy addresses these challenges by integrating three complementary deployment pathways: ICESEE-Spack for source-based package management and extensibility, ICESEE-Containers for portable and preconfigured runtime environments, and CryoLauncher for browser-based interactive execution and cloud-enabled workflows. Together, these components provide a unified software ecosystem that supports local development, HPC deployment, educational use, and cloud-native execution for scalable ice-sheet modeling and data assimilation.


# Software architecture

ICESEE-Deploy is structured as a meta-repository containing three interoperable software components.

## ICESEE-Spack

ICESEE-Spack provides source-based reproducible installation of ICESEE and supported ice-sheet models using the \texttt{Spack} package manager [@gamblin2015spack]. This pathway is optimized for developers and advanced users requiring direct source access, debugging capabilities, and model extensibility.

## ICESEE-Containers

ICESEE-Containers provides containerized execution environments using Docker and Singularity/Apptainer. These images bundle all required dependencies into portable runtime environments, enabling fast installation, reproducible execution, and simplified cloud deployment.

## CryoLauncher

CryoLauncher provides a browser-accessible computational interface for launching and monitoring ICESEE workflows. It supports both forward ice-sheet simulations and ensemble data assimilation workflows, allowing users to interact with complex cryosphere models without requiring local software installation.

These three pathways form a layered deployment ecosystem: source-native for extensibility, container-native for portability, and browser-native for accessibility.

## Example workflow

A typical ICESEE-Deploy workflow may involve:

1. Installing ICESEE and its dependencies via ICESEE-Spack for development and customization.
2. Running standardized experiments using ICESEE-Containers for reproducible execution.
3. Launching educational or cloud-based simulations through CryoLauncher for interactive access.

This multi-pathway design enables flexible movement between development, production, and educational workflows while maintaining software reproducibility.

# Impact

ICESEE-Deploy lowers technical barriers to entry for ice-sheet modeling and ensemble data assimilation by providing reproducible and portable deployment strategies. It supports a broad user base including cryosphere scientists, data assimilation researchers, educators, and students.

By unifying package-based installation, containerized execution, and browser-based access into a single software ecosystem, ICESEE-Deploy improves reproducibility, simplifies onboarding, and expands participation in computational cryosphere science.

# Acknowledgements

Development of ICESEE-Deploy has been supported through collaborative efforts in cryosphere modeling, scientific computing, and data assimilation research at Georgia Institute of Technology. The authors acknowledge the broader open-source communities supporting Spack, Docker, Apptainer, ISSM, and Icepack.



 <!-- [@key2024].

# Mathematics

Inline math can be written as $F = ma$ and display math as:

$$E = mc^2 \label{eq:einstein}$$

# Figures

Figures can be included like this:

![Caption for the example figure. \label{fig:example}](figure.png)

And referenced from text using \autoref{fig:example}.

# Acknowledgements

We acknowledge contributions from...

# References -->
