---
title: "Tolerations, Init containers and Security in Fission"
date: 2019-03-04T16:41:33+05:30
draft: false
author: "[Vishal Biyani](https://twitter.com/vishal_biyani)"
---

# Introduction

Fission now supports "podspec" in addition to earlier supported "container spec" which enable a Fission function user to extend the capabilities of the function.



## Tolerations

Taint two nodes with "reservation=fission" and two nodes with "reservation=microservices" as shown below:

```
$ kubectl taint nodes gke-vishal-fission-dev-default-pool-87c8b616-549c gke-vishal-fission-dev-default-pool-87c8b616-5q2c reservation=fission:NoSchedule

node "gke-vishal-fission-dev-default-pool-87c8b616-549c" tainted
node "gke-vishal-fission-dev-default-pool-87c8b616-5q2c" tainted

$ kubectl taint nodes gke-vishal-fission-dev-default-pool-87c8b616-pg05 gke-vishal-fission-dev-default-pool-87c8b616-t5q1 reservation=microservices:NoSchedule

node "gke-vishal-fission-dev-default-pool-87c8b616-pg05" tainted
node "gke-vishal-fission-dev-default-pool-87c8b616-t5q1" tainted
```

In the fission env spec, let's add spec and toleration for "reservation=fission"

```
    podspec:
      tolerations:
      - key: "reservation"
        operator: "Equal"
        value: "fission"
        effect: "NoSchedule"
```

Once we apply fission specs and run the function - you will notice that the pods go only on nodes with taints that match the toleration:

```
$kgpo $ff -owide
NAME                                                 READY     STATUS    RESTARTS   AGE       IP             NODE
newdeploy-pyfunc-default-kgsuik0l-66cd755675-jgjj6   2/2       Running   0          51s       10.16.177.16   gke-vishal-fission-dev-default-pool-87c8b616-549c
poolmgr-python-default-okhvkdsv-57b866b774-hbz7q     2/2       Running   0          49s       10.16.176.34   gke-vishal-fission-dev-default-pool-87c8b616-5q2c
poolmgr-python-default-okhvkdsv-57b866b774-hqnl2     2/2       Running   0          49s       10.16.176.35   gke-vishal-fission-dev-default-pool-87c8b616-5q2c
poolmgr-python-default-okhvkdsv-57b866b774-pmtzv     2/2       Running   0          49s       10.16.177.17   gke-vishal-fission-dev-default-pool-87c8b616-549c

```

Let's remove taint from all nodes:

```
$ kubectl taint nodes gke-vishal-fission-dev-default-pool-87c8b616-549c gke-vishal-fission-dev-default-pool-87c8b616-5q2c gke-vishal-fission-dev-default-pool-87c8b616-pg05 gke-vishal-fission-dev-default-pool-87c8b616-t5q1 reservation:NoSchedule-
```

## Volumes with InitContainers

```
    podspec:
      initContainers:
      - name: init-py
        image: fission/python-env 
        command: ['sh', '-c', 'cat /etc/infopod/labels']
        volumeMounts:
          - name: infopod
            mountPath: /etc/infopod
            readOnly: false
      volumes:
        - name: infopod
          downwardAPI:
            items:
              - path: "labels"
                fieldRef:
                  fieldPath: metadata.labels
```


```
$ kgpo $ff
NAME                                               READY     STATUS            RESTARTS   AGE
poolmgr-python-default-9eik2gxd-6fdc8d9696-hkkgn   0/2       Init:0/1          0          10s
poolmgr-python-default-9eik2gxd-6fdc8d9696-lpmgl   0/2       PodInitializing   0          10s
poolmgr-python-default-9eik2gxd-6fdc8d9696-tkmdc   0/2       PodInitializing   0          10s
```


```
$ k logs $ff poolmgr-python-default-9eik2gxd-6fdc8d9696-lpmgl -c init-py
environmentName="python"
environmentNamespace="default"
environmentUid="68e3f909-3e86-11e9-9378-42010aa00057"
executorInstanceId="dqaukdxy"
executorType="poolmgr"
pod-template-hash="2987485252"

```


## Security


**_Author:_**

* [Vishal Biyani](https://twitter.com/vishal_biyani)  **|**  [Fission Contributor](https://github.com/vishal-biyani)  **|**  CTO - [Infracloud Technologies](http://infracloud.io/)