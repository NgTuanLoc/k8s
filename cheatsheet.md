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

## Mental model: spec vs status (Module 1)

```bash
kubectl apply -f file.yaml        # declarative: create OR update to desired state
kubectl delete -f file.yaml       # remove what the file declares

kubectl get pod NAME -o yaml      # full object: see spec: (desired) and status: (actual)
kubectl get pod NAME -o jsonpath='{.spec.containers[0].image}{"\n"}'   # desired image
kubectl get pod NAME -o jsonpath='{.status.phase}{"\n"}'               # actual phase

kubectl api-resources             # every kind you can create (+ short names)
kubectl explain pod               # docs for a kind
kubectl explain pod.spec.containers   # drill into any field
```

Reconcile loop = controller runs forever: observe → compare spec vs status → act.

## Pods: logs & exec (Module 2)

```bash
kubectl get pod NAME -o wide          # + Pod IP and node
kubectl describe pod NAME             # events, container states, why it's stuck

kubectl logs NAME                     # single-container pod
kubectl logs NAME -c CONTAINER        # pick a container (multi-container pod)
kubectl logs NAME --previous          # logs from the crashed previous instance

kubectl exec -it NAME -- sh                  # shell into first container
kubectl exec NAME -c CONTAINER -- cat /path  # run in a specific container
```

Pod = smallest unit; 1+ containers sharing one IP (`localhost`) + volumes (sidecar pattern).

<!-- More sections appended as modules are completed. -->
