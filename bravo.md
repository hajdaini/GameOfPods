# TYRO

```shell
ssh root@node01 "mkdir /drupal-mysql-data /drupal-data"
```

```shell
kubectl create secret generic drupal-mysql-secret --from-literal=MYSQL_ROOT_PASSWORD=root_password --from-literal=MYSQL_DATABASE=drupal-database --from-literal=MYSQL_USER=root
```

```yaml
apiVersion: v1
kind: Service
metadata:
  name: drupal-service
spec:
  selector:
    app: drupal
  ports:
    - protocol: TCP
      nodePort: 30095
      port: 8080
      targetPort: 8080
  type: NodePort

---
apiVersion: v1
kind: Service
metadata:
  name: drupal-mysql-service
spec:
  selector:
    app: drupal-mysql
  ports:
    - protocol: TCP
      port: 3306
      targetPort: 3306
  type: ClusterIP
```

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: drupal-pv
spec:
  capacity:
    storage: 5Gi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: /drupal-data
    type: Directory

---
apiVersion: v1
kind: PersistentVolume
metadata:
  name: drupal-mysql-pv
spec:
  capacity:
    storage: 5Gi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: /drupal-mysql-data
    type: Directory
```


```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: drupal-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 5Gi

---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: drupal-mysql-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 5Gi
```


```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: drupal
  labels:
    app: drupal
spec:
  replicas: 1
  selector:
    matchLabels:
      app: drupal
  template:
    metadata:
      labels:
        app: drupal
    spec:
      containers:
      - name: drupal
        image: drupal:8.6
        volumeMounts:
        - mountPath: /var/www/html/modules
          name: volume-drupal-pvc
          subPath: modules  
        - mountPath: /var/www/html/profiles
          name: volume-drupal-pvc
          subPath: profiles  
        - mountPath: /var/www/html/sites
          name: volume-drupal-pvc
          subPath: sites
        - mountPath: /var/www/html/themes
          name: volume-drupal-pvc
          subPath: themes 
      initContainers:
      - name: init-sites-volume
        image: drupal:8.6
        command: [ "/bin/bash", "-c" ]
        args: [ 'cp -r /var/www/html/sites/ /data/; chown www-data:www-data /data/ -R' ]
        volumeMounts:
        - mountPath: /data
          name: volume-drupal-pvc
      volumes:
        - name: volume-drupal-pvc
          persistentVolumeClaim:
            claimName: drupal-pvc

---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: drupal-mysql
  labels:
    app: drupal-mysql
spec:
  replicas: 1
  selector:
    matchLabels:
      app: drupal-mysql
  template:
    metadata:
      labels:
        app: drupal-mysql
    spec:
      containers:
      - name: drupal
        image: mysql:5.7
        volumeMounts:
        - mountPath: /var/lib/mysql
          name: volume-drupal-mysql-pvc
          subPath: dbdata
        env:
        - name: MYSQL_ROOT_PASSWORD
          valueFrom:
            secretKeyRef:
              name: drupal-mysql-secret
              key: MYSQL_ROOT_PASSWORD
        - name: MYSQL_DATABASE
          valueFrom:
            secretKeyRef:
              name: drupal-mysql-secret
              key: MYSQL_DATABASE
        - name: MYSQL_USER
          valueFrom:
            secretKeyRef:
              name: drupal-mysql-secret
              key: MYSQL_USER
      volumes:
        - name: volume-drupal-mysql-pvc
          persistentVolumeClaim:
            claimName: drupal-mysql-pvc
```
