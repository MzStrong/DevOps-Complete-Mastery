# 17. Lab 14: Deploy App เข้า Kubernetes

## วัตถุประสงค์

- สร้าง Namespace, Deployment, Service
- Scale app
- Rollout และ rollback

## สร้าง Namespace

```bash
kubectl create namespace devops-lab
```

## Deployment

```bash
nano simple-api-deployment.yaml
```

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: simple-api
  namespace: devops-lab
spec:
  replicas: 2
  selector:
    matchLabels:
      app: simple-api
  template:
    metadata:
      labels:
        app: simple-api
    spec:
      containers:
        - name: simple-api
          image: devops-control:5000/simple-api:1.0.0
          ports:
            - containerPort: 3000
          readinessProbe:
            httpGet:
              path: /health
              port: 3000
            initialDelaySeconds: 5
          livenessProbe:
            httpGet:
              path: /health
              port: 3000
            initialDelaySeconds: 15
          resources:
            requests:
              cpu: "100m"
              memory: "128Mi"
            limits:
              cpu: "500m"
              memory: "256Mi"
```

Apply:

```bash
kubectl apply -f simple-api-deployment.yaml
```

## Service

```bash
nano simple-api-service.yaml
```

```yaml
apiVersion: v1
kind: Service
metadata:
  name: simple-api
  namespace: devops-lab
spec:
  selector:
    app: simple-api
  ports:
    - port: 80
      targetPort: 3000
  type: ClusterIP
```

Apply:

```bash
kubectl apply -f simple-api-service.yaml
```

## ทดสอบใน cluster

```bash
kubectl run test-curl -n devops-lab --rm -it --image=curlimages/curl -- sh
curl http://simple-api/health
```

## Scale

```bash
kubectl scale deployment simple-api -n devops-lab --replicas=4
kubectl get pods -n devops-lab -o wide
```

## Rollout/Rollback

```bash
kubectl set image deployment/simple-api simple-api=devops-control:5000/simple-api:1.0.1 -n devops-lab
kubectl rollout status deployment/simple-api -n devops-lab
kubectl rollout undo deployment/simple-api -n devops-lab
```

## ตรวจสอบ

```bash
kubectl get all -n devops-lab
kubectl describe pod -n devops-lab <pod-name>
kubectl logs -n devops-lab <pod-name>
```

## ข้อสรุป

Deployment คุมจำนวน pod และ rollout ส่วน Service ทำให้ app ถูกเรียกด้วยชื่อคงที่ แม้ pod จะเปลี่ยน IP

---
