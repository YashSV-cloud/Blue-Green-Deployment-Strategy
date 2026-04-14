# Blue-Green Deployment using K3s and NGINX

## 🚀 Overview

This project demonstrates a **Blue-Green Deployment strategy** using Kubernetes (K3s) and NGINX. The goal is to deploy two versions of an application (Blue and Green) and switch traffic between them without any downtime.

Blue-Green deployment ensures that the existing version (Blue) continues to serve users while a new version (Green) is deployed and tested. Once verified, traffic is switched from Blue to Green instantly.

---

## 🎯 Objective

* Deploy two versions of an application (v1 and v2)
* Use NGINX as a reverse proxy to control traffic
* Achieve zero downtime deployment
* Enable easy rollback if issues occur

---

## 🧠 Concept

* **Blue Version (v1):** Currently running production version
* **Green Version (v2):** New version deployed alongside Blue
* **NGINX:** Acts as a traffic controller
* **Switching:** Done by updating NGINX configuration

Both versions run simultaneously, and only traffic routing changes.

---

## ⚙️ Prerequisites

* Linux/Ubuntu EC2 instance
* K3s installed
* kubectl configured
* Docker images for Blue and Green apps pushed to a registry (Docker Hub or ECR)
* Security group configured to allow NodePort access

---

## ☸️ Setup K3s Cluster

Install K3s:

```
curl -sfL https://get.k3s.io | sh -
```

Verify installation:

```
sudo kubectl get nodes
```

Configure kubectl:

```
export KUBECONFIG=/etc/rancher/k3s/k3s.yaml
sudo chmod 644 /etc/rancher/k3s/k3s.yaml
```

---

## 📦 Deploy Applications

Deploy Blue and Green versions:

```
kubectl apply -f blue-deployment.yaml
kubectl apply -f green-deployment.yaml
```

Verify pods:

```
kubectl get pods
```

---

## 🌐 Create Services

Expose both versions internally:

```
kubectl apply -f blue-service.yaml
kubectl apply -f green-service.yaml
```

These services route traffic to their respective pods using labels.

---

## ⚙️ Configure NGINX

Apply ConfigMap:

```
kubectl apply -f nginx-configmap.yaml
```

This contains:

* Upstream configuration
* Traffic routing logic

Initially, traffic is directed to Blue:

```
server blue-service:80;
```

---

## 🚦 Deploy NGINX

```
kubectl apply -f nginx-deployment.yaml
kubectl apply -f nginx-service.yaml
```

NGINX acts as:

* Reverse proxy
* Entry point for users
* Traffic switcher

---

## 🌍 Access Application

Get NodePort:

```
kubectl get svc nginx-service
```

Open in browser:

```
http://<EC2-PUBLIC-IP>:<NODE-PORT>
```

Expected output:

```
BLUE VERSION (v1)
```

---

## 🔁 Switch Traffic to Green

Edit NGINX ConfigMap:

```
kubectl edit configmap nginx-config
```

Change:

```
server blue-service:80;
```

To:

```
server green-service:80;
```

Restart NGINX:

```
kubectl rollout restart deployment nginx
```

---

## ✅ Verify Deployment

Refresh browser:

```
GREEN VERSION (v2)
```

Traffic is now successfully routed to the new version without downtime.

---

## 🔙 Rollback

If any issue occurs, revert configuration:

Edit ConfigMap:

```
kubectl edit configmap nginx-config
```

Change back to:

```
server blue-service:80;
```

Restart NGINX:

```
kubectl rollout restart deployment nginx
```

---

## 🧠 How It Works

### Before Switching

User → NGINX → Blue Service → Blue Pods

### After Switching

User → NGINX → Green Service → Green Pods

Only routing changes; no pods are stopped or restarted.

---

## 💡 Key Features

* Zero downtime deployment
* Instant traffic switching
* Easy rollback
* Both versions run simultaneously
* Safe production deployment strategy

---

## ⚠️ Common Issues

### Application not accessible

* Check Security Group inbound rules
* Verify NodePort is open

### Pods not running

```
kubectl describe pod <pod-name>
```

### Image pull error

* Ensure image exists in registry
* Verify image name in YAML

---

## 📌 Conclusion

This project demonstrates how Blue-Green deployment can be implemented using K3s and NGINX. By introducing a reverse proxy layer, traffic can be switched seamlessly between application versions, ensuring high availability and zero downtime.

---

## 🎯 One-Line Explanation

Blue-Green deployment is achieved by running two versions of an application and switching user traffic between them using NGINX without interrupting service.
