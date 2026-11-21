# Synthetics Pilot

A hands-on SRE learning project: a small Python app (frontend + backend + Redis queue) running on a local Kubernetes cluster, exposed to the internet, monitored with synthetic API and UI tests, with all metrics in Grafana and autoscaling under load.

Everything is open source and runs on one laptop.

## Architecture

```
Internet ──> Cloudflare Tunnel ──> Ingress (nginx)
                                     ├── frontend pod (Flask UI)
                                     │      └── calls backend API
                                     └── backend pod (FastAPI)
                                            ├── pushes jobs ──> Redis (queue)
                                            └── worker threads process jobs
Prometheus <── scrapes /metrics (frontend, backend, Redis exporter)
Prometheus <── remote-write from k6 synthetics (API + UI tests)
Grafana    <── Prometheus
HPA / KEDA ──> scale frontend (CPU) and backend (queue length)
```

## Tech stack

| Layer | Tool |
| --- | --- |
| App | Python, Flask (frontend), FastAPI (backend) |
| Queue | Redis + redis-exporter |
| Runtime | Docker Desktop, minikube, kubectl, Helm |
| Ingress | ingress-nginx (minikube add-on) |
| Public access | Cloudflare Tunnel (cloudflared) |
| Metrics and alerts | Prometheus, Grafana, Alertmanager (kube-prometheus-stack) |
| Synthetics and load | k6, k6 browser |
| Autoscaling | metrics-server + HPA, KEDA |

## Progress

| Phase | What | Status |
| --- | --- | --- |
| 0 | Setup: tools, cluster, add-ons | Done (except project folder) |
| 1 | Build the app (backend, frontend, metrics) | Next |
| 2 | Containerize and deploy to minikube | Not started |
| 3 | Prometheus + Grafana dashboard | Not started |
| 4 | Expose to the internet | Not started |
| 5 | Synthetics: API and UI tests | Not started |
| 6 | Autoscaling under load | Not started |
| 7 | SRE layer: SLOs, chaos drills, postmortem | Not started |

## Documentation

- [Day 1: Setup](docs/day-01-setup.md): every step, command and fix from setting up the Mac and the cluster
- [Day 2: Plan](docs/day-02-plan.md): what comes next
- [Command cheatsheet](docs/commands.md): all commands in one place

## Environment

- MacBook, Apple Silicon (arm64), 8 GB RAM
- Docker Desktop with 4 CPUs and 6 GB memory
- minikube cluster: 4 CPUs, 5 GB memory, Kubernetes v1.37.0

Because the Mac has 8 GB, the project runs on a memory budget: Prometheus keeps 2 days of data, autoscaling caps at 3 replicas per service, and load tests run from the Mac rather than inside the cluster.

## Daily start and stop

```bash
# Start of day: open Docker Desktop first, wait for the whale icon, then
minikube start

# End of day: free up memory
minikube stop
```
