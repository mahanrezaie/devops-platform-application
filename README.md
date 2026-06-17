# ArgoCD Application Manifests

This repository contains Kubernetes application definitions managed by ArgoCD.

It is focused purely on **application workloads**, using shared Helm charts and environment-specific configurations.

Cluster-level components (like Prometheus, ingress controllers, storage classes, etc.) are managed in a separate repository.

---

## Structure

```
base/   → reusable Helm charts (application templates)stage/  → stage environment applications and overrides
```

Production follows the same structure pattern (mirrored in a separate branch or repo, depending on setup).

---

## Base Charts

The `base/` directory contains generic, reusable Helm charts used across multiple services.

### webapp-base

A standard Helm chart for HTTP-based applications.

It includes common Kubernetes resources:

- Deployment
- Service
- Ingress / HTTPRoute support
- HPA (autoscaling)
- ConfigMap for environment variables
- PersistentVolumeClaim
- ServiceAccount
- ServiceMonitor (for application metrics)
- Pre-upgrade / maintenance jobs
- Basic smoke tests

This chart is meant to be extended via values, not modified per service.

---

### cronjob-base

A lightweight Helm chart for scheduled workloads.

Includes:

- CronJob definition
- Optional PVC support
- ServiceAccount

Used for background tasks, batch jobs, and scheduled processing.

---

## Stage Environment

The `stage/` directory defines all applications deployed in the **staging environment**.

It contains:

### Application manifests

```
stage/apps/
```

These are ArgoCD application definitions pointing to Helm charts and values.

Examples:

- `ip-lookup.yaml` → IP lookup API deployment
- `postgres-cluster.yaml` → database workload (application-managed)

---

### Service-specific Helm releases

```
stage/devops-platform/ip-lookup-api/
```

This is a Helm-based ArgoCD application for the IP lookup service.

It includes:

- `Chart.yaml` → Helm chart definition
- `values.yaml` → environment-specific configuration
- `templates/servicemonitor.yaml` → Prometheus scraping configuration

This service typically builds on top of `base/webapp-base`.

---

### Platform / internal tools (app-level)

```
stage/tools/
```

These are internal platform applications that are still deployed via ArgoCD but are considered **application workloads**, not cluster infrastructure.

Example:

- PostgreSQL cluster (application-managed deployment)

---

## What this repo is NOT

This repository does **not** manage:

- Kubernetes cluster provisioning
- Prometheus / Grafana stack
- Ingress controller installation
- Storage classes / CSI drivers

Those are maintained in a separate cluster repository.

---

## Deployment model

ArgoCD continuously watches this repository and:

1. Reads application definitions from `stage/apps/`
2. Resolves Helm charts from `base/`
3. Applies environment-specific values from `stage/`
4. Syncs desired state into the Kubernetes cluster

---

## Design principles

- Keep Helm charts reusable and environment-agnostic
- Keep environment configuration isolated in `stage/`
- Treat each application as an independent deployment unit
- Avoid mixing cluster-level and app-level concerns
- Prefer composition over duplication

---

## Typical workflow

### Adding a new application

1. Create or reuse a base chart (`webapp-base` or `cronjob-base`)
2. Add a new application definition under `stage/apps/`
3. Add service-specific overrides under `stage/<project>/...`
4. Commit changes → ArgoCD syncs automatically

---

## Notes

- This repo is strictly for **application lifecycle management**
- Infrastructure and cluster services live in a separate repository
- Designed for GitOps-driven deployments using ArgoCD + Helm
