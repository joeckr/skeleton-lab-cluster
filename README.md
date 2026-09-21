# skeleton-lab-cluster

A starter skeleton repository tailored as a source of truth, documentation, and multiple Helm charts across different flavors of Kubernetes (vanilla Kubernetes, Talos, OpenShift, etc.) in a lab environment.

> **Note**: This repository intentionally **does NOT include a Dockerfile**. If a custom image build is required, use an OCI image repository (such as `skeleton-oci-modified`). Lab cluster repositories focus on cluster configuration, deployment definitions, documentation, and Helm charts.

---

## Features

- **Helm Charts (`charts/`)**:
  - Multi-chart repository layout supporting multiple charts across Kubernetes flavors and components.
  - Automated recursive chart discovery, linting, packaging, and publishing in CI.
  - Starter chart (`charts/lab-cluster`) with support for Kubernetes and OpenShift (standard Ingress vs. OpenShift Routes).
  - Secure defaults: non-root execution (`runAsNonRoot: true`), `RuntimeDefault` seccomp profile, and dropping `ALL` capabilities (Pod Security Standards / OpenShift SCC compliant).
  - Master and control-plane node tolerations for compact lab clusters.
- **Environment & Tooling (`mise` & `hk`)**:
  - `mise.toml`: Tool version management (`helm`, `betterleaks`, `addlicense`, `trivy`, `actionlint`, `hadolint`, `shellcheck`, `zizmor`, `hk`, `pkl`, `tombi`, `yamllint`) and convenient task aliases.
  - `hk.pkl`: Fast git hooks powered by [`hk`](https://hk.jdx.dev/) enforcing Conventional Commits, branch protection, secrets scanning (`betterleaks`), Helm linting across charts, workflow linting (`actionlint`), security audits (`zizmor`), shell script linting (`shellcheck`), YAML linting (`yamllint`), TOML formatting (`tombi`), and license headers (`addlicense`).
- **GitHub Actions CI (`.github/workflows/`)**:
  - Reusable workflows powered by [`joeckr/ci-templates`](https://github.com/joeckr/ci-templates):
    - `lint.yml`: Workflow linting (`actionlint`), Conventional Commits validation (`commitlint`), and shell script linting (`shellcheck`).
    - `security.yml`: Secrets scanning (`betterleaks`) and workflow security audit (`zizmor`).
    - `release.yml`: Runs on push to `main` to compute SemVer tags, generate GitHub releases, and package & publish Helm charts under `charts/` to GHCR as OCI artifacts.
    - `test_release.yml`: PR dry-run validation for both Helm packaging and Semantic Versioning.

---

## Directory Structure

```text
.
├── .github/
│   ├── FUNDING.yml              # Ko-fi sponsorship configuration
│   └── workflows/
│       ├── lint.yml             # actionlint, commitlint, shellcheck
│       ├── release.yml          # SemVer tagging, GitHub releases, and Helm publishing to GHCR
│       ├── security.yml         # betterleaks secrets scanning and zizmor audit
│       └── test_release.yml     # PR dry-run tests for Helm and Semantic releases
├── charts/
│   └── lab-cluster/             # Starter Helm chart (add additional charts/wrappers here)
│       ├── Chart.yaml           # Helm chart definition
│       ├── values.yaml          # Default configuration values
│       ├── .helmignore          # Ignore rules for chart packaging
│       └── templates/
│           ├── deployment.yaml  # Workload deployment
│           ├── service.yaml     # Kubernetes Service
│           ├── ingress.yaml     # Kubernetes Ingress
│           └── route.yaml       # OpenShift Route
├── scripts/
│   └── template.sh              # Starter script placeholder
├── mise.toml                    # Mise tools and tasks
├── hk.pkl                       # hk git hooks configuration
└── README.md
```

---

## Quickstart

### 1. Bootstrap Local Environment

Ensure [`mise`](https://mise.jdx.dev/) and [`hk`](https://hk.jdx.dev/) are installed:

```bash
# Verify environment and install git hooks
mise run install

# Run checks across all files
mise run check
```

### 2. Helm Chart Development & Validation

#### Dependency Management & Linting

```bash
# Build Helm chart dependencies across all charts
mise run helm-dep

# Recursively lint all charts under charts/ (runs helm-dep first)
mise run helm-lint

# Or via command line directly
find charts -name "Chart.yaml" -exec dirname {} + | xargs helm lint
```

#### Template Rendering & Validation

Verify rendered manifests for Kubernetes or OpenShift environments:

```bash
# Render default templates (ClusterIP Service + Deployment)
helm template lab-cluster charts/lab-cluster/

# Render standard Kubernetes Ingress
helm template lab-cluster charts/lab-cluster/ \
  --set ingress.enabled=true

# Render OpenShift Route
helm template lab-cluster charts/lab-cluster/ \
  --set ingress.enabled=true \
  --set ingress.route="true"
```

#### Security & Vulnerability Scanning

Scan repository files and manifests for vulnerabilities and misconfigurations using Trivy:

```bash
# Run Trivy filesystem scan
mise run trivy-fs
```

### 3. Cluster Deployment & Testing

Deploy and test charts on your Kubernetes cluster (e.g. Talos Linux, vanilla Kubernetes, k3s, OpenShift):

```bash
# Install or upgrade chart in the current cluster context
helm upgrade --install lab-cluster charts/lab-cluster/

# Deploy with custom values or ingress enabled
helm upgrade --install lab-cluster charts/lab-cluster/ \
  --set ingress.enabled=true \
  --set ingress.host="lab.example.com"

# Check deployment status
kubectl get pods,svc,ingress -l app=lab-service

# For OpenShift clusters, inspect routes
oc get routes -l app=lab-service

# Teardown / uninstall release
helm uninstall lab-cluster
```

---

## Mise Tasks Reference

Run tasks with `mise run <task>`:

| Task | Description | Command |
|---|---|---|
| `install` | Install tools and set up git hooks | `hk install --mise` |
| `check` (or `hk`) | Run all linters and hook checks across repository | `hk check --all` |
| `helm-dep` | Build Helm chart dependencies across all charts | `find charts -name "Chart.yaml" -exec dirname {} + \| xargs -n1 helm dependency build` |
| `helm-lint` | Recursively lint all Helm charts under `charts/` | `find charts -name "Chart.yaml" -exec dirname {} + \| xargs helm lint` |
| `trivy-fs` | Scan repository filesystem for security vulnerabilities | `trivy fs .` |

---

## Security & Compliance

The charts in this repository are configured with secure production-ready defaults:

- **Pod Security Standards (PSS)**: Compatible with `restricted` and `baseline` admission policies:
  - `runAsNonRoot: true` enforces non-root container execution.
  - `seccompProfile.type: RuntimeDefault` restricts system calls.
  - `capabilities.drop: ["ALL"]` drops all Linux capabilities.
  - `allowPrivilegeEscalation: false` prevents elevation of privileges.
  - `automountServiceAccountToken: false` prevents unintended credential leakage to workloads.
- **OpenShift Security Context Constraints (SCC)**: Compatible with `restricted-v2` / `restricted` SCCs without requiring elevated service accounts.
- **Compact Lab Clusters**: Includes pre-configured tolerations for `node-role.kubernetes.io/control-plane` and `node-role.kubernetes.io/master`, allowing deployments on single-node or compact lab clusters where workloads share control-plane nodes.

---

## Conventional Commits & Releases

Commits must follow the [Conventional Commits](https://www.conventionalcommits.org/) specification:
- `feat: add new feature` -> Triggers a **minor** release (e.g., `v0.1.0` -> `v0.2.0`).
- `fix: resolve issue` -> Triggers a **patch** release (e.g., `v0.1.0` -> `v0.1.1`).
- `feat!: breaking change` -> Triggers a **major** release.
- `chore:`, `docs:`, `ci:`, `test:`, `refactor:` -> Maintenance changes (no release bump).

Upon merging to `main`, the `release.yml` workflow automatically computes the next version, creates a Git tag, publishes a GitHub Release, and recursively discovers, packages, and pushes all charts under `charts/` to GHCR as OCI artifacts.

## Support

If you find this project useful, consider supporting my work on [Ko-fi](https://ko-fi.com/joeckr):

[![ko-fi](https://ko-fi.com/img/githubbutton_sm.svg)](https://ko-fi.com/joeckr)

## License

Please refer to the `LICENSE` file for details.
