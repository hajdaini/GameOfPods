# Iron Gallery of Braavos


```yaml
apiVersion: v1
kind: Service
metadata:
  name: iron-gallery-service
spec:
  selector:
    run: iron-gallery
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
  type: ClusterIP

---
apiVersion: v1
kind: Service
metadata:
  name: iron-db-service
spec:
  selector:
    db: mariadb
  ports:
    - protocol: TCP
      port: 3306
      targetPort: 3306
  type: ClusterIP
```


```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: iron-gallery
  labels:
    run: iron-gallery
spec:
  replicas: 1
  selector:
    matchLabels:
      run: iron-gallery
  template:
    metadata:
      labels:
        run: iron-gallery
    spec:
      containers:
      - resources:
          limits:
            memory: "100Mi"
            cpu: "50m"
        name: iron-gallery
        image: kodekloud/irongallery:2.0
        volumeMounts:
        - mountPath: /usr/share/nginx/html/data
          name: config
        - mountPath: /usr/share/nginx/html/uploads
          name: images
      volumes:
      - name: config
        emptyDir: {}
      - name: images
        emptyDir: {}

---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: iron-db
  labels:
    db: mariadb
spec:
  replicas: 1
  selector:
    matchLabels:
      db: mariadb
  template:
    metadata:
      labels:
        db: mariadb
    spec:
      containers:
      - name: iron-db
        image: kodekloud/irondb:2.0
        volumeMounts:
        - mountPath: /var/lib/mysql
          name: db
        env:
        - name: MYSQL_ROOT_PASSWORD
          value: Braavo
        - name: MYSQL_DATABASE
          value: lychee
        - name: MYSQL_USER
          value: lychee
        - name: MYSQL_PASSWORD
          value: lychee
      volumes:
      - name: db
        emptyDir: {}
```

```yaml
apiVersion: networking.k8s.io/v1beta1
kind: Ingress
metadata:
  name: iron-gallery-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  rules:
  - host: iron-gallery-braavos.com
    http:
      paths:
      - path: /
        backend:
          serviceName: iron-gallery-service
          servicePort: 80
```


```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: iron-gallery-firewall
spec:
  podSelector:
    matchLabels:
      db: mariadb
  policyTypes:
  - Ingress
  - Egress
  ingress:
  - from:
    - podSelector:
        matchLabels:
          run: iron-gallery
    ports:
    - protocol: TCP
      port: 3306
```
