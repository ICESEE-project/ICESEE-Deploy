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
  - name: Angelo S. Tarzona
    orcid: 0000-0002-3464-7537
    affiliation: "1"
  - name: Eliza J. Dawson
    orcid: 0000-0002-2662-3201
    affiliation: "1"
  - name: Winnie Chu
    orcid: 0000-0002-8107-7450
    affiliation: "1"
affiliations:
  - name: School of Earth and Atmospheric Sciences, Georgia Institute of Technology, Atlanta, GA, USA
    index: 1
date: 19 October 2026
bibliography: paper.bib
---

# Summary

CryoStack is an open-source cyberinfrastructure stack for assembling
reproducible cryosphere workflows from scientific applications, user
workspaces, data catalogs, simulation configurations, and heterogeneous execution resources. It connects ice-sheet modeling, ensemble data assimilation, and radar observations behind one shared platform.

CryoStack grew from deployment tooling for an ice sheet data assimilation library into a wider stack for computational cryospheric research. Its
current applications are CryoLauncher, ICESEE [@kyanjo2026icesee], LIVIST
(Living Ice Sheet Temperature), and Frozen Legacies. Applications retain their domain-specific interfaces while reusing shared platform services where applicable. Supported workflows use local resources, institutional computing, or Cloud execution, currently backed by AWS (Amazon Web Services).

CryoStack is available at <https://cryostack.eas.gatech.edu/> and its source is maintained at <https://github.com/ICESEE-project/CryoStack>.

# Statement of need

Cryosphere research increasingly combines large datasets,
interactive data interpretation, numerical ice-sheet models, inverse methods,
ensemble simulations, and scalable computing. These components have different
software and resource requirements. Data discovery and quality control benefit
from interactive maps and browser interfaces. Model development may occur on
a workstation before simulations are executed on a range of different high-performance architectures. Ensemble assimilation and production simulations commonly require Message Passing Interface (MPI)-enabled libraries, batch schedulers, or programmatic access to cloud resources. Historical radar datasets additionally require dataset-specific ingestion, geolocation, quality-control tools, and preservation of provenance.

Today these capabilities are frequently delivered as separate repositories configured via
manual procedures executed locally. A researcher may need to install compiled model
dependencies, translate data formats, write scheduler scripts, transfer
inputs, track job identifiers, retrieve outputs, and remember which
parameters and environment created a result. The work is repeated when the
model, data product, institution, or compute system changes. This integration
burden disproportionately affects students and groups without dedicated
research-software or HPC support.

CryoStack is intended for researchers moving between exploratory and
production workflows, collaborators who do not share the same computing
environment, radar scientists preserving difficult legacy observations, and
instructors who need a consistent entry point for introducing students to ice sheet modeling and data analysis.
Its architectural goal is to make transitions among applications, data, and
resources explicit, repeatable, and inspectable.

# State of the field

Computational cryosphere research is supported by mature scientific software
for numerical modeling, data assimilation, and observational analysis.
ISSM (Ice-sheet and Sea-level System Model) [@larour2012issm], and Icepack [@shapero2021icepack] are two different numerical models of ice sheet flow and evolution that provide complementary capabilities at different levels of complexity and in different software environments.
Ensemble data-assimilation frameworks such as ICESEE [@kyanjo2026icesee] provide capabilities for
state and parameter estimation within ice sheet models, including approaches tailored to ice-sheet
applications. Observational cryosphere research introduces
a different class of requirements around the discovery, preservation,
geolocation, interpretation, and visualization of
datasets. Initial observational applications in CryoStack are focused on ice-penetrating radar and borehole data.

Software capabilities are generally developed around their respective scientific
tasks. Numerical models provide the physics and solvers; data-assimilation
systems provide estimation algorithms; and observational applications provide
specialized interfaces for manipulating and analyzing specific datasets. Moving among
these capabilities, however, also requires infrastructure for user access,
persistent workspaces, configuration, data transfer, experiment tracking,
remote computing, and software deployment.

CryoStack addresses this integration layer rather than reimplementing the
underlying scientific algorithms or replacing their specialized interfaces.
It provides shared cyberinfrastructure through which different classes of
cryosphere applications can be deployed and, where applicable, connected to
heterogeneous computing resources. Keeping this layer separate from the
scientific applications allows the applications and computing environments
to evolve independently.

# Software design

CryoStack separates access and operations, scientific applications, execution
backends, and reproducible software environments (Figure 1). Its design treats
the scientific experiment, dataset, and application as related but independent
objects. Identity and experiment persistence live outside the applications,
while explicit containment checks enforce per-user workspace roots. A
capability registry states which configuration subsets, result packages,
licensing requirements, and execution environments apply to each model or
workflow. Deployable applications are declared in a registry rather than
hard-coded into a single service.

CryoStack standardizes the shared infrastructure through which applications access computing resources, connectivity, and cloud services, while allowing each application to retain its own interface and scientific workflow. Scientific configuration is separated from execution, allowing a
configuration to move across supported execution environments while
backend-specific resource and readiness checks remain separate. Execution
support is application- and workflow-specific rather than implemented through
one universal execution pipeline. Shared connectivity and cloud-onboarding
infrastructure are reused where applicable.

![CryoStack's layered architecture. Four scientific applications reuse platform
services and shared contracts. Execution support is workflow-specific; the
Local, Remote-HPC, and AWS Batch backends do not apply to every application.
Connector/Relay also supports private institutional connectivity for Cloud
workloads, a path not shown explicitly here.](cryostack_architecture.pdf){#fig:architecture}

The persistence layer stores users, sessions, saved configurations,
per-application workspace state, and experiments in SQLite [@sqlite]. Each
experiment records an immutable configuration snapshot together with the
application, backend, job identifiers, output paths, and a status-event
timeline. Canonical examples remain read-only and are copied into user-owned
working copies before editing or execution. A user can return to a saved
workspace, reuse a named configuration, and relate scheduler state to the
configuration that produced it. These records provide a basis for fuller
provenance, rather than a complete archival package of inputs, environments,
transformations, and outputs.

# Scientific applications

CryoStack currently brings together four scientific applications that cover
a diverse range of computational cryosphere research.

**CryoLauncher** provides an interface for configuring and running simulations with the
ice-sheet models ISSM and Icepack. Basic and Advanced modes operate over the
same underlying run configuration: Basic exposes a curated, solver-aware
parameter subset of model configurations that can be modified through dropdown menus and select parameter inputs, while Advanced provides access to the full working copy of model configurations scripts.
CryoLauncher supports Remote and Cloud execution, currently through
institutional high-performance computing resources and AWS. ISSM workflows use MATLAB and
require a license, while Icepack runs Firedrake-based notebooks or scripts
without any proprietary dependencies.

**ICESEE** provides ensemble-based state and parameter estimation
[@kyanjo2026icesee]. It currently couples with ISSM, Icepack, and test implementations of the Lorenz-96 model, and a one-dimensional glacier flowline model [@kyanjo_2026_21796612], with dependencies determined by the selected
model. ICESEE supports Local, Remote, and Cloud execution through its own
submission and result-handling implementation while reusing CryoStack's
shared connectivity and cloud-onboarding infrastructure where applicable.
It also currently retains its own structure for assimilation diagnostics rather than packaging these outputs through CryoStack's common result-handling conventions.

**Frozen Legacies** extends CryoStack capabilities to the preservation and reuse of
historical Antarctic ice-penetrating radar observations [@schroeder2019multidecadal]. CryoStack uses dataset manifests to describe the available datasets and their metadata, while ingestion adapters translate these datasets into a catalog of geolocated observation points, flight geometries, metadata, and links to derived products. The initial adapter targets newly processed data from historical airborne radar surveys, with additional derived products to be incorporated as processing continues.
Integrated interpretation tools support radar picking and tracing, producing
geolocated picks, travel-time or power measurements, ice-thickness estimates,
and quality-control records. A new historical collection can be incorporated
through a dataset manifest and ingestion adapter rather than a model execution
adapter.

**LIVIST** (Living Ice Sheet Temperature) provides an interactive application
for exploring Antarctic englacial-temperature products inferred from radar
observations and constrained by borehole measurements. Its front-end and
documentation are built and served through the CryoStack deployment registry,
while retaining its specialized interface for observational data exploration.

Unlike CryoLauncher and ICESEE, LIVIST and Frozen Legacies are data-oriented applications rather than computational workflows organized around Local, Remote, or Cloud execution modes. CryoStack provides data-serving interfaces and adapters that allow these applications to expose data from different underlying sources, including external data repositories, without requiring the data to reside on the CryoStack server itself.

# Remote execution and the Connector/Relay

CryoStack's ``Remote''' path allows a user to bring computing resources they
already have access to, such as an institutional high-performance computing cluster or workstation,
into a CryoLauncher or ICESEE workflow. Each application retains its own
scheduler-aware implementation for submission, status, logs, termination,
and result retrieval. CryoLauncher and ICESEE therefore share the access
infrastructure described here, but can differ in their execution or result
pipeline.

Two access modes connect CryoStack to the target resource. **Direct** uses
SSH and file transfer from the application host when network policy and
reachability allow it. **Connector** supports resources that are reachable
only from the user's workstation, VPN, or an approved edge environment.
A packaged Connector exchanges commands and files through an authenticated
relay without requiring an inbound connection from CryoStack to the
institution (Figure 2). The target institution retains its accounts,
authentication, scheduler policy, and allocation enforcement.

Connector/Relay is shared connectivity infrastructure rather than an
execution backend. In addition to Remote access, the same infrastructure can
support Cloud workloads (see details in the next section) that need access to a private institutional service.
The implemented case is an ISSM workflow submitted through CryoLauncher to
AWS Batch, where the running container uses Connector/Relay to reach an
institutional MATLAB network-license service without exposing that service
directly to the cloud. The scientist supplies the institutional license
endpoint, while CryoStack handles the cloud-side license configuration and
connectivity. Cloud execution that does not require private institutional
connectivity does not require Connector.

![CryoStack's four applications and their shared connectivity. CryoLauncher
and ICESEE each implement their own Remote and Cloud execution; both reach
an institutional or HPC resource by direct SSH or through the shared
Connector/Relay. The same Connector/Relay can also carry a Cloud workload's
connection to a private institutional service (dashed) -- the demonstrated
case is CryoLauncher's ISSM workflow on AWS reaching the Georgia Tech MATLAB
license. Frozen Legacies and LIVIST are equally first-class CryoStack
applications with their own interfaces, not organized around
Local/Remote/Cloud.](cryostack_hpc_bridge.pdf){#fig:bridge}

# Cloud execution on AWS

Cloud execution is implemented through provider-independent driver contracts,
with AWS as the currently supported provider. A supported workflow
can be seamlessly moved between Remote and Cloud execution paths,  reusing its scientific configuration, although
resource settings and validation differ. A personal AWS account connects
through a role-based onboarding flow. AWS Batch on Fargate is the default
compute mode, while EC2 is available as an opt-in alternative.

Cloud support is qualified by workflow rather than by infrastructure
availability alone. CryoLauncher's Icepack workflows have completed
end-to-end execution on both Fargate and EC2 On-Demand using a single CPU
node. CryoLauncher's ISSM workflow has also completed end-to-end execution
on Fargate and EC2 On-Demand, including MPI-parallel solver execution,
post-processing into CryoStack's shared result package, retrieval, and
visualization. The ISSM runs accessed the configured Georgia Tech
institutional MATLAB license through the Connector/Relay infrastructure
described above.

ICESEE uses its own Cloud submission path. Its Lorenz-96 example has
completed Fargate execution under its single-process contract.

These demonstrations qualify specific workflows and configurations rather
than every infrastructure option exposed by the platform. They do not
establish scientific execution for EC2 Spot, custom networking, GPU, or
multi-node configurations.

Two environment strategies complement these execution paths. ICESEE-Spack
uses Spack [@gamblin2015spack] to resolve source builds against site
compilers, MPI implementations, and system libraries. ICESEE-Containers
uses Docker and Apptainer for preconfigured execution; Apptainer follows
the scientific-container approach described by @kurtzer2017singularity.

# Auto-config $\cdot$ Beta

Auto-config $\cdot$ Beta is available for the configurable modeling and data-assimilation workflows in CryoLauncher and ICESEE. In short, Auto-config generates configuration proposals for ice sheet simulations from simple English-language user prompts, based on the user's existing simulation examples
on CryoStack and its registered metadata. In technical terms, Auto-config produces deterministic configuration proposals from natural-language requests using the current configuration and existing model, example, parameter, resource, and capability metadata. The interface uses deterministic lexical matching and rule-based extraction to interpret supported requests within this
constrained configuration space. This follows the broader use of constrained natural-language interfaces to map human-readable requests into structured, machine-processable representations [@rutledge2022rapid; @caruso2024cnl2asp]. In Auto-config, this constrained interpretation is combined with capability-aware validation and provenance tracking, such that natural-language
requests produce reviewable configuration proposals rather than directly executable workflows.

The resulting proposal is checked against the registered capabilities and presented for user review rather than allowing the natural-language interface to directly execute a workflow. Manual controls remain authoritative. A compact change preview distinguishes values taken from the request, suggested adjustments, retained settings, and model context drawn from named examples in the user's own workspace. Omitted settings are preserved where compatible, while ambiguous or unsupported choices are not silently resolved.

The same interface supports read-only configuration diagnosis and explanations, together with deterministic repair proposals derived from the registered constraints. Refinements operate on the current controls rather than conversation history. Applying a proposal updates the configuration only and refuses stale or altered proposals. Existing validation and execution controls remain authoritative; Auto-config does not submit jobs, provision infrastructure, or configure licenses.

# Results, packaging, and visualization

For standalone CryoLauncher workflows, a completed ISSM or Icepack run is
exported to a transport-neutral, versioned result package containing metadata,
mesh geometry, field arrays, and available figures. If a tutorial produces no
plottable field, the package records what could and could not be exported
without turning successful scientific execution into a failed run.

A reader for each schema loads the package without requiring the model's
runtime: MATLAB for ISSM or Firedrake for Icepack. Both readers satisfy a
shared result-package protocol, and a shared visualization layer renders
fields and time series from the neutral package. A third model can gain a
results view by providing a conforming exporter and reader. The Icepack
exporter records its conversion of Firedrake fields to a first-order nodal
representation for display; its behavior on live Firedrake output still
requires confirmation.

This result-package pathway is specific to the standalone model workflows
currently handled by CryoLauncher. ICESEE retains its own assimilation
diagnostics and result-handling implementation, while Frozen Legacies and
LIVIST use application-specific interfaces for their observational data and
derived products.

Figure \ref{fig:cryolauncher-results} illustrates this standalone
results workflow through the CryoLauncher interface, showing an ISSM
velocity field alongside controls for field selection, rendering,
preview, and download.

![CryoLauncher results interface displaying the velocity magnitude
(`Vel`) from an ISSM stress-balance solution (`StressbalanceSolution`).
The panel provides solution and field selectors, field rendering,
and controls for previewing and downloading results.](images/cryolauncher-results.pdf){#fig:cryolauncher-results}

# Availability, verification, and limitations

Automated tests cover the gateway, authentication, model adapters,
Connector/Relay, cloud modules, and application-specific execution paths.
A separate offline acceptance command checks Auto-config safety properties,
capability-registry consistency, and workspace isolation. These tests exercise
software behavior without requiring live institutional or cloud resources;
they do not establish scientific correctness or replace the workflow-specific
live execution checkpoints described above.

Current deployment limitations include process-local Connector session state,
incomplete resource-policy and audit enforcement, and further qualification
needed for shared AWS accounts and recovery from operational failures. SQLite
supports the current single-node deployment; broader scaling requires shared
persistence and durable task and session storage. Packaged Connector binaries
must remain compatible with the relay protocol, and reference container images
need publication under a project registry namespace.

Scientific and workflow support also remains application-specific. Additional
model workflows and Cloud configurations require qualification before they can
be treated as supported, while model-specific result support and archival
provenance remain incomplete. Frozen Legacies packages historical radar data
and interpretation tools, but not every desktop workflow is currently
available as a browser-native, provenance-captured service. 

CryoStack is distributed under the MIT License. The source code is maintained
at <https://github.com/ICESEE-project/CryoStack>, and the version associated
with this paper is archived on Zenodo [@cryostack2026zenodo].

# Research impact statement

CryoStack's evidence to date is functional and architectural rather than
adoption-based. It brings together scientific applications and capabilities
grounded in published and peer-reviewed research, spanning ice-sheet modeling
with ISSM [@larour2012issm] and Icepack [@shapero2021icepack], ensemble data
assimilation with ICESEE [@kyanjo2026icesee], and observational applications
built around historical radar and radar/borehole-derived temperature products.
CryoStack provides shared cyberinfrastructure around these capabilities while
allowing each scientific application to retain its own interface and workflow.

For a research group, CryoStack provides a common entry point for moving
supported computational workflows among available computing resources and
for connecting observational applications to shared deployment and data
infrastructure. Researchers can extend the platform through application
registration, model adapters and result readers, or dataset manifests and
ingestion adapters, depending on the scientific task. This separation allows
new scientific capabilities to be incorporated without requiring them to
adopt a single execution or result-handling model.

# Acknowledgements

This work was supported in part by the U.S. National Science Foundation through CAREER Award No. 2235920. We gratefully acknowledge the Astera Institute for funding the backend infrastructure for the LIVIST project. The authors also thank Renette Jones-Ivey of the University at Buffalo for her assistance with the initial Jupyter Book backend; Justin Quinn for helping launch the CryoStack-GHUB backends before their migration to a Georgia Tech virtual machine; Troy Hilley of Research Computing Services; and the Partnership for an Advanced Computing Environment (PACE) for providing computational resources. We further thank Research Scientist Fang (Cherry) Liu for her assistance in resolving challenges related to user authentication and remote access to PACE.

# References
