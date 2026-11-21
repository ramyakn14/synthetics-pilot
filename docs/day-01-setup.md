# Day 1: Setup (Phase 0)

**Date:** 2026-11-18
**Goal:** install the tools and start a working local Kubernetes cluster.
**Result:** all tools installed, cluster running and healthy, ingress and metrics-server enabled.

## Starting point

| Tool | Before Day 1 |
| --- | --- |
| Python | 3.15.1 installed |
| Homebrew, Docker, minikube, kubectl, Helm | Not installed |

Machine: MacBook with Apple Silicon (arm64), 8 GB RAM.

---

## Step 1: Check what's installed

```bash
python3 --version
docker --version
minikube version
kubectl version --client
helm version
```

Only Python was present.

---

## Step 2: Install the tools

### 2a. Homebrew

Homebrew is the standard installer for developer tools on a Mac.

```bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

- Asks for your Mac login password (nothing shows while typing; that's normal).
- May install Apple's Command Line Tools first, which takes 5 to 15 minutes.

Add Homebrew to your PATH so the terminal can find it:

```bash
echo 'eval "$(/opt/homebrew/bin/brew shellenv)"' >> ~/.zprofile
eval "$(/opt/homebrew/bin/brew shellenv)"
brew --version
```

Result: `Homebrew 7.0.15`

### 2b. Docker Desktop

Docker runs containers. On a Mac, it runs a hidden Linux virtual machine, and every container lives inside it.

```bash
brew install --cask docker
open -a Docker
```

First launch: accept the terms, choose **Use recommended settings**, and **Skip** sign-in. Wait for the whale icon in the menu bar.

```bash
docker run hello-world
```

Result: `Hello from Docker!`

### 2c. minikube, kubectl and Helm

- **minikube** creates a one-node Kubernetes cluster on your laptop.
- **kubectl** is the command-line remote control for any Kubernetes cluster.
- **Helm** installs packaged apps (like Prometheus) into Kubernetes.

```bash
brew install minikube kubectl helm
minikube version
kubectl version --client
helm version
```

Results: minikube v1.39.0, kubectl v1.36.1, Helm v4.3.2.

---

## Step 3: Start the Kubernetes cluster

**Key ideas**

- **Kubernetes** runs and looks after containers: restarts them, scales them and routes traffic to them.
- A **cluster** is a group of machines; each machine is a **node**. Ours has one node.
- With `--driver=docker`, the minikube node is a Docker container acting as a small computer.

### 3a. Check your Mac's memory

```bash
sysctl -n hw.memsize | awk '{print $1/1024/1024/1024 " GB"}'
```

Result: 8 GB.

### 3b. Give Docker enough memory

In Docker Desktop: **Settings → Resources** → CPUs **4**, Memory **6 GB** → **Apply & restart**.

| Mac RAM | Docker CPUs | Docker memory | minikube memory |
| --- | --- | --- | --- |
| 8 GB | 4 | 6 GB | 5 GB |
| 16 GB or more | 4 | 10 GB | 8 GB |

Docker needs a little more than minikube, and the Mac needs memory left for itself.

### 3c. Start minikube

```bash
minikube start --driver=docker --cpus=4 --memory=5g
```

Done when you see: `Done! kubectl is now configured to use minikube cluster`.

### 3d. Check the cluster

```bash
kubectl get nodes
kubectl wait --for=condition=Ready node/minikube --timeout=120s
kubectl get pods -n kube-system
```

Result: node `minikube` Ready on Kubernetes v1.37.0, all system pods Running 1/1.

**What's in `kube-system`**

| Pod | Role |
| --- | --- |
| etcd | The cluster's database: stores everything |
| kube-apiserver | Front desk that every `kubectl` command talks to |
| kube-scheduler | Decides which node each pod runs on |
| kube-controller-manager | Keeps the cluster matching what you asked for |
| coredns | Turns service names like `backend` into addresses |
| kindnet | Network plugin that lets pods talk to each other |
| kube-proxy | Routes traffic to services |
| storage-provisioner | Creates storage volumes for pods |

**Reading `kubectl get pods`**

| Column | Meaning | Warning signs |
| --- | --- | --- |
| READY | Containers ready / total | `0/1` |
| STATUS | What the pod is doing | `CrashLoopBackOff`, `Pending`, `Error` |
| RESTARTS | Crash-and-restart count | A climbing number |
| AGE | Time running | Pods that keep getting recreated |

---

## Step 4: Enable ingress and metrics-server

- **Ingress:** the cluster's front door. An nginx web server that routes requests by path (`/` to frontend, `/api` to backend).
- **metrics-server:** measures current CPU and memory per pod so Kubernetes knows when to autoscale. Unlike Prometheus, it keeps no history.

```bash
minikube addons enable ingress
minikube addons enable metrics-server
```

After enabling ingress, minikube suggests running `minikube tunnel`. The tunnel bridges your Mac to the cluster inside Docker's VM, and only works while it runs in its own terminal. It isn't needed until the app is deployed; stop it with **Ctrl+C**.

Check both add-ons (in a second terminal):

```bash
kubectl wait --namespace ingress-nginx --for=condition=Ready pod --selector=app.kubernetes.io/component=controller --timeout=180s
kubectl top node
```

Results:

```
pod/ingress-nginx-controller-d7cd8c989-xzhxm condition met

NAME       CPU(cores)   CPU(%)   MEMORY(bytes)   MEMORY(%)
minikube   342m         5%       830Mi           14%
```

- **342m** = 342 millicores, about a third of one CPU core.
- **830Mi** = about 0.8 GB used by Kubernetes itself, which matches the memory budget estimate.

---

## Step 5: Make tools available in every terminal

Homebrew tools live in `/opt/homebrew/bin`. A terminal only finds commands in folders listed in its **PATH** variable. Some terminals (such as the one in VS Code) read `~/.zshrc` rather than `~/.zprofile`, so add the line there too:

```bash
echo 'eval "$(/opt/homebrew/bin/brew shellenv)"' >> ~/.zshrc
```

Open a new terminal and check:

```bash
which minikube
```

Result: `/opt/homebrew/bin/minikube`

---

## Memory budget (5 GB cluster on an 8 GB Mac)

| Component | Estimated memory | How we keep it small |
| --- | --- | --- |
| Kubernetes + ingress + metrics-server | ~0.8 GB (measured: 830Mi) | Already running |
| Prometheus | ~0.6 GB | Keep 2 days of data, scrape every 30s |
| Grafana | ~0.2 GB | Default settings |
| Redis + exporter | ~0.1 GB | Already tiny |
| Frontend + backend | 0.2 to 0.8 GB | Cap autoscaling at 3 replicas each |
| k6 UI test (real browser) | ~0.6 GB while running | Runs under a minute every few minutes |
| **Total** | **~2.5 to 3.1 GB** | Leaves headroom inside 5 GB |

Habits for an 8 GB Mac:

- Run the big load test from the Mac, not inside the cluster.
- Close heavy apps (browsers with many tabs, video calls) during load tests.
- Run `minikube stop` at the end of each day.

---

## Troubleshooting log

| Problem | Cause | Fix |
| --- | --- | --- |
| `brew: command not found` | Homebrew not installed yet | Install Homebrew (Step 2a) |
| Docker installed but no whale icon | App installed but never opened | `open -a Docker`, complete first-launch screens |
| `MK_USAGE: Docker Desktop has only 3916MB memory but you specified 8192MB` | minikube asked for more memory than Docker's VM had | Raise Docker memory in Settings → Resources; start minikube with `--memory=5g` |
| Node shows `NotReady` | Cluster only seconds old; network plugin still starting | Wait: `kubectl wait --for=condition=Ready node/minikube --timeout=120s` |
| `minikube: command not found` | Typo (`minicube`) or Homebrew not in that terminal's PATH | Check spelling; run `eval "$(/opt/homebrew/bin/brew shellenv)"`; add it to `~/.zshrc` |

## Lessons learned

- **Know your capacity at each layer** (Mac → Docker VM → minikube node → pods) before placing a workload.
- **"Still starting" vs "stuck":** new nodes and pods pass through a not-ready state; time is the main clue.
- **Check estimates against measurements:** `kubectl top node` confirmed the 0.8 GB estimate for Kubernetes itself.
- **"Works here, not there"** is very often a PATH or environment-variable difference.
