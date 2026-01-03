# homelab-k8s-environments-apps

## Overview

This repository contains all application configuration values and ArgoCD Application definitions for services deployed in the Homelab environment. It serves as the **configuration hub** in the GitOps workflow, defining how applications are deployed and configured across different environments.

*Note: This document represents a plan. The contents of this repository will evolve as the homelab is built and refined.*

## 🔗 Related Repositories

This repository is part of a 4-repository GitOps architecture:

- **[homelab-k8s-argo-config](https://github.com/anvaplus/homelab-k8s-argo-config)** - ArgoCD configurations for platform tools
- **[homelab-k8s-base-manifests](https://github.com/anvaplus/homelab-k8s-base-manifests)** - Helm chart templates for applications
- **[homelab-k8s-environments](https://github.com/anvaplus/homelab-k8s-environments)** - Version declarations for applications
- **[homelab-k8s-environments-apps](https://github.com/anvaplus/homelab-k8s-environments-apps)** (This repository) - Application configuration values

## Purpose

This repository provides:
- **ArgoCD Application definitions** for all services.
- **Helm values** for application configuration (e.g., resources, replicas, environment variables).
- A **root Application structure** that implements the App-of-Apps pattern.
- **Kustomization** for grouping related applications.
- **Environment-specific configurations** for development (DEV) and production (PROD).

## Repository Structure

The planned structure for this repository is as follows:
```
└── homelab/
    ├── dev/                      # Development environment configs
    │   ├── _root/                # Root application (App-of-Apps)
    │   │   ├── root-homelab-dev.yaml
    │   │   └── root-app/
    │   │       ├── kustomization.yaml
    │   │       ├── root-integration-layer.yaml
    │   │       └── root-documents.yaml
    │   ├── integration-layer/    # Integration applications
    │   │   ├── kustomization.yaml
    │   │   └── integration-app1/
    │   │       ├── integration-app1.yaml
    │   │       └── values.yaml
    │   └── documents/            # Document handling applications
    │       ├── kustomization.yaml
    │       └── document-app1/
    │           ├── document-app1.yaml
    │           └── values.yaml
    └── prod/                     # Production environment configs
        └── (same structure as dev)
```

## Key Components

### 1. Root Application (App-of-Apps Pattern)

The `root-homelab-<environment>.yaml` file in `homelab/<environment>/_root/` is the top-level ArgoCD Application that deploys other "root" applications for different application groups (e.g., `integration-layer`, `documents`).

### 2. Individual Application Definitions

Each application has its own directory containing:
- An **ArgoCD Application manifest** (`<app-name>.yaml`).
- A **`values.yaml` file** for its Helm chart configuration.

Example: `integration-layer/integration-app1/integration-app1.yaml`
```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: integration-app1
  namespace: argocd
spec:
  destination:
    namespace: integration-layer
    server: https://kubernetes.default.svc
  project: default
  sources:
    # Version repository (homelab-k8s-environments)
    - repoURL: 'https://github.com/anvaplus/homelab-k8s-environments.git'
      targetRevision: main
      ref: valuesRepoDefault
    
    # Configuration repository (this repo)
    - repoURL: 'https://github.com/anvaplus/homelab-k8s-environments-apps.git'
      targetRevision: main
      ref: valuesRepo
    
    # Chart repository (homelab-k8s-base-manifests)
    - repoURL: 'https://github.com/anvaplus/homelab-k8s-base-manifests.git'
      targetRevision: main
      path: 'charts/common' # Or a specific chart
      helm:
        valueFiles:
          - $valuesRepo/homelab/dev/integration-layer/integration-app1/values.yaml
          - $valuesRepoDefault/homelab/dev/integration-layer/integration-app1/values.yaml
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
```

### 3. Helm Values Files

The `values.yaml` file for each application contains its specific configuration, such as:
- `replicaCount`
- `resources` (requests and limits)
- `env` (environment variables)
- `service` configuration
- `ingress` rules

### 4. Kustomization Files

`kustomization.yaml` files are used to group related applications, making them easier to manage.

## How It Works

This repository defines *what* to deploy and *how* it should be configured. The `homelab-k8s-environments` repository specifies the *version* of the application to deploy, and `homelab-k8s-base-manifests` provides the Helm chart templates.

ArgoCD combines these sources to render the final Kubernetes manifests and apply them to the cluster.

## Making Changes

### Adding a New Application

1. Create a directory for the new application within the appropriate domain and environment (e.g., `homelab/dev/new-domain/new-app/`).
2. Add the ArgoCD Application manifest and a `values.yaml` file.
3. Update the `kustomization.yaml` in the domain's directory to include the new application.
4. Create a corresponding version entry in the `homelab-k8s-environments` repository.
5. Submit a pull request for review.

### Updating Application Configuration

1. Modify the `values.yaml` file for the application you want to change.
2. Submit a pull request with a clear description of the changes.
3. Once merged, ArgoCD will automatically sync the changes to the cluster.
