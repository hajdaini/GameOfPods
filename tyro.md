# TYRO

```shell
kubectl create role developer-role --verb=* --resource=services --resource=persistentvolumeclaims --resource=pods --namespace development -o yaml --dry-run > developer-role.yaml

# add the namespace in the yaml

kubectl create -f eveloper-role.yaml

kubectl create rolebinding developer-rolebinding --role=developer-role --user=drogo --namespace=development
```


```yaml
apiVersion: v1
kind: Service
metadata:
  name: jekyll
  namespace: development
spec:
  selector:
    run: jekyll
  ports:
    - protocol: TCP
      nodePort: 30097
      port: 8080
      targetPort: 4000
  type: NodePort
```


```yaml
apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
  labels:
    app: myapp
spec:
  containers:
  - name: myapp-container
    image: busybox:1.28
    command: ['sh', '-c', 'echo The app is running! && sleep 3600']
  initContainers:
  - name: copy-jekyll-site
    image: kodekloud/jekyll
    command: [ "jekyll", "new", "/site" ]
    volumeMounts:
    - mountPath: /site
      name: site
```


```yaml
apiVersion: v1
kind: Pod
metadata:
  name: jekyll
  namespace: development
  labels: 
    run: jekyll
spec:
  containers:
  - name: jekyll
    image: kodekloud/jekyll-serve
    volumeMounts:
    - mountPath: /site
      name: site
  initContainers:
  - name: copy-jekyll-site
    image: kodekloud/jekyll
    command: [ "jekyll", "new", "/site" ]
    volumeMounts:
    - mountPath: /site
      name: site
  volumes:
    - name: site
      persistentVolumeClaim:
        claimName: jekyll-site
```


```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: jekyll-site
  namespace: development
spec:
  accessModes:
    - ReadWriteMany
  volumeMode: Filesystem
  resources:
    requests:
      storage: 1Gi
```

