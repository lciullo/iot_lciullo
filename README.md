# iot_lciullo
A Kubernetes deployment repository for the **Inception of Things - Part 3** project using **Argo CD** for continuous deployment.

## Overview

This repository contains Kubernetes manifests for deploying the `wil42/playground` application in a K3d cluster. It demonstrates **GitOps** principles by storing infrastructure and application configuration in Git, with Argo CD automatically syncing changes to the cluster.

## Repository Structure

```
iot_lciullo/
├── README.md              # This file
├── application.yaml       # Argo CD Application manifest
└── deployment/
    └── deployment.yaml    # Kubernetes Deployment and Service
```

## Files Description

### `application.yaml`

Argo CD Application manifest that:
- Points to this repository as the source
- Watches the `deployment/` directory for manifests
- Automatically deploys to the `dev` namespace
- Enables auto-sync and self-healing

**Key Configuration:**
```yaml
source:
  repoURL: https://github.com/lciullo/iot_lciullo.git
  path: deployment
destination:
  namespace: dev
```

### `deployment/deployment.yaml`

Contains two Kubernetes resources:

#### 1. Deployment
- **Name**: `wil-playground`
- **Image**: `wil42/playground:v1` (or v2)
- **Replicas**: 1
- **Namespace**: `dev`

#### 2. Service
- **Name**: `wil-playground-service`
- **Type**: ClusterIP
- **Port**: 80 (internal Kubernetes port)
- **Target Port**: 8888 (application port)

## How It Works

### GitOps Flow

```
1. Developer modifies deployment.yaml
           ↓
2. Push changes to GitHub
           ↓
3. Argo CD detects changes
           ↓
4. Argo CD syncs new manifests to cluster
           ↓
5. Kubernetes applies changes
           ↓
6. New application version runs
```

### Port Mapping

- **Internal (Kubernetes)**: Service port `80`
- **Pod/Container**: Application port `8888`
- **Local machine** (via port-forward): `8888`

To access the application locally:
```bash
kubectl port-forward svc/wil-playground-service 8888:80 -n dev
# Then visit: http://localhost:8888
```

## Usage

### Initial Setup

The repository is automatically deployed by Argo CD when configured in Part 3:

```bash
cd /home/lciullo/Inception-of-things/p3
make install-tools
make config-cluster
```

### Testing Version Updates

To test continuous deployment:

1. **Update the image version** in `deployment/deployment.yaml`:
   ```yaml
   image: wil42/playground:v2  # Change from v1 to v2
   ```

2. **Commit and push to GitHub**:
   ```bash
   git add deployment/deployment.yaml
   git commit -m "Update playground to v2"
   git push origin main
   ```

3. **Argo CD will automatically detect and deploy the new version**

4. **Verify the deployment**:
   ```bash
   make version-status  # From p3 directory
   ```

## Application: wil42/playground

A simple HTTP server used for testing Kubernetes deployments.

- **v1**: Returns `{"status":"ok", "message":"v1"}`
- **v2**: Returns `{"status":"ok", "message":"v2"}`

Access it via:
```bash
curl http://localhost:8888/
# Output: {"status":"ok","message":"v1"}
```

## Kubernetes Commands

View deployment status:
```bash
kubectl get deployment -n dev
kubectl get pods -n dev
kubectl get svc -n dev
```

View Argo CD Application status:
```bash
kubectl get application -n argocd
kubectl describe application wil-playground-app -n argocd
```

Check application logs:
```bash
kubectl logs -n dev -l app=wil-playground -f
```

## GitOps Benefits

- ✓ **Declarative**: Infrastructure defined in Git
- ✓ **Auditable**: All changes tracked in version control
- ✓ **Automated**: Changes automatically deployed
- ✓ **Reversible**: Easy rollback to previous versions
- ✓ **Self-healing**: Cluster automatically syncs to desired state

## Related Projects

- **Inception of Things**: Main project repository
  - Part 1: K3s & Vagrant
  - Part 2: K3s with Ingress
  - Part 3: K3d and Argo CD (uses this repo)

## Author

Created for the 42 school Inception of Things project.

---

**Repository**: [https://github.com/lciullo/iot_lciullo](https://github.com/lciullo/iot_lciullo)

**Status**: Active | **Last Updated**: December 2024