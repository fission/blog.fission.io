+++
title = "Rook Nfs Fission"
date = "2020-04-19T16:00:28+05:30"
author = "Vishal Biyani"
description = "Run Fission functions backed by a NFS storage powered by Rook" 
categories = ["Fission", "Rook", "NFS", "PodSpec"]
type = "post"
+++

https://rook.io/docs/rook/v1.3/nfs.html


```
$ git clone --single-branch --branch release-1.3 https://github.com/rook/rook.git
$ cd rook/cluster/examples/kubernetes/nfs
$ kubectl create -f operator.yaml

namespace/rook-nfs-system created
customresourcedefinition.apiextensions.k8s.io/nfsservers.nfs.rook.io created
clusterrole.rbac.authorization.k8s.io/rook-nfs-operator created
serviceaccount/rook-nfs-operator created
clusterrolebinding.rbac.authorization.k8s.io/rook-nfs-operator created
deployment.apps/rook-nfs-operator created
clusterrole.rbac.authorization.k8s.io/rook-nfs-provisioner-runner created
clusterrolebinding.rbac.authorization.k8s.io/run-nfs-provisioner created
serviceaccount/rook-nfs-provisioner created
deployment.apps/rook-nfs-provisioner created

$ kubectl get pods -nrook-nfs-system
NAME                                    READY   STATUS    RESTARTS   AGE
rook-nfs-operator-6b9759bbb4-s5lmk      1/1     Running   0          2m43s
rook-nfs-provisioner-784bfdd6d7-6zr8v   1/1     Running   0          2m42s

```



```
apiVersion: v1
kind: Namespace
metadata:
  name:  rook-nfs
---
# A default storageclass must be present
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: nfs-default-claim
  namespace: rook-nfs
spec:
  accessModes:
  - ReadWriteMany
  resources:
    requests:
      storage: 1Gi
---
apiVersion: nfs.rook.io/v1alpha1
kind: NFSServer
metadata:
  name: rook-nfs
  namespace: rook-nfs
spec:
  serviceAccountName: rook-nfs
  replicas: 1
  exports:
  - name: share1
    server:
      accessMode: ReadWrite
      squash: "none"
    # A Persistent Volume Claim must be created before creating NFS CRD instance.
    persistentVolumeClaim:
      claimName: nfs-default-claim
  # A key/value list of annotations
  annotations:
  #  key: value
 ```

 ```
$ kubectl -n rook-nfs get pod -l app=rook-nfs

NAME         READY     STATUS    RESTARTS   AGE
rook-nfs-0   1/1       Running   0          2m

```

#### 

```
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: nfs-server
spec:
  replicas: 1
  selector:
    matchLabels:
      role: nfs-server
  template:
    metadata:
      labels:
        role: nfs-server
    spec:
      containers:
      - name: nfs-server
        image: gcr.io/google_containers/volume-nfs:0.8
        ports:
          - name: nfs
            containerPort: 2049
          - name: mountd
            containerPort: 20048
          - name: rpcbind
            containerPort: 111
        securityContext:
          privileged: true
        volumeMounts:
          - mountPath: /exports
            name: mypvc
      volumes:
        - name: mypvc
          gcePersistentDisk:
            pdName: gce-nfs-disk
            fsType: ext4
```            



gcloud compute disks create --size=200GB --zone=asia-south1-a gce-nfs-disk