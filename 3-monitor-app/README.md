# Module 7 - Monitoring on Kubernetes

**Goal**: Deploy Prometheus and Grafana to an existing Kubernetes cluster and build a basic dashboard backed by a sample metrics app.

## What you deploy

The manifests in `kubernetes/` create:

- `metrics-app`: a small Python service that exposes the `request_processing_seconds` metric on port 8000.
- `prometheus`: collects metrics from itself and from `metrics-app`.
- `grafana`: pre-provisioned with a Prometheus data source.

All services use internal cluster ports - access via `kubectl port-forward`.

## Multi-Group Support

This exercise supports 4 groups (g1, g2, g3, g4) each running in their own namespace with isolated resources.

### Directory Structure

```
kubernetes/
├── base/                    # Base manifests (reusable)
│   ├── kustomization.yaml
│   ├── namespace.yaml
│   ├── metrics-app.yaml
│   ├── prometheus.yaml
│   ├── grafana.yaml
│   ├── prometheus-configmap.yaml
│   ├── grafana-configmap.yaml
│   └── metrics-app-configmap.yaml
└── overlays/                # Group-specific overlays
    ├── g1/                  # Group 1 -> namespace g1
    ├── g2/                  # Group 2 -> namespace g2
    ├── g3/                  # Group 3 -> namespace g3
    └── g4/                  # Group 4 -> namespace g4
```

## Prerequisites

- Complete exercise 2

## Step-by-step

### 1. Build image

Build a Docker Image and upload it to Docker Hub (remember to replace the DOCKERHUB_USERNAME in the base manifest `kubernetes/base/metrics-app.yaml`)

The image name should be: `YOUR_DOCKERHUB_USERNAME/metrics-app:latest`

### 2. Deploy to Kubernetes

Choose your group and deploy:

```bash
# For Group 1 (g1)
kubectl apply -k kubernetes/overlays/g1

# For Group 2 (g2)
kubectl apply -k kubernetes/overlays/g2

# For Group 3 (g3)
kubectl apply -k kubernetes/overlays/g3

# For Group 4 (g4)
kubectl apply -k kubernetes/overlays/g4
```

Wait for rollout to complete:

```bash
kubectl rollout status deployment/prometheus -n <your-namespace>
kubectl rollout status deployment/grafana -n <your-namespace>
kubectl rollout status deployment/metrics-app -n <your-namespace>
```

## Accessing Services

All services are internal-only. Use `kubectl port-forward` to access them from your local machine.

### Prometheus

```bash
kubectl port-forward svc/prometheus -n <your-namespace> 9090:9090
```

Then open: `http://localhost:9090`

Verify the `metrics_app` target is up (Status -> Targets).

### Grafana

```bash
kubectl port-forward svc/grafana -n <your-namespace> 3000:3000
```

Then open: `http://localhost:3000`

Grafana credentials: `admin` / `admin`

Create a new dashboard panel. Use the Prometheus data source that is already configured and run a query such as `rate(request_processing_seconds_sum[1m])` to see the synthetic load produced by the metrics app.

## Cleanup

Once you are done exploring dashboards, clean everything up with:

```bash
kubectl delete -k kubernetes/overlays/<your-group>
```

## Notes

- The original `compose.yml` is still available if you want to compare the container-based setup with the Kubernetes manifests.
- All services use ClusterIP (internal) type - access requires kubectl port-forward.
- This provides better security than NodePort since services are not exposed on the cluster nodes.
