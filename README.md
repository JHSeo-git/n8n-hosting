# n8n-hosting

## 참고: n8n cloud resource spec

- Start: 320mb RAM, 10 millicore CPU burstable
- Pro (10k 실행): 640mb RAM, 20 millicore CPU burstable
- Pro (50k 실행): 1280mb RAM, 80 millicore CPU burstable

## kubernetes 실행

```bash
kubectl apply -f kubernetes/namespace.yaml
```

```bash
kubectl apply -f kubernetes/postgres-secret.yaml
kubectl apply -f kubernetes/postgres-configmap.yaml
kubectl apply -f kubernetes/postgres-claim0-persistentvolumeclaim.yaml
kubectl apply -f kubernetes/postgres-deployment.yaml
kubectl apply -f kubernetes/postgres-service.yaml
```

```bash
kubectl apply -f kubernetes/n8n-secret.yaml
kubectl apply -f kubernetes/n8n-claim0-persistentvolumeclaim.yaml
kubectl apply -f kubernetes/n8n-deployment.yaml
kubectl apply -f kubernetes/n8n-service.yaml
```
