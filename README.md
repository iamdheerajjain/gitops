# 🚀 GitOps with ArgoCD, Prometheus, Grafana, and Kind

![GitOps](https://img.shields.io/badge/GitOps-Automation-blue?style=for-the-badge)
![Kubernetes](https://img.shields.io/badge/Kubernetes-1.31-brightgreen?style=for-the-badge&logo=kubernetes)
![ArgoCD](https://img.shields.io/badge/ArgoCD-Continuous%20Delivery-orange?style=for-the-badge&logo=argo)
![Grafana](https://img.shields.io/badge/Grafana-Monitoring-yellow?style=for-the-badge&logo=grafana)
![Prometheus](https://img.shields.io/badge/Prometheus-Metrics-red?style=for-the-badge&logo=prometheus)
![License](https://img.shields.io/badge/License-MIT-lightgrey?style=for-the-badge)

---

## 🧠 Overview

**GitOps** is a modern approach to Continuous Delivery where your **Git repository acts as the single source of truth** for your infrastructure and applications. This project demonstrates a complete **GitOps workflow** using ArgoCD for automated deployments, Prometheus for metrics collection, and Grafana for visualization on a local Kubernetes cluster powered by Kind.

---

## 🧩 Tech Stack

| Tool | Purpose |
|------|---------|
| **Kind** | Local Kubernetes cluster for development |
| **ArgoCD** | GitOps continuous delivery & automation |
| **Prometheus** | Metrics collection & monitoring |
| **Grafana** | Data visualization & dashboards |
| **NGINX Ingress** | External access routing & load balancing |

---

## 🧠 Key Concepts Demonstrated

* **GitOps Workflow** — Continuous sync between Git repository and Kubernetes cluster
* **Declarative Configurations** — Infrastructure as Code using YAML manifests
* **Observability** — Real-time monitoring with Prometheus and Grafana
* **Security** — Namespaces, RBAC, and NetworkPolicies implementation
* **Scalability** — Multi-replica deployments and service mesh

---

## ⚙️ Architecture

```
              ┌──────────────────────────────┐
              │        GitHub Repo           │
              │ (manifests/, deployments/)   │
              └──────────────┬───────────────┘
                             │
                             ▼
                   ┌────────────────────┐
                   │     ArgoCD         │
                   │ Syncs manifests    │
                   │  from GitHub repo  │
                   └────────┬───────────┘
                            │
                            ▼
               ┌─────────────────────────────┐
               │   Kind Kubernetes Cluster   │
               │   (Deployed Application)    │
               └─────────────┬───────────────┘
                             │
                             ▼
          ┌──────────────────────────────────────┐
          │     Prometheus + Grafana Stack       │
          │  Metrics, Dashboards, Visualization  │
          └──────────────────────────────────────┘
```

---

## 🧰 Prerequisites

Before running this project, ensure you have the following installed:

- 🐳 **Docker** → [Install Docker](https://docs.docker.com/get-docker/)
- ⚙️ **Kind** → [Install Kind](https://kind.sigs.k8s.io/)
- ☸️ **kubectl** → [Install kubectl](https://kubernetes.io/docs/tasks/tools/)
- 🚀 **ArgoCD CLI** → [Install ArgoCD CLI](https://argo-cd.readthedocs.io/en/stable/cli_installation/)
- 📊 **Prometheus & Grafana** (optional for system monitoring)

---

## 🚀 Setup Instructions

### 1️⃣ Create a Kind Cluster

```bash
kind create cluster --name gitops
```

### 2️⃣ Install ArgoCD

Create the ArgoCD namespace:

```bash
kubectl create namespace argocd
```

Install ArgoCD:

```bash
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

Verify the installation:

```bash
kubectl get pods -n argocd
```

Wait until all pods are in `Running` state.

### 3️⃣ Access ArgoCD UI

Port forward to access the ArgoCD server:

```bash
kubectl port-forward svc/argocd-server -n argocd 8080:443
```

Get the initial admin password:

```bash
kubectl get secret argocd-initial-admin-secret -n argocd -o jsonpath="{.data.password}" | base64 -d
```

### 4️⃣ Login to ArgoCD CLI

```bash
argocd login localhost:8080 --username admin --password <your-password> --insecure
```

> **Note:** Replace `<your-password>` with the actual password retrieved in step 3.

### 5️⃣ Create GitOps Application

```bash
argocd app create gitops \
  --repo https://github.com/iamdheerajjain/gitops.git \
  --path manifests \
  --dest-server https://kubernetes.default.svc \
  --dest-namespace gitops
```

### 6️⃣ Sync the Application

```bash
argocd app sync gitops
```

### 7️⃣ Verify Deployment

Check if the application is deployed successfully:

```bash
kubectl get pods -n gitops
kubectl get svc -n gitops
```

---

## 📡 Access Your Application

You can expose the app in one of two ways:

### ➤ Option 1: Port Forwarding

```bash
kubectl port-forward svc/gitops-service 9090:80 -n gitops
```

Then open: **http://localhost:9090**

### ➤ Option 2: Using Ingress

If using NGINX ingress controller:

```bash
kubectl apply -f manifests/ingress.yaml
```

Add this line to `/etc/hosts`:

```bash
127.0.0.1 app.local
```

Now open: **http://app.local**

---

## 📈 Monitoring Setup

### 🔸 Prometheus

Prometheus scrapes metrics from your cluster and ArgoCD components.

Start Prometheus:

```bash
sudo systemctl start prometheus
sudo systemctl enable prometheus
```

Access Prometheus at: **http://localhost:9090**

### 🔸 Grafana

Grafana visualizes metrics collected by Prometheus.

Start Grafana:

```bash
sudo systemctl start grafana-server
sudo systemctl enable grafana-server
```

Access Grafana at: **http://localhost:3000**

**Default credentials:**
- Username: `admin`
- Password: `admin`

> **Note:** You can change these credentials in `/etc/grafana/grafana.ini`

---

## 🎯 GitOps Workflow

1. **Developer** pushes changes to the GitHub repository
2. **ArgoCD** detects the changes automatically
3. **ArgoCD** syncs the latest manifests to the Kubernetes cluster
4. **Kubernetes** deploys the updated application
5. **Prometheus** collects metrics from the application
6. **Grafana** visualizes the metrics in dashboards

---

## 🖼️ Sample Results

### ArgoCD Dashboard
![ArgoCD Dashboard](op1.jpg)

### Application Running
![Application Running](op2.jpg)

### Monitoring Dashboard
![Monitoring Dashboard](op3.jpg)

---

## 🛠️ Troubleshooting

### ArgoCD pods not starting
```bash
kubectl describe pods -n argocd
kubectl logs <pod-name> -n argocd
```

### Application not syncing
```bash
argocd app get gitops
argocd app sync gitops --force
```

### Port forwarding issues
Make sure no other process is using the port:
```bash
lsof -i :8080
lsof -i :9090
```

---

## 🧹 Cleanup

To remove all resources:

```bash
# Delete the ArgoCD application
argocd app delete gitops

# Delete the Kind cluster
kind delete cluster --name gitops
```

---

## 📚 Additional Resources

- [ArgoCD Documentation](https://argo-cd.readthedocs.io/)
- [Kind Documentation](https://kind.sigs.k8s.io/)
- [Prometheus Documentation](https://prometheus.io/docs/)
- [Grafana Documentation](https://grafana.com/docs/)
- [GitOps Principles](https://www.gitops.tech/)

---

## 📄 License

This project is licensed under the MIT License.

---

## 🤝 Contributing

Contributions are welcome! Please feel free to submit a Pull Request.

---

## 📧 Contact

For questions or feedback, please open an issue on GitHub.

---

**⭐ If you found this project helpful, please give it a star!**
