# Homelab Monitoring Stack

A production-pattern observability stack deployed on Kubernetes, built to mirror
the architecture used in enterprise edge environments (NCR Voyix SDS infrastructure).

## Stack

| Component | Role |
|---|---|
| **Prometheus** | Metrics collection and alert evaluation |
| **Grafana** | Dashboard visualisation (Node Exporter Full — ID 1860) |
| **Node Exporter** | Host-level metrics (CPU, RAM, disk, network) |
| **Alertmanager** | Alert routing with SLA-style thresholds |
| **k3d** | Local Kubernetes cluster (k3s in Docker) |

## Architecture
MacBook Air M5 (Docker Desktop)
└── k3d cluster: monitoring
└── namespace: monitoring
├── node-exporter   (DaemonSet — one pod per node)
├── prometheus      (Deployment — scrapes + evaluates rules)
├── grafana         (Deployment — dashboards on :3001)
└── alertmanager    (Deployment — alert routing on :9093)
## Prerequisites

- Docker Desktop
- [k3d](https://k3d.io) — `brew install k3d`
- kubectl — `brew install kubectl`

## Quick Start

```bash
# 1. Create the cluster
k3d cluster create monitoring \
  --agents 1 \
  --port "80:80@loadbalancer"

# 2. Deploy the full stack
kubectl apply -f namespace/namespace.yaml
kubectl apply -f node-exporter/
kubectl apply -f prometheus/
kubectl apply -f grafana/
kubectl apply -f alertmanager/

# 3. Access dashboards
kubectl port-forward svc/grafana 3001:3000 -n monitoring
kubectl port-forward svc/prometheus 9090:9090 -n monitoring
kubectl port-forward svc/alertmanager 9093:9093 -n monitoring
```

Open **http://localhost:3001** — login: `admin / homelab123`

## Alert Rules

| Alert | Threshold | Severity |
|---|---|---|
| NodeDown | Instance unreachable > 1 min | Critical |
| HighCPUUsage | CPU > 80% for 2 min | Warning |
| HighMemoryUsage | RAM > 85% for 2 min | Warning |
| HighDiskUsage | Disk > 85% for 5 min | Warning |

## Roadmap

- [ ] Phase 2 — Store cluster: CouchDB with replication simulation
- [ ] Phase 3 — Mock POS simulator (fake transaction generator)
- [ ] Phase 4 — Cross-cluster Prometheus federation
- [ ] Phase 5 — Failure drills (replication failure, node down, load spike)
- [ ] Slack/email integration for Alertmanager
- [ ] Persistent volumes for Prometheus storage
- [ ] Helm chart migration

## Project Context

Built as a personal learning project to expand Docker and Kubernetes skills,
modelled on real Software Defined Store (SDS) infrastructure — edge servers,
CouchDB replication, and remote observability.
