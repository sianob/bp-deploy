# 🩺 Blood Pressure App – Kubernetes Deployment

This repo contains two ways to deploy the Blood Pressure (BP) application on Kubernetes:

1. **`bp-k8s/`** – Basic Kubernetes manifests
2. **`bp-chart/`** – A Helm chart for templated and environment-specific deployments

---

## 🚀 Application Overview

The BP app includes:

- `mongo` – MongoDB backing database
- `backend` – Spring Boot API
- `frontend` – Web UI (Vite/React/etc.)
- `Ingress` – Exposes `/` and `/api` via `bp.local`

---

## ⚙️ Prerequisite: Install the NGINX Ingress Controller

Both deployment methods rely on an Ingress resource. You must install the NGINX Ingress controller into your cluster:

```bash
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.12.1/deploy/static/provider/cloud/deploy.yaml
```
or

```bash
- name: Install or upgrade Ingress NGINX via Helm
  run: |
    helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
    helm upgrade --install ingress-nginx ingress-nginx/ingress-nginx \
      --namespace ingress-nginx --create-namespace

```

Then confirm it's running:

```bash
kubectl get pods -n ingress-nginx
```

You can also check the logs

```bash
kubectl logs deployment/ingress-nginx-controller -n ingress-nginx
```

You should see:
```
ingress-nginx-controller-xxxxx   1/1   Running
```

> ✅ You only need to install the Ingress controller **once per cluster**.

---

## 📁 Option 1: Deploy with Raw Manifests (`bp-k8s/`)

This folder contains standalone `.yaml` files for Mongo, the frontend, backend, PVCs, and the Ingress.

### Steps:

1. Apply all resources:

```bash
kubectl apply -f bp-k8s/
```

2. Add this entry to your `/etc/hosts` file:

```plaintext
127.0.0.1 bp.local
```

3. Visit the app in your browser:

```
http://bp.local
```

---

## 📁 Option 2: Deploy with Helm (`bp-chart/`)

This folder contains a Helm chart that templates all resources and supports multiple environments (e.g., `dev`, `prod`), namespaces, and overrides.

### 🧾 Files Included:

- `Chart.yaml` – Chart metadata
- `values.yaml` – Default config
- `values-dev.yaml` – Dev-specific overrides (e.g., namespace `bp-dev`)
- `values-prod.yaml` – Prod-specific overrides (e.g., namespace `bp-prod`)
- `templates/` – Kubernetes manifests as templates

### 🧪 Install for Staging

```bash
helm install bp-app ./bp-chart -f bp-chart/values-staging.yaml --namespace bp --create-namespace
```

### 🚢 Install for Prod (AKS)

Ensure you are using the correct context e.g.

```bash
export KUBECONFIG=~/.kube/aks-config
```

```bash
helm install bp-prod ./bp-chart -f bp-chart/values-prod.yaml --namespace bp --create-namespace
```

Get the external IP for the cluster

```bash
kubectl get svc -n ingress-nginx
```
Add this to your hosts file i.e.

```bash
<EXTERNAL-IP>  bp.example.com
```

### 🔄 Upgrade a Release

```bash
helm upgrade bp-dev ./bp-chart -f bp-chart/values-dev.yaml -n bp
```

### 🗑 Uninstall

```bash
helm uninstall bp-dev --namespace bp
```

---

## 🛠 Troubleshooting

### Can't Access `bp.local`?

Make sure you've added this to your `/etc/hosts`:

```plaintext
127.0.0.1 bp.local
```

Also confirm that your Ingress controller is installed and the `bp-ingress` resource is created.

### Image Not Updating?

Make sure your `imagePullPolicy` is set to `Always` (this is already done in the Helm chart and manifest files).

### Connecting to the database

You can use mongocli or a gui like Studio 3T to connect to the mongo database running in kubernetes. First you need to use port forwarding to make it accessible via localhost

`kubectl -n bp-dev port-forward svc/mongo 27017:27017`

Then use the connection string: `mongodb://localhost:27017`

---

## 🧼 Cleanup

To remove everything:

### If using raw manifests:
```bash
kubectl delete -f bp-k8s/
```

### If using Helm:
```bash
helm uninstall bp-dev --namespace bp
kubectl delete namespace bp
```

---

## 🙌 Contributions

Feel free to fork and adapt this project. PRs welcome!

---

## 📎 License

MIT – use it, ship it, share it.

---

Notes
IS THIS A MICROSERVICE ?

- Frontend — React/Vite app in its own container

- Backend — Spring Boot API in its own container

- Database — MongoDB in its own container

- Ingress — routing traffic to the right service

Each component is:

- Independently deployable

- Containerized

- Communicating over a network

(That’s a key trait of microservices!)

| Trait | Do you have it? |
|-----|------|
|Multiple business-domain services | ❌ (You have one backend)|
|Independent data storage per service | ✅ (MongoDB is tied to backend)|
|Service-to-service communication | ❌ (No internal APIs between backend services)|
|Lightweight APIs per domain (e.g., /users, /orders) | ❌|
|Central API gateway / discovery / distributed tracing | ❌|

A modular monolith or a well-containerized app with separation of concerns, which is a great starting point.

✅ What Would Make It a Full Microservice App?
Split your backend into multiple services:

e.g., user-service, bp-service, tip-service

Each service with its own DB or data model

Use internal HTTP or gRPC APIs to communicate between services

Add a service mesh or API gateway (optional)

Separate deploys, pipelines, scaling logic

“One small, focused service that does one thing well and doesn’t share its database.”