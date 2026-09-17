---
title: 'CryoStack: A modular cyberinfrastructure stack for cryosphere data, models, data assimilation, and heterogeneous computing'
tags:
  - Python
  - cryosphere
  - scientific gateways
  - ice-sheet modeling
  - data assimilation
  - radar data
  - reproducible workflows
  - high-performance computing
  - cloud computing
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
date: 2 September 2026
bibliography: paper.bib
---

# Summary

CryoStack is an open-source cyberinfrastructure stack for assembling
reproducible cryosphere workflows from scientific applications, user
workspaces, data catalogs, experiment records, and heterogeneous execution
resources. It connects ice-sheet modeling, ensemble data assimilation, and
the discovery and reuse of historical radar observations behind one shared
platform. Its current applications are **CryoLauncher** for configuring and
running ice-sheet models; **ICESEE** for ensemble-based state and parameter
estimation [@kyanjo2026icesee]; **LIVIST** (Living Ice Sheet Temperature) for
exploring Antarctic englacial-temperature products inferred from radar and
constrained by boreholes; and **Frozen Legacies** for historical Antarctic
radar observations and derived products.

CryoStack grew from deployment tooling for ICESEE into a wider stack for the
computational cryosphere. Applications retain their domain-specific interfaces
while reusing identity, persistence, execution, and deployment services.
Supported workflows use local resources, institutional computing, or Cloud
execution, currently backed by AWS.

CryoStack is available at <https://cryostack.eas.gatech.edu/> and its source
is maintained at <https://github.com/ICESEE-project/CryoStack>.

# Statement of need

Cryosphere research increasingly combines large observational collections,
interactive interpretation, numerical ice-sheet models, inverse methods,
ensemble simulations, and scalable computing. These components have different
software and resource requirements. Data discovery and quality control benefit
from interactive maps and browser interfaces. Model development may occur on
a workstation. Ensemble assimilation and production simulations commonly
require Message Passing Interface (MPI)-enabled libraries, batch schedulers,
or cloud resources. Historical radar holdings additionally require
dataset-specific ingestion, geolocation, quality-control tools, and
preservation of provenance.

Today these stages are frequently delivered as separate repositories and
manual procedures. A researcher may need to install compiled model
dependencies, translate data formats, write scheduler scripts, transfer
inputs, track job identifiers, retrieve outputs, and remember which
parameters and environment created a result. The work is repeated when the
model, data product, institution, or compute system changes. This integration
burden disproportionately affects students and groups without dedicated
research-software or HPC support.

CryoStack is intended for researchers moving between exploratory and
production workflows, collaborators who do not share the same computing
environment, radar scientists preserving difficult legacy observations, and
instructors who need a consistent entry point for computational examples.
Its architectural goal is to make transitions among applications, data, and
resources explicit, repeatable, and inspectable.

# State of the field

CryoLauncher's supported models draw on established community software: ISSM
(Ice-sheet and Sea-level System Model) provides continental-scale ice-sheet
modeling and inversion [@larour2012issm], and Icepack provides composable
glacier-flow modeling in Python [@shapero2021icepack]. DART and PDAF provide
mature, general-purpose data-assimilation capabilities [@anderson2009dart;
@nerger2013pdaf]. ICESEE adds model-agnostic ensemble Kalman filtering
tailored to ice-sheet applications, multiple filter variants, MPI
parallelism, and couplings to ISSM and Icepack [@kyanjo2026icesee].
CryoStack does not reimplement these scientific algorithms.

These packages do not individually provide the shared access and operations
layer needed across the applications integrated here. CryoStack supplies
identity, data discovery, model configuration, remote execution, experiment
history, and application deployment across heterogeneous environments.
Keeping this layer outside the scientific packages allows applications and
compute backends to evolve independently.

# Software design

CryoStack separates access and operations, scientific applications, execution
backends, and reproducible software environments (Figure 1). Its design treats
the scientific experiment, dataset, and application as related but independent
objects. Identity and experiment persistence live outside the applications;
per-user workspace roots are enforced by explicit containment checks. A
capability registry states which configuration subsets, result packages,
licensing requirements, and execution environments apply to each model or
workflow. Deployable applications are declared in a registry rather than
hard-coded into a single service.

The platform standardizes what an application shares, not how it presents
itself. Scientific configuration is separated from execution, so a given
configuration is portable across execution environments, while
backend-specific resource and readiness checks remain necessary.
CryoLauncher and ICESEE each implement their own submission, status, log,
and termination lifecycle today rather than through one shared execution
adapter; both reuse the same Connector/Relay connectivity and AWS
account-onboarding layer (Figure 1).

![CryoStack's layered architecture. Four scientific applications reuse platform services and shared contracts. Execution support is workflow-specific; the Local, Remote-HPC, and AWS Batch backends do not apply to every application. Connector/Relay also supports private institutional connectivity for Cloud workloads, a path not shown explicitly here.](cryostack_architecture.svg){#fig:architecture}

The persistence layer stores users, sessions, saved configurations,
per-application workspace state, and experiments in SQLite. Each experiment
records an immutable configuration snapshot with application, backend, job
identifiers, output paths, and a status-event timeline. Canonical examples
remain read-only and are copied into user-owned working copies before editing
or execution. A user can return to a saved workspace, reuse a named
configuration, and relate scheduler state to the configuration that produced
it. These records provide a basis for fuller provenance, not a complete
archival package of inputs, environments, transformations, and outputs.

# Scientific applications

**CryoLauncher** configures and runs ISSM and Icepack. Basic, Advanced, and
Auto-config · Beta operate over the same underlying run configuration:
Basic exposes a curated, solver-aware parameter subset; Advanced exposes
the full working copy; Auto-config proposes changes from natural language.
CryoLauncher's execution paths are Remote and Cloud, currently AWS Batch;
it has no local execution mode for standalone ISSM or Icepack runs. ISSM's
container workflow uses MATLAB and requires a license. Icepack runs
Firedrake-based notebooks or scripts without that dependency.

**ICESEE** supplies ensemble state and parameter estimation
[@kyanjo2026icesee]; CryoStack does not reimplement its filtering algorithms.
ICESEE organizes execution as Local, Remote, or Cloud, with Auto-config ·
Beta available alongside it. Local execution runs a selected workflow as a
Python subprocess on the application host; Remote and Cloud run through
ICESEE's own submission
and result-handling implementation, sharing the platform's Direct-SSH/
Connector connectivity and AWS account-onboarding layer with CryoLauncher
rather than a common execution or result pipeline. ICESEE can wrap ISSM,
Icepack, or a synthetic model such as
Lorenz-96 as its forecast model. Dependencies follow that choice: an
ISSM-based workflow requires MATLAB, whereas Icepack and Lorenz-96 do not.
ICESEE's assimilation diagnostics do not yet use CryoStack's shared result
schema.

**Frozen Legacies** extends CryoStack to the preservation and reuse of
historical Antarctic radar observations. Dataset manifests and ingestion
adapters produce a catalog with geolocated observation points, flight
geometries, metadata, and links to products. The initial adapter targets
LYRA-derived records from historical airborne radar surveys. Integrated
interpretation tools support radar picking and tracing, producing geolocated
picks, travel-time or power measurements, ice-thickness estimates, and
quality-control records. CryoStack makes the catalog and documentation
discoverable and packages the tools; it does not yet execute every desktop
workflow as a browser-native, provenance-captured service. A new historical
collection needs a dataset manifest and ingestion adapter rather than a
model execution adapter.

**LIVIST** is an interactive application for Antarctic englacial-temperature
products inferred from radar observations and constrained by borehole
measurements. Its own frontend and documentation are built and served
through the CryoStack deployment registry, sharing routing and application
context. LIVIST and Frozen Legacies retain their specialized interfaces;
Remote, Cloud, and Auto-config · Beta do not apply to them.

# Auto-config · Beta

Auto-config · Beta produces deterministic configuration proposals from
natural-language requests using existing model, example, resource, and
capability metadata. The manual controls remain authoritative. A compact
change preview distinguishes values from the request, suggested adjustments,
and retained settings. Omitted valid settings are preserved; ambiguous or
unsupported choices are not silently resolved.

The same interface supports configuration diagnosis, deterministic repair
proposals, and read-only explanations. Refinements use the current controls
rather than conversation history. Applying a proposal updates configuration
only and refuses stale or altered proposals. Existing validation and
execution controls remain authoritative; Auto-config does not submit jobs,
provision infrastructure, or configure licenses.

# Results, packaging, and visualization

A completed standalone ISSM or Icepack run is exported to a transport-neutral,
versioned result package containing metadata, mesh geometry, field arrays,
and available figures. If a tutorial produces no plottable field, the package
records what could and could not be exported without turning successful
scientific execution into a failed run.

A reader for each schema loads the package without the model's runtime:
MATLAB for ISSM or Firedrake for Icepack. Both readers satisfy a shared
result-package protocol, and a shared visualization layer renders fields
and time series from the neutral package. A third model gains a results
view by providing a conforming exporter and reader. The Icepack exporter
records its conversion of Firedrake fields to a first-order nodal
representation for display; its behavior on live Firedrake output still
needs confirmation.

# Remote execution and the Connector/Relay

CryoStack's Remote path lets a user bring computing resources they already
have access to, such as an institutional cluster or workstation, into a
CryoLauncher or ICESEE run. Each application's own scheduler-aware launcher
manages submission, status, logs, and result retrieval; CryoLauncher and
ICESEE implement this separately today, sharing the Direct/Connector access
pattern below rather than one launcher or one result representation.

Two access modes reach the target resource. **Direct** invokes SSH and file
transfer from the application host when network policy and reachability
allow it. **Connector** serves resources reachable only from the user's
workstation, VPN, or an approved edge environment. A packaged Connector
exchanges commands and files through an authenticated relay without requiring
an inbound connection from CryoStack to the institution (Figure 2). The
target institution retains its accounts, authentication, scheduler policy,
and allocation enforcement.

Connector/Relay is shared infrastructure for Remote access and Cloud
workloads that need a private institutional service. The concrete case
implemented today is ISSM submitted through CryoLauncher to AWS Batch:
the running container uses the same Connector/Relay infrastructure to reach
an institutional MATLAB network-license service without exposing that
service directly to the cloud. The scientist supplies the ordinary
institutional license endpoint; CryoStack handles the cloud-side license
configuration and connectivity. Cloud execution that does not need private
institutional connectivity does not require Connector.

![CryoStack's four applications and their shared connectivity. CryoLauncher and ICESEE each implement their own Remote and Cloud execution; both reach an institutional or HPC resource by direct SSH or through the shared Connector/Relay. The same Connector/Relay can also carry a Cloud workload's connection to a private institutional service (dashed) -- the demonstrated case is CryoLauncher's ISSM workflow on AWS reaching the Georgia Tech MATLAB license. Frozen Legacies and LIVIST are equally first-class CryoStack applications with their own interfaces, not organized around Local/Remote/Cloud.](cryostack_hpc_bridge.svg){#fig:bridge}

# Cloud execution on AWS

Cloud execution is implemented through provider-independent driver contracts;
AWS is the currently supported provider. Moving a supported workflow between
Remote and Cloud reuses its scientific configuration, although resource
settings and validation differ. A personal AWS account connects through a
role-based onboarding flow. AWS Batch on Fargate is the default compute
mode; EC2 is an opt-in alternative. GPU and multi-node infrastructure options
do not enable qualified scientific execution.

CryoLauncher's tested Icepack workflows have completed end-to-end execution
on Fargate and EC2 On-Demand with a single CPU node. ICESEE's Lorenz-96
example has completed Fargate execution under its single-process contract.
These results do not qualify other examples, EC2 Spot capacity, custom
networking, GPU, or multi-node execution. CryoLauncher's ISSM workflow has
completed end-to-end execution on both AWS Batch Fargate and EC2 On-Demand,
including MPI-parallel solver execution, postprocessing into CryoStack's
shared result package, retrieval, and visualization. These runs accessed
the configured Georgia Tech institutional MATLAB license through the same
Connector/Relay infrastructure used for Remote access. This validates one
institutional Cloud/Connector configuration rather than arbitrary
license-server arrangements. ICESEE's Cloud interface exposes the same
MATLAB license field as CryoLauncher's, for interface consistency, but that
connectivity is not currently wired into ICESEE's own Cloud submission
path: an ISSM-coupled ICESEE Cloud workflow is not operational today,
independently of its example/process restriction to Lorenz-96 at a single
process.

Two environment strategies complement these execution paths. ICESEE-Spack
uses Spack [@gamblin2015spack] to resolve source builds against site
compilers, MPI implementations, and system libraries. ICESEE-Containers
uses Docker and Apptainer for preconfigured execution; Apptainer follows
the scientific-container approach described by @kurtzer2017singularity.

# Availability, verification, and limitations

Automated tests cover the gateway, authentication, model adapters,
Connector, and cloud modules. A separate offline acceptance command checks
Auto-config safety properties, capability-registry consistency, and
workspace isolation. These tests exercise software behavior without live
institutional access; they do not establish scientific correctness or
replace the workflow-specific live checkpoints above.

Current deployment limitations include process-local Connector session
state, incomplete resource-policy and audit enforcement, and further
qualification needed for shared AWS accounts and recovery from operational
failures. SQLite supports the current single-node deployment; broader
scaling requires shared persistence and durable task/session storage.
Packaged Connector binaries must match the relay protocol, and reference
container images need publication under a project registry namespace.
Model-specific result support and archival provenance remain incomplete.

The codebase is
distributed under the MIT License; a subset of source files inherited from
earlier scaffolding still carries SPDX BSD-3-Clause identifiers, which
remains an open reconciliation task before a formal archival release.

# Research impact statement

CryoStack's evidence to date is functional and architectural rather than
adoption-based. It integrates ISSM and Icepack modeling, ICESEE's ensemble
data assimilation [@kyanjo2026icesee], a historical radar catalog with
geolocation and interpretation tools, and a deployed radar/borehole
temperature application. Execution and experiment tracking, implemented
separately by CryoLauncher and ICESEE over shared connectivity and
onboarding infrastructure, support the two modeling applications, while
dataset adapters and application deployment support the observational
applications.

For a research group, supported model workflows retain their configuration
and experiment history across available computing resources rather than
rebuilding operational procedures per backend. A researcher can extend
CryoStack with an application registration, model adapter and result reader,
or dataset manifest and ingestion adapter, according to the scientific task.
The live execution checkpoints and automated tests support these integration
claims; external adoption and reductions in researcher effort have not yet
been established.

# Acknowledgements

This work was supported in part by U.S. National Science Foundation CAREER
award 2235920. The authors thank Renette Jones-Ivey from the University at
Buffalo for help with the initial Jupyter Book backend, and Troy Hilley from
Research Computing Services, and also thank the Partnership for an Advanced
Computing Environment (PACE) for providing computing resources.

# References
