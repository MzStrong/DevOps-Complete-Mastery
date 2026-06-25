# ภาคผนวก: คำสั่งที่ควรจำ

## Linux

```bash
ip a
ip route
ss -tulpen
curl -I http://example.com
systemctl status <service>
journalctl -u <service> -f
df -h
free -h
top
```

## Docker

```bash
docker build -t name:tag .
docker run -d --name app -p 8080:80 name:tag
docker ps
docker logs -f app
docker exec -it app sh
docker compose up -d --build
docker compose logs -f
```

## Kubernetes

```bash
kubectl get nodes
kubectl get pods -A
kubectl get all -n <namespace>
kubectl describe pod <pod>
kubectl logs <pod>
kubectl apply -f file.yaml
kubectl rollout status deployment/<name>
kubectl rollout undo deployment/<name>
```

## Helm

```bash
helm install <release> <chart> -n <namespace>
helm upgrade <release> <chart> -n <namespace>
helm rollback <release> <revision> -n <namespace>
helm history <release> -n <namespace>
```

---
