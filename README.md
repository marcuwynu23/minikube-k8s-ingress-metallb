
# 🚀 Kubernetes Ingress with MetalLB External IP (Helm-Based)

This guide shows you how to deploy an Nginx application with Ingress, exposed via MetalLB for a real external IP (`172.25.101.190`), **using Helm for simpler management**.

---

## 📁 Helm Chart Structure

```
nginx-ingress-helm/
├── Chart.yaml
├── values.yaml
├── templates/
│   ├── deployment.yaml
│   ├── service.yaml
│   ├── ingress.yaml
│   ├── metallb-config.yaml
```

---

## 🚀 Step-by-Step Guide

### 1️⃣ Start Minikube

```bash
minikube start --driver=hyperv
```

---

### 2️⃣ Install MetalLB (for LoadBalancer support)

Install the MetalLB CRDs and components:

```bash
kubectl apply -f https://raw.githubusercontent.com/metallb/metallb/v0.13.10/config/manifests/metallb-native.yaml
```

---

### 3️⃣ Deploy Everything with Helm

✅ **Install the Helm chart**:

```bash
helm install nginx-ingress ./nginx-ingress-helm
```

✅ This single command will:
- Create the Nginx Deployment & Service
- Create the Ingress resource
- Create the MetalLB IP pool and advertisement for LoadBalancer
- Ensure the Ingress controller’s Service is automatically of type `LoadBalancer` for external access

---

### 4️⃣ Edit the Ingress Controller Service (If Needed)

If you used the **Minikube Ingress addon**, the Ingress controller’s Service may still be of type `NodePort` by default.  
✅ Change it to `LoadBalancer` using:

```bash
kubectl edit svc -n ingress-nginx ingress-nginx-controller
```

In the editor, change:

```yaml
spec:
  type: LoadBalancer
```

✅ Save and exit the editor. MetalLB will automatically assign the external IP (`172.25.101.190`).

---

### 5️⃣ Verify the External IP

```bash
kubectl get svc -n ingress-nginx
```

✅ You should see:

```
NAME                      TYPE           CLUSTER-IP      EXTERNAL-IP      PORT(S)
ingress-nginx-controller  LoadBalancer   10.x.x.x        172.25.101.190   80:xxxxx/TCP,443:xxxxx/TCP
```

---

### 6️⃣ Update Your `hosts` File

Edit your hosts file (`C:\Windows\System32\drivers\etc\hosts` on Windows) and add:

```
172.25.101.190 nginx.local
```

---

### 7️⃣ Test Access!

✅ In the browser:

```
http://nginx.local
```

✅ With curl:

```bash
curl -v http://nginx.local
```

You should see the default Nginx welcome page!

---

### 🔧 Uninstall

When you’re done, clean up everything:

```bash
helm uninstall nginx-ingress
```

✅ This removes all resources created by this Helm release.

---

## 💡 Key Notes

✅ **MetalLB assigns** the external IP (`172.25.101.190`) to your Ingress controller’s LoadBalancer Service, simulating a cloud load balancer.

✅ **Using Helm makes management simple**:
- `helm install` for deployment
- `helm upgrade` for changes
- `helm uninstall` for clean removal

✅ **You no longer need**:
- `kubectl apply` for individual YAMLs
- `kubectl edit` for changing Service types (except in the Minikube addon case)
- Manual MetalLB config updates

---

## 🎉 Done!

You now have:
- A real external IP for Ingress (`172.25.101.190`)
- Nginx served through Ingress
- A Helm-managed, cloud-like Kubernetes environment

💡 **Expand this** with TLS/HTTPS, advanced Ingress rules, or more services as needed!

---
