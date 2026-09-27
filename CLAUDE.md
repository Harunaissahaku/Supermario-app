# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

This repo holds only Kubernetes manifests for deploying a Super Mario web game. There is no application source code, build system, linter, or test suite. The game comes from the prebuilt public image `sevenajay/mario`, which serves on port 80.

## Layout

- `kubernetes1/deployment.yaml`: the only real manifest. It is one multi-document YAML file with three resources:
  - `Deployment` `mario-deployment`: runs the `sevenajay/mario` container on port 80, with pods labeled `app: mario`.
  - `Service` `mario-service`: selects `app: mario` and maps port 80 to port 80. It is currently `NodePort`. Git history shows it has been switched between ClusterIP, LoadBalancer, and NodePort, so check the current value before changing it.
  - `Ingress` `mario-ingress`: uses `ingressClassName: nginx` and routes host `mario.local`, path `/`, to `mario-service:80`. It needs an NGINX ingress controller in the cluster, and `mario.local` must resolve to the ingress IP (for example, through `/etc/hosts`).
- `Kubernetes` (at the repo root) is an empty placeholder file, not a directory.

The label `app: mario`, the service name, and port 80 must stay consistent across all three resources.

## Commands

```bash
# Validate manifests without applying them
kubectl apply --dry-run=client -f kubernetes1/deployment.yaml

# Deploy / update
kubectl apply -f kubernetes1/deployment.yaml

# Inspect
kubectl get deploy,svc,ingress,pods -l app=mario
kubectl get svc mario-service   # shows the assigned NodePort

# Local access without ingress
kubectl port-forward svc/mario-service 8080:80

# Tear down
kubectl delete -f kubernetes1/deployment.yaml
```

On minikube, run `minikube addons enable ingress` for the Ingress to work. You can also use `minikube service mario-service` to open the NodePort.
