# Command Cheatsheet

Every command used so far, grouped by purpose. All are for macOS on Apple Silicon.

## Check installed tools

```bash
python3 --version
brew --version
docker --version
minikube version
kubectl version --client
helm version
which minikube                 # shows where a command lives
```

## Homebrew

```bash
# Install Homebrew
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"

# Add Homebrew to PATH (permanent, for both terminal types)
echo 'eval "$(/opt/homebrew/bin/brew shellenv)"' >> ~/.zprofile
echo 'eval "$(/opt/homebrew/bin/brew shellenv)"' >> ~/.zshrc

# Add Homebrew to PATH for the current terminal only
eval "$(/opt/homebrew/bin/brew shellenv)"
```

## Install tools

```bash
brew install --cask docker
brew install minikube kubectl helm
```

## Docker

```bash
open -a Docker                 # start Docker Desktop
docker run hello-world         # confirm Docker works
```

## Mac resources

```bash
sysctl -n hw.memsize | awk '{print $1/1024/1024/1024 " GB"}'    # total RAM
```

## minikube (cluster lifecycle)

```bash
minikube start --driver=docker --cpus=4 --memory=5g    # first start
minikube start                                         # later starts (remembers settings)
minikube stop                                          # end of day
minikube addons enable ingress
minikube addons enable metrics-server
minikube tunnel                                        # bridge Mac to cluster; Ctrl+C to stop
```

## kubectl (inspect the cluster)

```bash
kubectl get nodes
kubectl wait --for=condition=Ready node/minikube --timeout=120s
kubectl get pods -n kube-system
kubectl wait --namespace ingress-nginx --for=condition=Ready pod \
  --selector=app.kubernetes.io/component=controller --timeout=180s
kubectl top node               # current CPU and memory use
```

## Daily routine

```bash
# Start: open Docker Desktop, wait for the whale icon
minikube start

# Stop
minikube stop
```
