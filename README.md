# n8n-hosting

## 참고: n8n cloud resource spec

- Start: 320mb RAM, 10 millicore CPU burstable
- Pro (10k 실행): 640mb RAM, 20 millicore CPU burstable
- Pro (50k 실행): 1280mb RAM, 80 millicore CPU burstable

## kubernetes 실행

- postgres는 azure database for postgresql을 사용한다면 postgres 적용은 생략
- 현재 설정으로는 여러 파드가 동시에 실행될 수 없는 구조. 따라서 스케일링이나 고가용성을 고려하면 추가로 설정이 필요함(replicas, accessModes 설정 등)

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
kubectl apply -f kubernetes/n8n-ingress.yaml
kubectl apply -f kubernetes/n8n-service.yaml
```
