# Pento


```shell
kubectl create namespace vote
```


```yaml
apiVersion: v1
kind: Service
metadata:
  name: vote-service
  namespace: vote
spec:
  selector:
    app: vote-deployment
  ports:
    - protocol: TCP
      port: 5000
      targetPort: 80
      nodePort: 31000
  type: NodePort

---
apiVersion: v1
kind: Service
metadata:
  name: redis
  namespace: vote
spec:
  selector:
    app: redis-deployment
  ports:
    - protocol: TCP
      port: 6379
      targetPort: 6379
  type: ClusterIP

---
apiVersion: v1
kind: Service
metadata:
  name: db
  namespace: vote
spec:
  selector:
    app: db-deployment
  ports:
    - protocol: TCP
      port: 5432
      targetPort: 5432
  type: ClusterIP

---
apiVersion: v1
kind: Service
metadata:
  name: result-service
  namespace: vote
spec:
  selector:
    app: result-deployment
  ports:
    - protocol: TCP
      port: 5001
      targetPort: 80
      nodePort: 31001
  type: NodePort
```

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: vote-deployment
  namespace: vote
  labels:
    app: vote-deployment
spec:
  replicas: 1
  selector:
    matchLabels:
      app: vote-deployment
  template:
    metadata:
      labels:
        app: vote-deployment
    spec:
      containers:
      - name: vote-deployment
        image: kodekloud/examplevotingapp_vote:before

---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: redis-deployment
  namespace: vote
  labels:
    app: redis-deployment
spec:
  replicas: 1
  selector:
    matchLabels:
      app: redis-deployment
  template:
    metadata:
      labels:
        app: redis-deployment
    spec:
      containers:
      - name: redis-deployment
        image: redis:alpine
        volumeMounts:
        - mountPath: /data
          name: redis-data
      volumes:
      - name: redis-data
        emptyDir: {}

---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: worker
  namespace: vote
  labels:
    app: worker
spec:
  replicas: 1
  selector:
    matchLabels:
      app: worker
  template:
    metadata:
      labels:
        app: worker
    spec:
      containers:
      - name: worker
        image: kodekloud/examplevotingapp_worker


---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: db-deployment
  namespace: vote
  labels:
    app: db-deployment
spec:
  replicas: 1
  selector:
    matchLabels:
      app: db-deployment
  template:
    metadata:
      labels:
        app: db-deployment
    spec:
      containers:
      - name: db-deployment
        image: postgres:9.4
        volumeMounts:
        - mountPath: /var/lib/postgresql/data
          name: db-data
      volumes:
      - name: db-data
        emptyDir: {}
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: result-deployment
  namespace: vote
  labels:
    app: result-deployment
spec:
  replicas: 1
  selector:
    matchLabels:
      app: result-deployment
  template:
    metadata:
      labels:
        app: result-deployment
    spec:
      containers:
      - name: result-deployment
        image: kodekloud/examplevotingapp_result:before
```
