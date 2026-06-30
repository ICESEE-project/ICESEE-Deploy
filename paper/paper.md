---
title: 'ICESEE-Deploy: Reproducible packaging, containerization, and browser-based execution for ensemble ice-sheet data assimilation and modeling'
tags:
  - Python
  - ice-sheet modeling
  - data assimilation
  - reproducibility
  - scientific workflows
  - high-performance computing
  - containers
authors:
  - name: Brian Kyanjo
    orcid: 0000-0002-0995-1051
    affiliation: "1"
    corresponding: true
  - name: Alexander A. Robel
    orcid: 0000-0003-4520-0105
    affiliation: "1"
affiliations:
  - name: School of Earth and Atmospheric Sciences, Georgia Institute of Technology, Atlanta, GA, USA
    index: 1
date: 15 July 2026
bibliography: paper.bib
---

# Summary

ICESEE-Deploy is a reproducible software deployment ecosystem for the ICESEE
(Ice Sheet State and Parameter Estimator) framework [@kyanjo2024icesee],
designed to simplify installation, execution, and extension of ensemble data
assimilation and ice-sheet modeling workflows across heterogeneous computing
environments. Modern cryosphere workflows require complex dependency management,
high-performance computing (HPC) configurations, and model-specific compilation
pipelines, creating significant barriers to reproducibility and accessibility.

ICESEE-Deploy addresses these challenges by integrating three complementary
deployment pathways: ICESEE-Spack for source-based package management and
extensibility on HPC systems [@gamblin2015spack], ICESEE-Containers for
portable and preconfigured runtime environments [@kurtzer2017singularity], and
CryoLauncher for browser-based interactive execution and cloud-enabled workflows
[@iceseeghub2026]. Together, these components provide a unified software
ecosystem that supports local development, HPC deployment, educational use,
and cloud-native execution for scalable ice-sheet modeling and data assimilation.

# Statement of Need

Ensemble-based data assimilation (DA) is now standard practice in atmospheric
and ocean modeling, yet its adoption in ice-sheet science remains limited. A
key obstacle is software accessibility: existing community frameworks such as
DART [@anderson2009dart] and PDAF [@nerger2013pdaf] support ensemble Kalman
filtering for many geophysical models but require substantial configuration
effort and are not purpose-built for ice-sheet geometries or uncertain basal
parameters. Model-specific inversion capabilities in ISSM [@larour2012issm]
are tightly coupled to that codebase, making cross-model DA studies difficult.
ICESEE was designed to fill this gap with a model-agnostic DA engine —
supporting EnSRF, EnTKF, and DEnKF variants — that couples to multiple
ice-sheet back-ends including ISSM, Icepack, and simple flowline models
[@kyanjo2024icesee].

Deploying ICESEE across the heterogeneous compute environments used in
cryosphere research — institutional HPC clusters, cloud services, and local
workstations — introduces a secondary barrier: installation failures, dependency
conflicts, and the absence of portable environments are among the most commonly
cited obstacles to reproducibility in computational science. ICESEE-Deploy
addresses this directly, offering three pathways that together cover the full
spectrum from laptop-scale tutorial use to large-ensemble production runs on
multi-thousand-core clusters. The CryoLauncher component additionally enables
access for collaborators without HPC allocations, for classroom instruction, and
for rapid prototyping through a standard browser interface, with no local
software installation required.

# Software Architecture

ICESEE-Deploy is structured as a meta-repository with three interoperable
components pinned as git submodules. The overall architecture separates
concerns cleanly: ICESEE-Spack and ICESEE-Containers handle software
environment reproducibility, while CryoLauncher handles the orchestration and
interaction layer that routes user actions to compute resources.

## ICESEE-Spack

ICESEE-Spack provides source-based reproducible installation of ICESEE and
supported ice-sheet models using the Spack package manager [@gamblin2015spack].
Spack recipes pin exact package versions and resolve dependency graphs against
site-specific compilers and MPI implementations, enabling reproducible builds
on institutional HPC systems (e.g., Georgia Tech PACE, University at Buffalo
cluster) where Docker is unavailable. This pathway is intended for developers
requiring direct source access, debugging, and model extensibility.

## ICESEE-Containers

ICESEE-Containers provides prebuilt containerized execution environments using
Docker and Singularity/Apptainer [@kurtzer2017singularity]. Container images
bundle ICESEE and all dependencies into portable runtime environments, enabling
rapid deployment without dependency resolution. Docker images target cloud and
local use; Singularity/Apptainer images are generated from the same build
recipes for HPC environments where Docker is unavailable. This pathway is
intended for reproducible production runs and for users without HPC
allocations.

## CryoLauncher

CryoLauncher [@iceseeghub2026] is a browser-accessible orchestration and
interaction platform for ice-sheet modeling and data assimilation workflows,
served publicly at [cryolauncher.com](https://www.cryolauncher.com). Its
architecture is a lightweight Python web service providing two primary GUI
workflows through Jupyter Book and Voilà-based interfaces: the **ICESEE DA
GUI** for coupled ensemble data assimilation, and the **Ice-Sheet Modeling
GUI** for model-only forward simulations.

### Server Architecture

CryoLauncher is implemented as a single Python-based web service with the
following components:

- **Jupyter Book frontend** — provides documentation, workflow entry points,
  and embedded Voilà applications served through a unified public endpoint
  via a reverse-proxy layer.
- **FastAPI relay server** — manages user sessions, connector registration,
  and command routing between the browser frontend and remote compute
  resources.
- **WebSocket services** — maintain persistent bidirectional connections
  between browser sessions and remote execution environments, enabling
  real-time job monitoring, log streaming, and interactive result preview.

The server itself does not execute heavy compute jobs locally. Its role is
orchestration: it receives job configurations from the browser GUI, routes
them to the appropriate compute back-end, monitors execution, and stages
output data back for lightweight visualization and download.

### HPC Coupling and Security Model

CryoLauncher targets a public-facing interface: users connect their own HPC
systems or cloud resources, so access control is handled at the remote system
rather than the web server itself. The platform has been demonstrated with both
Georgia Tech PACE (Phoenix) and the University at Buffalo cluster using
outbound SSH-based job submission and monitoring via SLURM. For cloud
workflows (e.g., AWS), no VPN is required; for on-premises HPC such as PACE,
access inherits the user's existing VPN and SSH configuration.

Credential security is a first-class design concern:

- SSH keys remain entirely under the user's control and never leave the local
  machine.
- The platform itself never stores or handles institutional passwords.
- The public web service never directly initiates inbound connections into
  protected HPC environments.
- Each user session is isolated at the application level, with SSH connections
  established per session.

A second deployment model uses a lightweight local connector application
running on the user's workstation. The connector establishes an outbound
authenticated WebSocket connection to the CryoLauncher relay service, and all
SSH, rsync, SLURM submission, monitoring, and data staging operations are
executed locally on the user's machine using their own VPN and SSH environment.
This model further decouples the public server from protected cluster networks.

### ICESEE DA Workflow in CryoLauncher

Both GUIs share the same orchestration back-end. A typical CryoLauncher
session proceeds as follows:

1. The user opens the Run Center at `cryolauncher.com/icesee_jupyter_notebooks/run_center.html`
   and selects either the ICESEE DA or Modeling GUI.
2. The GUI presents configuration panels for the target model (Lorenz-96,
   flowline, ISSM, Icepack), ensemble size, DA algorithm (EnSRF/EnTKF/DEnKF),
   observation settings, and compute target (local, remote HPC, or cloud).
3. The FastAPI relay routes the job configuration to the selected compute
   back-end via outbound SSH, submits via SLURM, and streams job status and
   logs back to the browser in real time.
4. On completion, output data is staged back to the server for lightweight
   visualization and download through the browser interface.

Lightweight tutorials — including a self-contained Lorenz-96 DA demo — run
entirely in the browser without external compute. Larger experiments on ISSM or
Icepack are routed to external clusters through the same interface.

# Impact

ICESEE-Deploy lowers the barriers to entry for ice-sheet modeling and ensemble
data assimilation by providing reproducible and portable deployment strategies
across the full range of compute environments used in cryosphere research. It
supports a broad user base including cryosphere scientists, data assimilation
researchers, educators, and students, whether they are running production
large-ensemble experiments on institutional clusters, exploring tutorials in a
browser, or developing new model couplings in a containerized environment.

By unifying source-based installation, containerized execution, and
browser-based access into a single software ecosystem, ICESEE-Deploy improves
reproducibility, simplifies onboarding, and expands participation in
computational cryosphere science.

# Acknowledgements

Development of ICESEE-Deploy has been supported through NSF CAREER award
(award number to be added) to A. Robel and through collaborative efforts in
cryosphere modeling, scientific computing, and data assimilation research at
Georgia Institute of Technology. The authors thank Renette Jones-Ivey and
Justin Simle at Georgia Tech PACE for infrastructure support, and acknowledge
the broader open-source communities supporting Spack, Docker, Apptainer, ISSM,
and Icepack.

# References