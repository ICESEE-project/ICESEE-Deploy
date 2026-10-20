# ICESEE-Deploy

ICESEE-Deploy is the deployment and reproducibility ecosystem for ICESEE (Ice Sheet State and Parameter Estimator), providing multiple pathways for installing, running, and extending ensemble data assimilation and ice-sheet modeling workflows across local machines, HPC systems, and cloud environments.

The ecosystem integrates three complementary software components:

* ICESEE-Spack — source-based package management for reproducible installation, development, and extension of ICESEE and supported ice-sheet models.
* ICESEE-Containers — prebuilt containerized environments for rapid, portable, and reproducible execution.
* CryoLauncher — a browser-based platform for launching, monitoring, and interacting with ICESEE and ice-sheet simulations without local installation.

Together, these components provide a unified framework for reproducible scientific software deployment, lowering barriers to entry for students, researchers, and developers while supporting scalable production workflows in cryosphere modeling and data assimilation.# ICESEE-DEPLOY
The ICESEE software ecosystem 


## JOSS submission snapshot

This deployment repository contains the manuscript in [`paper/paper.md`](paper/paper.md)
and the component revisions used for review. The historical `CryoLauncher/`
directory contains the [CryoStack core](https://github.com/ICESEE-project/CryoStack),
pinned to **cryostack-v1.0.0**. Container and Spack recipes have their own pinned
revisions. Obtain this snapshot with:

```bash
git clone --recurse-submodules https://github.com/ICESEE-project/ICESEE-Deploy.git
cd ICESEE-Deploy
git submodule update --init --recursive
```

Use the parent commit identified in the submission; avoid `--remote` when
reproducing its recorded component revisions.

### Minimal offline check

```bash
python3.11 -m venv .venv-review
source .venv-review/bin/activate
python -m pip install -r requirements-review.txt
cd CryoLauncher
python -m cryostack_src.acceptance --offline
```

This checks core software invariants without starting the UI or installing
scientific solvers. `MANUAL` results require built artifacts or live resources.
It does not validate scientific correctness or paid/institutional execution.
For scientific setup and examples, see the
[getting-started guide](CryoLauncher/icesee_jupyter_book/applications/icesee/getting_started.md),
[HPC/cloud guide](CryoLauncher/icesee_jupyter_book/docs/hpc_cloud.md),
[container instructions](ICESEE-Containers/README.md), and
[Spack instructions](ICESEE-Spack/README.md).

### Citation, support, and licenses

Cite the [CryoStack archival release](https://doi.org/10.5281/zenodo.22901225)
and record the source commits used. `CITATION.cff` describes this deployment
repository. Use [ICESEE-Deploy Issues](https://github.com/ICESEE-project/ICESEE-Deploy/issues)
for integration/manuscript issues and [CryoStack Issues](https://github.com/ICESEE-project/CryoStack/issues)
for platform behavior. See [CONTRIBUTING.md](CONTRIBUTING.md).

This deployment repository and the container recipes use BSD-2-Clause;
the CryoStack core and Spack tooling use MIT. Scientific dependencies and
applications retain their own licenses. The manuscript's MIT statement refers
to the core, not a replacement of the component licenses.
