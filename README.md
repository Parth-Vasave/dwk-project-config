# DevOps with Kubernetes – project configuration

Kubernetes configuration for the todo project. The application code, Dockerfiles and CI workflows are in [Parth-Vasave/devops-kubernetes](https://github.com/Parth-Vasave/devops-kubernetes); this repository only describes what runs in the cluster, and Argo CD deploys it (Exercise 4.10).

| Path | Contents |
| --- | --- |
| `project/base` | Shared manifests of todo-app, todo-backend and broadcaster |
| `project/overlays/staging` | `staging` namespace: 1 broadcaster replica that only logs, no backup, staging banner |
| `project/overlays/production` | `production` namespace: 6 broadcaster replicas sending to Discord, daily database backup |
| `project/argocd` | Argo CD Applications `project-staging` and `project-production` |
| `todo-app/manifests`, `todo-backend/manifests`, `todo-backend/backup`, `broadcaster/manifests` | Manifests of each service |

## How releases reach this repository

- A commit to `main` in the code repository builds new images and commits their tags to `project/overlays/staging`.
- A git tag in the code repository builds images from the tagged commit and commits their tags to `project/overlays/production`.

Both commits are made by `github-actions[bot]` with a deploy key that can write only to this repository. Argo CD tracks `main` here, so a release is deployed within Argo CD's polling interval. Changes to the manifests themselves are committed here directly and are deployed the same way.

## Secrets

Secrets are encrypted with SOPS (age) and decrypted inside Argo CD by the KSOPS generators in the overlays. The Argo CD repo server setup is described in the code repository (`argocd/ksops-patch.yaml` and `project/README.md`).

Build an overlay locally:

```bash
docker run --rm -v "$PWD":/repo -v ~/.config/sops/age/keys.txt:/keys.txt:ro -e SOPS_AGE_KEY_FILE=/keys.txt -w /repo \
  viaductoss/ksops:v4.5.1 kustomize build --enable-alpha-plugins --enable-exec project/overlays/staging
```
