# homelab-k8s-environments-apps

## Overview

This repository is the workload application catalog and runtime configuration source in the homelab GitOps architecture.

It contains:

- Argo CD Application definitions for workloads.
- App-of-apps root structure per environment.
- Runtime Helm values for each app path.

## GitOps Architecture (4 Repositories)

- [homelab-k8s-argo-config](https://github.com/anvaplus/homelab-k8s-argo-config): Argo CD platform and AppProject.
- [homelab-k8s-base-manifests](https://github.com/anvaplus/homelab-k8s-base-manifests): chart source.
- [homelab-k8s-environments](https://github.com/anvaplus/homelab-k8s-environments): version registry.
- [homelab-k8s-environments-apps](https://github.com/anvaplus/homelab-k8s-environments-apps): app definitions and runtime values (this repository).

## Structure

Path convention:

environments/<environment>/<domain>/<appName>_<componentName>/

Example (dev):

```text
environments/
  dev/
    _root/
      root-homelab-dev.yaml
      root-app/
        kustomization.yaml
        root-web.yaml
    web/
      kustomization.yaml
      blog_hugo/
        blog_hugo.yaml
        values.yaml
```

## App-of-Apps Flow

1. root-homelab-dev points to environments/dev/_root/root-app.
2. root-app groups domain roots (for example root-web).
3. Domain roots point to domain folders (for example environments/dev/web).
4. Domain folder kustomization includes app Applications.

## Multi-Source Helm Contract

Each app Application uses three sources:

1. homelab-k8s-environments (version values)
2. homelab-k8s-environments-apps (runtime values)
3. homelab-k8s-base-manifests (chart path)

In current manifests, valueFiles order is:

1. $valuesRepo/.../values.yaml (this repository)
2. $valuesRepoDefault/.../values.yaml (homelab-k8s-environments)

Because later values win, homelab-k8s-environments should keep version-only keys.

## Runtime Values Owned Here

- deployment.spec.replicas
- image.repository and related image options
- envConf
- externalSecretsConfig
- resources
- other runtime settings needed by chart templates

Do not set version in this repository.

## Change Workflow

Adding a new app:

1. Add app folder and files in this repository.
2. Register app in domain kustomization and root chain.
3. Add matching version values path in homelab-k8s-environments.
4. Ensure chart path exists in homelab-k8s-base-manifests.
5. Verify sourceRepos allow-list in homelab-k8s-argo-config AppProject.

Updating an existing app:

1. Runtime behavior change: update values.yaml in this repository.
2. Version bump: update matching file in homelab-k8s-environments.
3. Chart behavior change: update homelab-k8s-base-manifests.
