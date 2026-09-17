# skeleton-lab-cluster

A starter skeleton repository tailored as a source of truth, documentation, and multiple Helm charts across different flavors of Kubernetes (vanilla Kubernetes, Talos, OpenShift, etc.) in a lab environment.

> **Note**: This repository intentionally **does NOT include a Dockerfile**. If a custom image build is required, use an OCI image repository (such as `skeleton-oci-modified`). Lab cluster repositories focus on cluster configuration, deployment definitions, documentation, and Helm charts.

---

## Features

- **Helm Charts (`charts/`)**:
  - Multi-chart repository layout supporting multiple charts across Kubernetes flavors and components.
  - Automated recursive chart discovery, linting, packaging, and publishing in CI.
  - Starter chart (`charts/lab-cluster`) with support for Kubernetes and OpenShift (Ingress vs. Routes).
  - Secure defaults: non-root execution (`runAsNonRoot: true`), `RuntimeDefault` seccomp profile, and dropping `ALL` capabilities.
  - Master and control-plane node tolerations for compact lab clusters.
- **Docker Compose (`docker-compose.yml`)**:
  - Spin up and test services locally without requiring a cluster.
  - Pre-configured with port forwarding and healthchecks.
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
├── docker-compose.yml           # Local lab service definition
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

### 2. Local Experimentation (Docker Compose)

Start the local lab service:

```bash
# Start container in detached mode
mise run compose

# Check status and logs
docker compose ps
mise run logs

# Stop container
mise run down
```

### 3. Kubernetes / Helm Experimentation

#### Lint and Template Locally
```bash
# Recursively lint all charts under charts/
mise run helm-lint

# Or via command line directly
find charts -name "Chart.yaml" -exec dirname {} + | xargs helm lint

# Test OpenShift Route rendering
helm template lab-cluster charts/lab-cluster/ --set ingress.enabled=true --set ingress.route=true
```

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
