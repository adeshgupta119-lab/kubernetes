# Kubernetes Monitoring Setup (Prometheus + Grafana)

## 1. Install Helm 3

```bash
curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash
```

### Verify Helm

```bash
helm version
```

Expected:

```text
version.BuildInfo{Version:"v3.x.x", ...}
```

---

## 2. Add Helm Repository

```bash
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update
```

---

## 3. Install Prometheus + Grafana

```bash
helm install monitoring prometheus-community/kube-prometheus-stack \
  -n monitoring \
  --create-namespace \
  --set grafana.service.type=NodePort \
  --set grafana.service.nodePort=32000 \
  --set prometheus.service.type=NodePort \
  --set prometheus.service.nodePort=32001
```

---

## 4. Verify Installation

### Check Pods

```bash
kubectl get pods -n monitoring
```

### Check Services

```bash
kubectl get svc -n monitoring
```

Expected:

```text
monitoring-grafana                     NodePort   ...   80:32000/TCP
monitoring-kube-prometheus-prometheus  NodePort   ...   9090:32001/TCP
```

---

## 5. Get Grafana Password

```bash
kubectl get secret monitoring-grafana \
-n monitoring \
-o jsonpath="{.data.admin-password}" | base64 -d && echo
```

### Login Credentials

```text
Username: admin
Password: <command output>
```

---

## 6. Access Using NodePort

### Grafana

```text
http://<EC2-PUBLIC-IP>:32000
```

### Prometheus

```text
http://<EC2-PUBLIC-IP>:32001
```

### AWS Security Group

```text
TCP 32000
TCP 32001
```

---

## 7. Port Forward (Recommended for Kind Cluster)

### Grafana

```bash
kubectl port-forward -n monitoring svc/monitoring-grafana 3000:80 --address 0.0.0.0
```

### Prometheus

```bash
kubectl port-forward -n monitoring svc/monitoring-kube-prometheus-prometheus 9090:9090 --address 0.0.0.0
```

---

## 8. Access Using Port Forward

### Grafana

```text
http://<EC2-PUBLIC-IP>:3000
```

### Prometheus

```text
http://<EC2-PUBLIC-IP>:9090
```

### AWS Security Group

```text
TCP 3000
TCP 9090
```

---

## 9. Uninstall Monitoring Stack

```bash
helm uninstall monitoring -n monitoring
kubectl delete namespace monitoring
```

---

## 10. Useful Commands

```bash
kubectl get all -n monitoring
kubectl get svc -n monitoring
kubectl get secrets -n monitoring
kubectl get events -n monitoring --sort-by=.metadata.creationTimestamp
helm list -A
```
