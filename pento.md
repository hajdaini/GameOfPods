# Pento

## master-node problem

### api-server

How I found the problem ?

```shell
docker ps -a | grep api
```

- SOLUTION part 1 : in the `/etc/kubernetes/manifests/kube-apiserver.yaml` change the value *"ca-authority.crt"* of the key *"--client-ca-file"* to *"ca.crt"*

- SOLUTION part 2 : in the ~/.kube/config change the cluster port to 6443

### coredns

```shell
kubectl edit deployments -n kube-system coredns
# use the right image k8s.gcr.io/coredns:1.3.1
```

## node01

```shell
kubectl uncordon node01
```


## Pod

```yaml
apiVersion: v1
kind: Service
metadata:
  name: gop-fs-service
spec:
  selector:
    app: gop-fileserver
  ports:
    - protocol: TCP
      port: 8080
      targetPort: 8080
  type: ClusterIP
```

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: data-pv
spec:
  capacity:
    storage: 1Gi
  accessModes:
    - ReadWriteMany
  hostPath:
    path: /web
    type: Directory
```


```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: data-pvc
spec:
  accessModes:
    - ReadWriteMany
  resources:
    requests:
      storage: 1Gi
```


```yaml
apiVersion: v1
kind: Pod
metadata:
  name: gop-fileserver
  labels:
    app: gop-fileserver
spec:
  containers:
  - name: gop-fileserver
    image: kodekloud/fileserver
    volumeMounts:
    - mountPath: /web
      name: data-store
  volumes:
  - name: data-store
    persistentVolumeClaim:
      claimName: data-pvc
```
