# helm-demo

## Overview
This repository is a simple Helm for web applications consisting of the Frontend/Backend/Postgresql. 
It is designed to have 3 individual Helm charts as follow:

- **backend-db:** This is the Postgres DB which uses the Postgresql official image: `postgres:latest`
- **backend:** This is a Node application which is expected to connect the `backend-db` to the default port `5432` and expose the Web server port: `3000`
- **frontend:** This is also a Node application which is expected to connect to the `backend` and while it is running the Web server port: `3000`, it exposes the Web server port: `8080`

> NOTE: It is designed to have 3 individual Helm charts in order to follow the `separation of concerns` so that each Helm chart can be individually updated and installed; however, all are inter-connected where the `backend` depends on the `backend-db` and the `frontend` depends on the `backend`. For this reason, it is expected to be installed sequentially: `bankend-db`, `backend`, and `frontend`. While it is possible to set dependenciess and design in such the way that all are in the same package, it is a design decision following `separation of concerns`.

## Assumption
1. This demo is developed and tested on the following setup:
 - the Kubernetes cluster set up on Docker Desktop for MacOS v.4.22.1
 - Helm CLI: v3.12.3

The details of `kubectl` and Kubernestes cluster:

```
$ kubectl version
Client Version: v1.28.1
Kustomize Version: v5.0.4-0.20230601165947-6ce0bf390ce3
Server Version: v1.27.2
```
2. To test the frontend Web server, it is verfied by the `port-forward` and browse the `http://localhost:8080` on the testing machine running the Kubernetes clusrter on Docker Desktop 
3. All resources of this demo will be deployed into the `internal-tools` namespace. 

## Prerequisites
The following requirements must be met and refer to the official documents to set up all of these prerequisites: 

1. Docker Desktop has been installed
2. Enable Kubernetes on the Docker Desktop 
3. Required CLIs such as `helm` and `kubectl` have been installed. 

## Installation 

1. Check that `kubectl` can connect to the Kubernetest cluster. If not, please verify and check your setup before proceeding.

```
$ kubectl cluster-info
Kubernetes control plane is running at https://127.0.0.1:6443
CoreDNS is running at https://127.0.0.1:6443/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy

To further debug and diagnose cluster problems, use 'kubectl cluster-info dump'.

$ kubectl get pods -A
NAMESPACE        NAME                                          READY   STATUS    RESTARTS        AGE
kube-system      coredns-5d78c9869d-4bb67                      1/1     Running   2 (3h10m ago)   20h
kube-system      coredns-5d78c9869d-mgznw                      1/1     Running   2 (3h10m ago)   20h
kube-system      etcd-docker-desktop                           1/1     Running   2 (3h10m ago)   20h
kube-system      kube-apiserver-docker-desktop                 1/1     Running   2 (3h10m ago)   20h
kube-system      kube-controller-manager-docker-desktop        1/1     Running   2 (3h10m ago)   20h
kube-system      kube-proxy-n5429                              1/1     Running   2 (3h10m ago)   20h
kube-system      kube-scheduler-docker-desktop                 1/1     Running   2 (3h10m ago)   20h
kube-system      storage-provisioner                           1/1     Running   4 (3h9m ago)    20h
kube-system      vpnkit-controller                             1/1     Running   2 (3h10m ago)   20h
```
2. Run `kubectl create namespace internal-tools` to create a new namespace
3. Clone the Helm Charts

> git clone https://github.com/sunsern-k/helm-demo.git

3. Go to the `helm-demo`

> cd helm-demo

There are 3 Helm charts:

- **backend-db**
- **backend**
- **frontend**


5.  Install `backend-db`

> helm install --namespace internal-tools backend-db-release backend-db

```
NAME: backend-db-release
LAST DEPLOYED: Mon Sep  4 12:30:38 2023
NAMESPACE: internal-tools
STATUS: deployed
REVISION: 1
TEST SUITE: None
```


6.  Verify the the PostgreSQL is up and run

> kubectl -n internal-tools get pod 

``` 
NAME                                          READY   STATUS    RESTARTS   AGE
backend-db-release-postgres-b5784db84-625bd   1/1     Running   0          44s
```
   
7. Install `backend`

> helm install --namespace internal-tools backend-release backend 

```
NAME: backend-release
LAST DEPLOYED: Mon Sep  4 12:32:06 2023
NAMESPACE: internal-tools
STATUS: deployed
REVISION: 1
TEST SUITE: None
```

8. Verify that the `backend` is up and run

> kubectl -n internal-tools get pod                              

```
NAME                                          READY   STATUS    RESTARTS   AGE
backend-db-release-postgres-b5784db84-625bd   1/1     Running   0          110s
backend-release-86dff577dd-mwq9d              1/1     Running   0          21s
```
10. Install `frontend`

> helm install --namespace internal-tools frontend-release frontend

``` 
NAME: frontend-release
LAST DEPLOYED: Mon Sep  4 12:34:10 2023
NAMESPACE: internal-tools
STATUS: deployed
REVISION: 1
TEST SUITE: None
```

11. Verify that all pods are now up and run

> kubectl -n internal-tools get pod

```                               
NAME                                          READY   STATUS    RESTARTS   AGE
backend-db-release-postgres-b5784db84-625bd   1/1     Running   0          3m53s
backend-release-86dff577dd-mwq9d              1/1     Running   0          2m24s
frontend-release-7dfbdd55dd-b6f5n             1/1     Running   0          21s
```


11. Check the service name of the `frontend`

> kubectl -n internal-tools get svc

```
NAME                         TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)    AGE
backend-db-release-service   ClusterIP   10.111.75.131   <none>        5432/TCP   4m21s
backend-release-service      ClusterIP   10.97.90.110    <none>        3000/TCP   2m53s
frontend-release-service     ClusterIP   10.111.237.33   <none>        8080/TCP   49s
```

In this example, the `frontend-release-service` is the one. 

12. Set the port-forward so that it can be locally tested.
> kubectl --namespace internal-tools port-forward service/frontend-release-service 8080:8080

```
Forwarding from 127.0.0.1:8080 -> 3000
Forwarding from [::1]:8080 -> 3000

``` 
13. Go to the Web browser and access [http://localhost:8080](http://localhost:8080) and the similar screenshot below should be shown:

![Output example](/images/output.png)

## Clean up? 

To clean up all of resources from the `internal-tools` namespace, it can be archieved by running the `helm uninstall` as examples below:

```
helm --namespace internal-tools list --no-headers | awk '{print $1}' | while read r; do helm --n internal-tools uninstall $r; done 
```

Then verify that there is no any resource on the `internal-tools` namespace. 

> kubectl -n internal-tools get all


