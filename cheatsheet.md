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

## Deployments: scale, rollout, rollback (Module 3)

```bash
kubectl get deploy,rs,pods -l app=web     # see all three layers
kubectl scale deploy/web --replicas=5     # scale

kubectl set image deploy/web nginx=nginx:1.26   # roll out new image
kubectl rollout status deploy/web               # watch (blocks till done/fails)
kubectl rollout history deploy/web              # list revisions
kubectl rollout undo deploy/web                 # roll back to previous
kubectl rollout undo deploy/web --to-revision=1 # roll back to specific
kubectl rollout restart deploy/web              # recreate all pods
```

Deployment → ReplicaSet → Pods. The ReplicaSet self-heals; `selector` must match `template.labels`.

## Deploy strategies (Module 4)

```bash
# strategy.type: RollingUpdate (default) | Recreate
# rollingUpdate.maxSurge (extra pods) / maxUnavailable (missing pods)
kubectl get pods -l app=NAME -w               # watch pods cycle during a roll

# blue-green: flip the Service selector (instant, atomic)
kubectl patch svc bluegreen -p '{"spec":{"selector":{"app":"bluegreen","version":"green"}}}'

# canary: split traffic by replica ratio behind one shared Service
kubectl scale deploy/canary-canary --replicas=3   # promote
kubectl scale deploy/canary-stable --replicas=0

kubectl run tmp --image=curlimages/curl --restart=Never -i --rm -- -s http://SVC   # test from inside
```

Recreate = downtime; RollingUpdate = zero-downtime. Safe rolls need readiness probes (Module 11).

## Services & DNS (Module 5)

```bash
kubectl get svc                       # types, cluster IPs, ports
kubectl get endpoints SVC             # live pod IPs behind a Service (#1 debug)
kubectl expose deploy/web --port=80 --target-port=80   # quick ClusterIP
kubectl port-forward svc/web 8080:80  # reach any Service from your laptop
# DNS: web | web.<ns> | web.<ns>.svc.cluster.local
```

Types: ClusterIP (in-cluster) · NodePort (node port) · LoadBalancer (cloud). Empty endpoints = selector/label mismatch.

## Config: ConfigMaps & Secrets (Module 6)

```bash
kubectl create configmap NAME --from-literal=K=V --from-file=app.conf
kubectl create secret generic NAME --from-literal=API_KEY=s3cr3t
kubectl get cm,secret ; kubectl describe cm NAME
kubectl get secret NAME -o jsonpath='{.data.KEY}' | base64 -d   # decode
kubectl rollout restart deploy/NAME   # REQUIRED for env CM/Secret changes to apply
```

Inject as env (`valueFrom`/`envFrom`, cached at start) or as files (volume mount, updates live). Secrets are base64, not encrypted.

## Storage: PVC/PV, StatefulSets (Module 7)

```bash
kubectl get storageclass            # provisioners (kind: 'standard' default)
kubectl get pvc                     # claims (want STATUS=Bound)
kubectl get pv                      # the actual volumes
kubectl describe pvc NAME           # why Pending
kubectl get statefulset ; kubectl get pods -l app=web-sts   # web-0, web-1
kubectl delete pvc www-web-0 www-web-1   # STS leaves PVCs behind on delete
```

PVC (request) → StorageClass provisions → PV (disk). Data on a PVC survives pod deletion. STS = stable names + PVC per pod.

## Jobs & CronJobs (Module 8)

```bash
kubectl get jobs                          # COMPLETIONS (e.g. 1/1)
kubectl logs job/NAME
kubectl get cronjobs                      # SCHEDULE, LAST SCHEDULE
kubectl create job --from=cronjob/NAME manual-run   # trigger now
kubectl patch cronjob NAME -p '{"spec":{"suspend":true}}'   # pause
```

Job = run-to-completion (restartPolicy Never/OnFailure). backoffLimit=retries, ttlSecondsAfterFinished=auto-clean. CronJob = scheduled Job; concurrencyPolicy Forbid/Replace avoids overlap.

## Scaling: requests/limits & HPA (Module 9)

```bash
# metrics-server (kind needs the insecure-tls patch):
kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml
kubectl patch -n kube-system deploy metrics-server --type=json \
  -p '[{"op":"add","path":"/spec/template/spec/containers/0/args/-","value":"--kubelet-insecure-tls"}]'

kubectl top nodes ; kubectl top pods
kubectl autoscale deploy/NAME --cpu-percent=50 --min=1 --max=5
kubectl get hpa            # TARGETS = current%/target%
```

requests = scheduler reserves + HPA baseline; limits = hard cap (CPU throttle, mem OOMKill). HPA needs metrics-server + CPU requests.

## Debugging loop (Module 10)

```bash
kubectl get pods [-o wide]                  # 1. state
kubectl describe pod NAME                    # 2. EVENTS (the 'why')
kubectl logs NAME [--previous] [-c CONT]     # 3. app output (crashed = --previous)
kubectl get events --sort-by=.lastTimestamp  # 4. cluster timeline
kubectl exec -it NAME -- sh                   # 5. inside a running container
kubectl debug -it NAME --image=busybox --target=CONT   # no shell in image
```

Pending = scheduling (describe). CrashLoopBackOff = app (logs). ImagePullBackOff = image/registry (describe).

## Health probes (Module 11)

```yaml
startupProbe:   { httpGet: {path: /, port: 80}, failureThreshold: 30, periodSeconds: 2 }
readinessProbe: { httpGet: {path: /, port: 80}, periodSeconds: 5, failureThreshold: 2 }
livenessProbe:  { httpGet: {path: /, port: 80}, periodSeconds: 5, failureThreshold: 3 }
# mechanisms: httpGet (2xx/3xx ok) | tcpSocket {port} | exec {command}
```

liveness = restart wedged container; readiness = gate traffic (Endpoints); startup = protect slow boot. Don't put dependency checks in liveness.

## Security & RBAC (Module 12)

```bash
kubectl get ns
kubectl config set-context --current --namespace=NS    # change default ns
kubectl get sa,role,rolebinding -n NS
kubectl auth can-i VERB RESOURCE -n NS                  # as you
kubectl auth can-i delete pods --as=system:serviceaccount:NS:SA -n NS
kubectl auth can-i --list --as=system:serviceaccount:NS:SA -n NS
kubectl create role NAME --verb=get,list --resource=pods -n NS --dry-run=client -o yaml
```

Role (namespaced) + RoleBinding → subject. ClusterRole/Binding = cluster-wide. Least privilege; every pod runs as a ServiceAccount.

## Observability (Module 13)

```bash
kubectl top nodes ; kubectl top pods [-A] [--sort-by=cpu]
kubectl get events --sort-by=.lastTimestamp [-A]
kubectl describe node NODE | grep -A8 "Allocated resources"
kubectl get pods -A --field-selector=status.phase=Pending
```

Built-ins show CURRENT state only (top/events/logs are ephemeral). Production adds Prometheus (metrics history) + Grafana (dashboards) + Loki/ELK (logs).

## Capstone: multi-tier with Kustomize (Module 14)

```bash
kubectl apply -k DIR/                  # deploy a whole app bundle
kubectl delete -k DIR/                 # one-command teardown
kubectl get all,pvc,cm,secret,hpa -n NS
kubectl wait --for=condition=ready pod --all -n NS --timeout=180s
kubectl port-forward -n NS svc/frontend 8080:80
```

Combine: Namespace + Secret + ConfigMap + StatefulSet(DB) + Deployments + Services + probes + HPA. initContainer (`nc -z db 5432`) enforces start ordering.

<!-- More sections appended as modules are completed. -->
