# 🚀 Kubernetes Canary Rollouts with Argo Rollouts & FastAPI

This project demonstrates a **canary deployment strategy** using **Argo Rollouts** with a simple **FastAPI** application. It includes:
- Manual promotion at each stage

---

## 📦 Tech Stack
- Kubernetes (YAML based manifests)
- Argo Rollouts
- FastAPI
- NGINX Ingress Controller
- Docker

---

## 🏗️ Project Structure
```
.
├── app/                  # FastAPI source code
├── rollout/              # Rollout YAML, service, ingress
├── scripts/              # Rollout automation scripts
└── README.md
```

---

## ⚙️ Setup Instructions

### 1. Clone the repository
```bash
git clone https://github.com/vedant-kakde/argo-canary-demo.git
cd argo-canary-demo
```

### 2. Build and Push Docker Images
```bash
cd app/
# v1
docker build -t vedantkakde/fastapi-app:v1 .
docker push vedantkakde/fastapi-app:v1

# v2 (modify response in main.py)
docker build -t vedantkakde/fastapi-app:v2 .
docker push vedantkakde/fastapi-app:v2

```

### 3. Install Argo Rollouts & NGINX Ingress
```bash
kubectl create namespace argo-rollouts
kubectl apply -n argo-rollouts -f https://github.com/argoproj/argo-rollouts/releases/latest/download/install.yaml
kubectl apply -f https://kind.sigs.k8s.io/examples/ingress/deploy-ingress-nginx.yaml
```

### 4. Install the kubectl-argo-rollouts Plugin
```bash
curl -LO https://github.com/argoproj/argo-rollouts/releases/latest/download/kubectl-argo-rollouts-linux-amd64
chmod +x kubectl-argo-rollouts-linux-amd64
sudo mv kubectl-argo-rollouts-linux-amd64 /usr/local/bin/kubectl-argo-rollouts
```

### 6. Deploy Resources
```bash
cd rollout
kubectl apply -f service.yaml
kubectl apply -f ingress.yaml
kubectl apply -f rollout.yaml
```

---

## 🚦 Canary Rollout Lifecycle

### Initial Rollout (v1)
Your `rollout.yaml` sets the initial image:
```yaml
image: fastapi-app:v1
```

### Upgrade to v2

```bash
kubectl edit rollout fastapi-rollout #update env var and image tag to v2
```
This triggers the 4-phase rollout (5% → 25% → 50% → 100%).

### Watch Rollout:
```bash
kubectl argo rollouts get rollout fastapi-rollout --watch
```

### Promote Rollout Step-by-Step:
```bash
kubectl argo rollouts promote fastapi-rollout
```

### Abort and Rollback:
```bash
kubectl argo rollouts abort fastapi-rollout
```

### Automated Demo:
```bash
./scripts/rollout-demo.sh
```
---

---

## 📊 Health Checks
- `/health` → readiness probe

---

## 🧪 Test the App
```bash
curl -s http://<INGRESS-IP>/
```
You’ll see: `"Hello from v1"` or `"Hello from v2"`

---

## 📸 Demo Screenshots

> - Initial rollout
> - Promotion steps
> - Failure + rollback

---

## 👤 Audit and Logging

All rollout actions are traceable via:
```bash
kubectl argo rollouts get rollout fastapi-rollout
```
You can also use Argo Rollouts UI (`kubectl argo rollouts dashboard`) if enabled.

---

## 📌 Notes

- Namespace used: `default`
- Update `APP_VERSION` env var in rollout YAML when changing versions
- Use `kubectl describe` or `kubectl logs` to debug health checks

