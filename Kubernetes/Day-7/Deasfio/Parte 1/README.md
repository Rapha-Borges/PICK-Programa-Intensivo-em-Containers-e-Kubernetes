### Desafio

```bash
cd manifests
vim headless-service.yaml
vim statefulset.yaml
kubectl apply -f headless-service.yaml
kubectl apply -f statefulset.yaml
kubectl set image statefulset/giropops-set nginx=nginx:1.19.0
```