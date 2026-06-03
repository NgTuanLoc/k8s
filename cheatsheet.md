# 📋 Kubernetes Master Cheatsheet

A living quick-reference. Grows as we complete modules.

## Contexts & clusters (Module 0)

```bash
kubectl config get-contexts        # list reachable clusters
kubectl config current-context     # active cluster
kubectl config use-context NAME    # switch active cluster
kubectl cluster-info               # control-plane URL
kubectl get nodes                  # nodes (want STATUS=Ready)
kubectl get pods -A                # pods in ALL namespaces
```

```bash
kind create cluster --name k8s-course   # create kind cluster
kind delete cluster --name k8s-course   # delete it
minikube start / minikube stop          # minikube lifecycle
```

## Pods (Module 0 intro)

```bash
kubectl run NAME --image=IMG --restart=Never   # create a bare pod
kubectl get pods                               # list pods
kubectl delete pod NAME                         # delete a pod
```

<!-- More sections appended as modules are completed. -->
