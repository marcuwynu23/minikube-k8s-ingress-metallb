
# 🚀 Kubernetes Ingress with MetalLB External IP

This guide walks you through deploying an Nginx application, exposing it via Ingress, and setting up MetalLB to ensure the Ingress has a real external IP (`172.25.101.190`).

---

## 📁 Files

### 1️⃣ `nginx-deployment.yaml`

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
        - name: nginx
          image: nginx:latest
          ports:
            - containerPort: 80
---
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
spec:
  type: ClusterIP
  selector:
    app: nginx
  ports:
    - port: 80
      targetPort: 80
```

---

### 2️⃣ `ingress.yaml`

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: nginx-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  ingressClassName: nginx
  rules:
    - host: nginx.local
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: nginx-service
                port:
                  number: 80
```

---

### 3️⃣ `metallb-config.yaml`

```yaml
apiVersion: metallb.io/v1beta1
kind: IPAddressPool
metadata:
  namespace: metallb-system
  name: my-ip-pool
spec:
  addresses:
    - 172.25.101.190/32
---
apiVersion: metallb.io/v1beta1
kind: L2Advertisement
metadata:
  namespace: metallb-system
  name: my-l2-adv
```

---

## 🚀 Step-by-Step Guide

### 1️⃣ Start Minikube

```bash
minikube start --driver=hyperv
```

---

### 2️⃣ Enable the Ingress Addon

```bash
minikube addons enable ingress
```

---

### 3️⃣ Deploy Nginx Deployment and Service

```bash
kubectl apply -f nginx-deployment.yaml
```

---

### 4️⃣ Deploy MetalLB

Install MetalLB to manage LoadBalancer IPs:

```bash
kubectl apply -f https://raw.githubusercontent.com/metallb/metallb/v0.13.10/config/manifests/metallb-native.yaml
```

---

### 5️⃣ Configure MetalLB to Use the External IP (`172.25.101.190`)

Apply the custom IP pool and advertisement:

```bash
kubectl apply -f metallb-config.yaml
```

---

### 6️⃣ Apply the Ingress Resource

```bash
kubectl apply -f ingress.yaml
```

---

### 7️⃣ Verify Ingress External IP

```bash
kubectl get svc -n ingress-nginx
```

You should see:

```
NAME                      TYPE           CLUSTER-IP      EXTERNAL-IP      PORT(S)
ingress-nginx-controller  LoadBalancer   10.x.x.x        172.25.101.190   80:xxxxx/TCP,443:xxxxx/TCP
```

---

### 8️⃣ Update Your `hosts` File

Edit your hosts file (`C:\Windows\System32\drivers\etc\hosts` on Windows) and add:

```
172.25.101.190 nginx.local
```

---

### 9️⃣ Test Access!

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

## 💡 Key Notes

✅ **MetalLB bridges the gap** between the internal cluster network and your host by assigning a real external IP (`172.25.101.190`).

✅ **This simulates** how cloud load balancers work in real-world clusters.

✅ **Direct NodePort fallback:** If needed, access via:
```
http://172.25.101.190:<NodePort>
```

---

## 🎉 Done!

You now have:
- A real external IP for Ingress (`172.25.101.190`)
- Nginx served through Ingress
- Local dev environment that mimics cloud production traffic!

---

💡 **Feel free to expand** this with TLS/HTTPS, custom backends, or other Ingress configurations as needed!

---
