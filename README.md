# Exploring Containers

Source for [containers.bradpenney.io](https://containers.bradpenney.io) — containerization
from first principles: what a container is, the runtimes underneath it, Docker and Podman as
tools, and the security/supply-chain discipline production demands.

Built with [MkDocs](https://www.mkdocs.org/) + [Material](https://squidfunk.github.io/mkdocs-material/).
Pushes to `main` are published to the `gh-pages` branch via `mkdocs gh-deploy` (see `.github/workflows/ci.yaml`).

## Where This Fits in the bradpenney.io Network

Containers sit in the middle of the stack, not at either end. This site is the layer that lets
every other site assume you already know what a container is, instead of re-explaining it:

- **[linux.bradpenney.io](https://linux.bradpenney.io)** — the kernel primitives a container is
  actually built out of (namespaces, cgroups, capabilities), taught as general Linux mechanisms.
  This site picks up exactly where that leaves off: the same primitives, applied.
- **[k8s.bradpenney.io](https://k8s.bradpenney.io)** — the orchestration layer that *consumes*
  containers as its unit of deployment (kubelet, the CRI, Pods). It doesn't teach what a
  container is; this site does.
- **[gitops.bradpenney.io](https://gitops.bradpenney.io)** — the pipeline that automates the
  `docker build` / `docker push` this site teaches by hand, and deploys the resulting image via
  Flux and OCI artifacts.
- **[bradpenney.io](https://bradpenney.io)** — the hub all of these (and cs/python/networking/
  storage/gitops) link out from.

The short version: Linux explains the kernel, this site explains the container, Kubernetes
explains what runs it at scale, and GitOps explains how it gets there without anyone typing
`docker push` by hand.

## Local development

```bash
poetry install
poetry run mkdocs serve   # live preview at http://127.0.0.1:8000
poetry run mkdocs build --strict   # what CI runs before deploying
```
