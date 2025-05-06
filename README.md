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
kubectl apply -f kubernetes/n8n-service.yaml
```

```bash
# 우회용 기본 rule 포함
az network application-gateway create \
  --name agw-aoai-kc-axpg-dev-01 \
  --resource-group rg-aoai-kc-aipg-dev \
  --location koreacentral \
  --sku Standard_v2 \
  --capacity 2 \
  --vnet-name vnet-aoai-kc-axpg-dev-01 \
  --subnet sbn-aoai-kc-axpg-dev-agw-01 \
  --public-ip-address axpg-agw-public-ip-01 \
  --frontend-port 80 \
  --http-settings-protocol Http \
  --http-settings-port 80 \
  --servers 127.0.0.1 \
  --priority 100 \
  --no-wait
```
