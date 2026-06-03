# Kubernetes Tutoring Curriculum — Modules 1–14 Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Author the remaining 14 hands-on Kubernetes lesson modules (1–14), each a self-contained markdown lab that takes a Docker-literate learner to work-ready, verified by its Practice lab running green on a real local cluster.

**Architecture:** Each module is one markdown file following the fixed 8-section format, backed by reusable YAML in `manifests/NN-<module>/`. Modules are authored **one at a time, on demand** (spec pacing rule) — this plan pre-defines each module's learning outcomes, exact manifests, and lab acceptance check (the "test"), while prose/diagrams are written at execution time. A module is "done" only when its Practice lab runs green on the learner's `kind-k8s-course` cluster and the README tracker + cheatsheet are updated.

**Tech Stack:** Kubernetes (kind cluster `k8s-course`), `kubectl`, YAML manifests, mermaid diagrams, Markdown. metrics-server for scaling/observability modules.

---

## How this plan reconciles the writing-plans skill with the spec's pacing rule

The writing-plans skill normally demands fully pre-written content per step. The spec (`docs/superpowers/specs/2026-06-03-k8s-tutoring-curriculum-design.md`, lines 97–98 and 105) forbids writing module content upfront — lessons must stay fresh and adapt to where the learner struggles.

Resolution:

- **Pre-defined now (the testable contract):** each module's learning outcomes, the exact manifest files to create, and a concrete **lab acceptance check** — real `kubectl` commands plus the output that proves the lab worked. This is the module's "test", written before its content.
- **Authored on demand (at execution):** the prose for the 8 sections, the mermaid diagram, and the final wording of the lab steps — written when the learner says *"let's do Module N"* / *"next"*, then taught live in chat.
- **The verification gate ("GREEN"):** the learner runs the Practice lab on `kind-k8s-course` and pastes output; the module is not done until the acceptance check passes live.

So "test-first" here = define outcomes + manifests + acceptance, *then* write the lesson, *then* prove it green on a real cluster.

## What already exists (do not recreate)

- `README.md` — course home + progress tracker (Module 0 = 🔧, Modules 1–14 = ⬜).
- `00-setup.md` — Module 0, complete.
- `cheatsheet.md` — living master quick-ref (Contexts + Pods-intro sections done).
- `manifests/README.md` — manifests dir placeholder.
- Cluster: `kind` cluster named `k8s-course`, context `kind-k8s-course`.

## Conventions (defined once — DRY; every module task obeys these)

### C1. Module file shape (8 sections, in order — from spec lines 48–60)

1. `## 🎯 What you'll learn` — 3–5 outcome bullets
2. `## 🐳 Docker bridge` — map to existing Docker knowledge
3. `## 🧠 Concept` — plain-language explanation
4. `## 📊 Diagram` — a mermaid block (`graph` / `flowchart` / `sequenceDiagram`)
5. `## 📋 Cheatsheet` — copy-paste `kubectl` + YAML daily drivers
6. `## 🧪 Practice` — numbered step-by-step lab on the learner's cluster
7. `## ✅ Check yourself` — outcome checkboxes + one small challenge
8. `## 🐛 Common gotchas` — real-world traps

Match the voice/format of `00-setup.md` exactly (emoji headers, `bash` code fences, blockquote tips, comparison tables where useful).

### C2. Manifest layout & naming

- One folder per module: `manifests/NN-<module-slug>/` (e.g. `manifests/03-deployments/`).
- Lowercase, hyphenated, purpose-named files: `deployment.yaml`, `clusterip.yaml`, `broken-crashloop.yaml`.
- Every manifest must `kubectl apply -f` cleanly against `kind-k8s-course` with no edits.
- Reuse the small `nginx`/`hashicorp/http-echo`/`busybox` images where possible to keep pulls fast.

### C3. Lab verification gate (the "test" — non-negotiable)

A module is GREEN only when, on context `kind-k8s-course`:
- every manifest in its folder applies cleanly, **and**
- the module's **Lab acceptance check** commands produce the stated output (learner pastes it in chat), **and**
- `kubectl delete -f manifests/NN-<module-slug>/` (or `kubectl delete ns <lab-ns>`) cleans up without orphans.

Use `superpowers:verification-before-completion` before claiming a module done — evidence (pasted output) before assertion.

### C4. README progress tracker update

In `README.md`, flip that module's row in the Progress tracker table from `⬜` to `✅` (use `🔧` while in progress). Do not touch other rows.

### C5. Cheatsheet growth

Append a new `## <Topic> (Module N)` section to `cheatsheet.md` with only that module's daily-driver commands, mirroring the existing terse style. Replace the trailing `<!-- More sections appended... -->` comment position (keep it last).

### C6. Commit format (from git-workflow rule; attribution disabled globally)

```bash
git add <module>.md manifests/NN-<slug>/ README.md cheatsheet.md
git commit -m "feat: add Module N — <title>"
```

One commit per completed module (after the lab is green). Commit only when the learner has cleared **Check yourself**.

---

## The Module Authoring Loop (every Task 1–13 runs exactly these steps)

Each module task below provides the *variable* content (outcomes, manifests, acceptance check, gotchas, cheatsheet additions). The *procedure* is identical and defined here once. When executing a module task, create one TodoWrite item per step:

- [ ] **Step A — Mark in progress.** Set the module's README row to `🔧`.
- [ ] **Step B — Write the manifests first (the test fixtures).** Create every file listed in the module's **Manifests** subsection, with complete YAML. Run `kubectl apply --dry-run=client -f manifests/NN-<slug>/` — expect `created (dry run)` for each, no schema errors.
- [ ] **Step C — Write the acceptance check as the lab's success criteria.** Encode the module's **Lab acceptance check** commands + expected output verbatim into the `🧪 Practice` and `✅ Check yourself` sections — define "passing" before writing the teaching prose.
- [ ] **Step D — Author the lesson.** Write `NN-<slug>.md` with all 8 sections per C1, using the outcomes, Docker-bridge hook, diagram concept, and gotchas from the module's brief. No placeholders.
- [ ] **Step E — Run the lab live (GREEN gate).** Learner runs the Practice lab on `kind-k8s-course`, pastes output. Confirm it matches the acceptance check (C3). Debug live with `superpowers:systematic-debugging` if it fails; fix the lesson/manifests, not the expectation.
- [ ] **Step F — Clean up.** `kubectl delete -f manifests/NN-<slug>/` (or delete the lab namespace); confirm no leftover pods (`kubectl get pods`).
- [ ] **Step G — Update tracker + cheatsheet.** Apply C4 and C5.
- [ ] **Step H — Commit.** Per C6.

> **Pacing rule (spec):** do **not** batch-author modules. Run this loop for one module only when the learner asks for it. The briefs below are the standing contract; the prose is written at that moment.

---

## Task 1: Module 1 — Mental model

**Files:**
- Create: `01-mental-model.md`
- Create: `manifests/01-mental-model/pod.yaml`
- Update: `README.md` (row 1), `cheatsheet.md`

**Learning outcomes (🎯):**
- Explain the reconcile/control loop: controllers drive **actual state → desired state**, continuously.
- Distinguish **declarative** (`kubectl apply -f`) from **imperative** (`kubectl run`/`create`).
- Read any object as `spec` (what you want) vs `status` (what is).
- Name the core control-plane parts (API server, scheduler, controllers, etcd) and that everything talks only to the API server.
- Discover the API surface with `kubectl api-resources` / `kubectl explain`.

**Docker bridge (🐳):** `docker run` = you imperatively start one container *now*; `kubectl apply` = you declare desired cluster state and a controller keeps it true forever. Docker has no reconcile loop.

**Diagram (📊):** reconcile loop — desired state (YAML) → API server → controller compares spec vs status → acts on nodes → status converges. (A `flowchart`/loop, distinct from Module 0's component diagram.)

**Manifests:**
- `manifests/01-mental-model/pod.yaml` — a single `nginx` pod named `mental-model` with a label `app: mental-model`.

**Lab acceptance check (the test):**
```bash
kubectl apply -f manifests/01-mental-model/pod.yaml      # pod/mental-model created
kubectl get pod mental-model -o jsonpath='{.spec.containers[0].image}{"\n"}'   # -> nginx (desired)
kubectl get pod mental-model -o jsonpath='{.status.phase}{"\n"}'                # -> Running (actual)
kubectl api-resources | findstr /I "deployments services"   # learner can list known kinds
kubectl explain pod.spec.containers | more                   # field docs appear
```
Pass = learner can point at `spec.containers[0].image` as *desired* and `status.phase` as *actual* and say why a bare pod won't self-heal (teases Module 3).

**Cheatsheet additions (📋):** `kubectl apply -f` vs `kubectl run`; `kubectl get X -o yaml`; `kubectl explain`; `kubectl api-resources`.

**Gotchas (🐛):** `kubectl edit` changes to controller-managed objects get reverted; imperative `run` vs declarative `apply` causes config drift; bare pods are not rescheduled if their node dies.

---

## Task 2: Module 2 — Pods

**Files:**
- Create: `02-pods.md`
- Create: `manifests/02-pods/pod.yaml`, `manifests/02-pods/multi-container.yaml`
- Update: `README.md` (row 2), `cheatsheet.md`

**Learning outcomes (🎯):**
- Define a Pod: the smallest deployable unit; one or more containers sharing network (localhost) and volumes.
- Explain why you rarely create bare Pods directly (no self-healing — controllers do that).
- Use the sidecar/multi-container pattern with a shared `emptyDir`.
- Read pod lifecycle phases (Pending → Running → Succeeded/Failed) and `restartPolicy`.
- Get logs and exec into a specific container (`-c`).

**Docker bridge (🐳):** a Pod ≈ several containers wired to share one network namespace + volumes (like `docker run --network container:...` plus a shared mount), but declared and managed together.

**Diagram (📊):** a Pod boundary containing 2 containers sharing `localhost` + an `emptyDir` volume.

**Manifests:**
- `manifests/02-pods/pod.yaml` — single `nginx` pod, label `app: web`.
- `manifests/02-pods/multi-container.yaml` — pod with a `nginx` container and a `busybox` sidecar writing to a shared `emptyDir` mounted in both.

**Lab acceptance check (the test):**
```bash
kubectl apply -f manifests/02-pods/pod.yaml
kubectl get pod web -o wide                      # STATUS Running, shows pod IP + node
kubectl logs web                                 # nginx startup logs
kubectl exec -it web -- nginx -v                 # exec works
kubectl apply -f manifests/02-pods/multi-container.yaml
kubectl logs <multi-pod> -c <sidecar>            # sidecar's logs only
kubectl exec <multi-pod> -c nginx -- cat /shared/hello   # reads file the sidecar wrote -> shared volume proven
```
Pass = learner reads logs/exec per-container and shows the two containers share a volume.

**Cheatsheet additions (📋):** `kubectl logs [-c]`, `kubectl exec -it -- ...`, `kubectl get pod -o wide`, `kubectl describe pod`.

**Gotchas (🐛):** `CrashLoopBackOff` (app exits immediately); `ImagePullBackOff`/`ErrImagePull` (bad image/tag); bare pods not rescheduled on node loss; `restartPolicy` only `Always`/`OnFailure`/`Never`; `kubectl logs` needs `-c` when multi-container.

---

## Task 3: Module 3 — Deployments

**Files:**
- Create: `03-deployments.md`
- Create: `manifests/03-deployments/deployment.yaml`
- Update: `README.md` (row 3), `cheatsheet.md`

**Learning outcomes (🎯):**
- Explain the chain Deployment → ReplicaSet → Pods and what each owns.
- Observe **self-healing**: delete a pod, watch it recreated.
- Scale with `kubectl scale` / by editing `replicas`.
- Roll out a new image and watch `rollout status`; inspect `rollout history`.
- Roll back with `rollout undo`.

**Docker bridge (🐳):** like `docker compose up --scale web=3`, but the controller *keeps* it at 3 and self-heals, declaratively.

**Diagram (📊):** Deployment owns ReplicaSet(s) owns Pods; rollout creates a new ReplicaSet and shifts pods over.

**Manifests:**
- `manifests/03-deployments/deployment.yaml` — `nginx:1.25` Deployment, `replicas: 3`, matching `selector`/`template` labels `app: web`.

**Lab acceptance check (the test):**
```bash
kubectl apply -f manifests/03-deployments/deployment.yaml
kubectl get deploy,rs,pods -l app=web            # 1 deploy, 1 rs, 3 pods Running
kubectl delete pod <one-web-pod>; kubectl get pods -l app=web   # back to 3 -> self-heal
kubectl scale deploy/web --replicas=5; kubectl get pods -l app=web   # 5 pods
kubectl set image deploy/web nginx=nginx:1.26
kubectl rollout status deploy/web                # "successfully rolled out"
kubectl rollout history deploy/web               # >=2 revisions
kubectl rollout undo deploy/web; kubectl rollout status deploy/web   # back to 1.25
```
Pass = learner triggers self-heal, scale, rollout, and rollback and reads the new/old ReplicaSets.

**Cheatsheet additions (📋):** `kubectl scale`, `kubectl set image`, `kubectl rollout status|history|undo`, `kubectl get deploy,rs,pods -l`.

**Gotchas (🐛):** `selector` must match `template.labels` or apply fails; a bad image leaves the rollout stuck (pods `ImagePullBackOff`) — `rollout undo` to recover; old ReplicaSets kept for history (`revisionHistoryLimit`).

---

## Task 4: Module 4 — Deploy strategies

**Files:**
- Create: `04-deploy-strategies.md`
- Create: `manifests/04-deploy-strategies/rolling.yaml`, `recreate.yaml`, `blue.yaml`, `green.yaml`, `bluegreen-svc.yaml`, `canary-stable.yaml`, `canary-canary.yaml`
- Update: `README.md` (row 4), `cheatsheet.md`

**Learning outcomes (🎯):**
- Contrast `RollingUpdate` (zero-downtime, gradual) vs `Recreate` (downtime, clean swap).
- Tune `maxSurge` / `maxUnavailable` and predict pod counts during a roll.
- Do a **blue-green** switch by flipping a Service's selector.
- Do a **canary** by running a small second Deployment behind the same Service.

**Docker bridge (🐳):** Compose just restarts containers; k8s gives you controlled, observable rollout strategies with surge/unavailable budgets.

**Diagram (📊):** rolling vs recreate timelines; blue-green Service selector flip; canary traffic split by replica ratio.

**Manifests:**
- `rolling.yaml` — Deployment, `strategy.type: RollingUpdate`, `maxSurge: 1`, `maxUnavailable: 0`.
- `recreate.yaml` — same app, `strategy.type: Recreate`.
- `blue.yaml`/`green.yaml` — two Deployments, labels `version: blue|green` (use `hashicorp/http-echo` with distinct `-text`).
- `bluegreen-svc.yaml` — Service selecting `version: blue` initially.
- `canary-stable.yaml` (replicas 3) + `canary-canary.yaml` (replicas 1), shared label `app: echo` so one Service spans both.

**Lab acceptance check (the test):**
```bash
kubectl apply -f manifests/04-deploy-strategies/rolling.yaml
kubectl set image deploy/rolling echo=hashicorp/http-echo:1.0 ; kubectl get pods -w   # surge=1, unavailable=0 visible
kubectl apply -f manifests/04-deploy-strategies/recreate.yaml
kubectl set image deploy/recreate ... ; kubectl get pods -w     # all old terminate THEN new start (downtime)
# blue-green
kubectl apply -f blue.yaml -f green.yaml -f bluegreen-svc.yaml
kubectl patch svc bluegreen -p '{"spec":{"selector":{"version":"green"}}}'   # traffic flips to green
# canary
kubectl apply -f canary-stable.yaml -f canary-canary.yaml      # ~1 in 4 requests hit canary
```
Pass = learner observes the surge/unavailable behavior, the blue→green flip, and canary split.

**Cheatsheet additions (📋):** `strategy.rollingUpdate.maxSurge/maxUnavailable`, `kubectl patch svc ... selector`, `kubectl get pods -w`.

**Gotchas (🐛):** `Recreate` = real downtime; `maxUnavailable: 0` requires room to surge; without a readiness probe (Module 11) a roll can send traffic to not-ready pods; canary "split" is only replica-ratio, not weighted routing.

---

## Task 5: Module 5 — Services

**Files:**
- Create: `05-services.md`
- Create: `manifests/05-services/deployment.yaml`, `clusterip.yaml`, `nodeport.yaml`
- Update: `README.md` (row 5), `cheatsheet.md`

**Learning outcomes (🎯):**
- Explain why pods need a stable front: pod IPs are ephemeral; a Service gives a stable virtual IP + DNS.
- Compare `ClusterIP` (in-cluster), `NodePort` (external via node port), `LoadBalancer` (cloud).
- See how `selector` → `Endpoints` (the live pod set).
- Resolve a Service by DNS: `name.namespace.svc.cluster.local`.

**Docker bridge (🐳):** like a Docker network alias + DNS, but the membership auto-updates as pods are created/destroyed.

**Diagram (📊):** Service (stable VIP) → Endpoints → set of Pods; client resolves Service DNS.

**Manifests:**
- `deployment.yaml` — `nginx` Deployment, replicas 3, label `app: web`.
- `clusterip.yaml` — Service type `ClusterIP`, selector `app: web`, port 80.
- `nodeport.yaml` — Service type `NodePort` for the same selector.

**Lab acceptance check (the test):**
```bash
kubectl apply -f manifests/05-services/deployment.yaml -f manifests/05-services/clusterip.yaml
kubectl get svc web ; kubectl get endpoints web      # 3 endpoint IPs listed
kubectl run tmp --rm -it --image=busybox --restart=Never -- wget -qO- http://web   # HTML returned via Service DNS
kubectl apply -f manifests/05-services/nodeport.yaml
kubectl get svc web-nodeport                          # shows 80:3xxxx/TCP
kubectl port-forward svc/web 8080:80                  # then curl localhost:8080 works
```
Pass = learner curls the app through the Service DNS name and sees Endpoints track the 3 pods.

**Cheatsheet additions (📋):** `kubectl get svc,endpoints`, `kubectl expose deploy ...`, `kubectl port-forward svc/...`, Service DNS format.

**Gotchas (🐛):** selector typo → empty Endpoints → connection refused; `LoadBalancer` stays `<pending>` on kind (use NodePort or `port-forward`); calling the wrong namespace's Service (use FQDN); targetPort vs port confusion.

---

## Task 6: Module 6 — Config (ConfigMaps & Secrets)

**Files:**
- Create: `06-config.md`
- Create: `manifests/06-config/configmap.yaml`, `secret.yaml`, `deployment-env.yaml`, `deployment-volume.yaml`
- Update: `README.md` (row 6), `cheatsheet.md`

**Learning outcomes (🎯):**
- Store non-secret config in a ConfigMap; sensitive values in a Secret (base64, *not* encryption at rest by default).
- Inject config two ways: as **env vars** and as **mounted files**.
- Know that env injected from a CM/Secret needs a pod restart to change; mounted files update eventually.
- Avoid committing Secrets to git.

**Docker bridge (🐳):** like `docker run -e KEY=val` and `-v ./conf:/etc/app`, but the values live as first-class cluster objects you can share across pods.

**Diagram (📊):** ConfigMap/Secret → injected into a Pod as env and as a mounted volume.

**Manifests:**
- `configmap.yaml` — keys e.g. `APP_COLOR=blue`, plus a file-style key `app.conf`.
- `secret.yaml` — `Opaque`, e.g. `API_KEY` (note: base64).
- `deployment-env.yaml` — consumes CM+Secret via `envFrom`/`valueFrom`.
- `deployment-volume.yaml` — mounts the CM as files under `/etc/app`.

**Lab acceptance check (the test):**
```bash
kubectl apply -f manifests/06-config/configmap.yaml -f manifests/06-config/secret.yaml
kubectl apply -f manifests/06-config/deployment-env.yaml
kubectl exec deploy/app-env -- printenv APP_COLOR API_KEY     # values present
kubectl apply -f manifests/06-config/deployment-volume.yaml
kubectl exec deploy/app-vol -- cat /etc/app/app.conf          # file content from CM
# change CM value, re-apply, show env pod still old until:
kubectl rollout restart deploy/app-env ; kubectl exec deploy/app-env -- printenv APP_COLOR   # now new
```
Pass = learner reads config from both env and file, and demonstrates env needs a `rollout restart`.

**Cheatsheet additions (📋):** `kubectl create configmap/secret ... --dry-run -o yaml`, `envFrom`, `valueFrom.secretKeyRef`, `kubectl rollout restart`.

**Gotchas (🐛):** Secrets are base64-encoded, **not** encrypted — don't treat as safe; env from CM/Secret won't change live (need restart); volume-mounted CM updates lag (kubelet sync period); never `git add` real Secrets.

---

## Task 7: Module 7 — Storage

**Files:**
- Create: `07-storage.md`
- Create: `manifests/07-storage/pvc.yaml`, `pod-pvc.yaml`, `statefulset.yaml`
- Update: `README.md` (row 7), `cheatsheet.md`

**Learning outcomes (🎯):**
- Distinguish ephemeral (`emptyDir`, container fs) from persistent storage.
- Explain PVC (request) ↔ PV (resource) binding and dynamic provisioning via StorageClass.
- Show data survives pod deletion when backed by a PVC.
- Intro StatefulSets: stable network identity + a PVC **per** pod.

**Docker bridge (🐳):** like `docker volume` + named volumes, but split into a *request* (PVC) and a *provisioned resource* (PV), often auto-created by a StorageClass.

**Diagram (📊):** PVC → (StorageClass dynamic provision) → PV → mounted into Pod; StatefulSet pods `web-0/1/2` each with own PVC.

**Manifests:**
- `pvc.yaml` — PVC, `ReadWriteOnce`, ~100Mi (kind's default StorageClass `standard` provisions it).
- `pod-pvc.yaml` — pod mounting the PVC, writes a file.
- `statefulset.yaml` — 2-replica StatefulSet with `volumeClaimTemplates`, headless Service.

**Lab acceptance check (the test):**
```bash
kubectl apply -f manifests/07-storage/pvc.yaml ; kubectl get pvc       # STATUS Bound
kubectl apply -f manifests/07-storage/pod-pvc.yaml
kubectl exec writer -- sh -c 'echo persisted > /data/file && cat /data/file'
kubectl delete pod writer ; kubectl apply -f manifests/07-storage/pod-pvc.yaml
kubectl exec writer -- cat /data/file                                   # "persisted" survives
kubectl apply -f manifests/07-storage/statefulset.yaml
kubectl get pods -l app=web ; kubectl get pvc                          # web-0, web-1 + one PVC each
```
Pass = learner shows data persisting across pod deletion and stable per-pod identity/PVCs.

**Cheatsheet additions (📋):** `kubectl get pv,pvc,sc`, `accessModes`, `volumeClaimTemplates`, headless Service (`clusterIP: None`).

**Gotchas (🐛):** PVC stuck `Pending` = no StorageClass/provisioner; `ReadWriteOnce` = one node at a time; deleting a StatefulSet does **not** delete its PVCs (data lingers/costs); `emptyDir` dies with the pod.

---

## Task 8: Module 8 — Jobs & CronJobs

**Files:**
- Create: `08-jobs-cronjobs.md`
- Create: `manifests/08-jobs-cronjobs/job.yaml`, `cronjob.yaml`
- Update: `README.md` (row 8), `cheatsheet.md`

**Learning outcomes (🎯):**
- Run a one-off task to completion with a Job; read `completions`/`parallelism`.
- Control retries with `backoffLimit` and auto-cleanup with `ttlSecondsAfterFinished`.
- Schedule recurring work with a CronJob; tune `schedule`, `concurrencyPolicy`, history limits.

**Docker bridge (🐳):** like `docker run` a batch container that exits, or a host `cron` — but cluster-managed, retried, and history-tracked.

**Diagram (📊):** CronJob → (on schedule) creates Job → creates Pod(s) → run to completion.

**Manifests:**
- `job.yaml` — `busybox` Job that prints + exits 0, `backoffLimit: 2`, `ttlSecondsAfterFinished: 120`.
- `cronjob.yaml` — `*/1 * * * *` `busybox` CronJob, `successfulJobsHistoryLimit: 3`.

**Lab acceptance check (the test):**
```bash
kubectl apply -f manifests/08-jobs-cronjobs/job.yaml
kubectl get jobs                                  # COMPLETIONS 1/1
kubectl logs job/<name>                           # task output
kubectl apply -f manifests/08-jobs-cronjobs/cronjob.yaml
kubectl get cronjob ; sleep ~70s ; kubectl get jobs    # a new job has appeared
```
Pass = learner sees the Job complete and the CronJob spawn at least one scheduled Job.

**Cheatsheet additions (📋):** `kubectl get jobs,cronjobs`, `kubectl logs job/...`, `kubectl create job --from=cronjob/...`, schedule/backoffLimit/ttl fields.

**Gotchas (🐛):** failed pods retried up to `backoffLimit` then Job fails; completed Jobs/pods linger until cleaned (`ttlSecondsAfterFinished`); CronJob uses controller TZ unless `timeZone` set; overlapping runs unless `concurrencyPolicy: Forbid`.

---

## Task 9: Module 9 — Scaling

**Files:**
- Create: `09-scaling.md`
- Create: `manifests/09-scaling/deployment-resources.yaml`, `hpa.yaml`, `load.yaml`
- Update: `README.md` (row 9), `cheatsheet.md`
- Prereq: metrics-server installed (see check below).

**Learning outcomes (🎯):**
- Scale manually (`replicas`) vs automatically (HPA).
- Set resource `requests` (scheduling) vs `limits` (cap/throttle); see how requests place pods.
- Install metrics-server (on kind needs `--kubelet-insecure-tls`) and read `kubectl top`.
- Create an HPA on CPU and watch it scale under load.

**Docker bridge (🐳):** Docker has no built-in autoscaler; HPA watches metrics and changes `replicas` for you.

**Diagram (📊):** metrics-server → HPA controller → adjusts Deployment `replicas` based on CPU vs target.

**Manifests:**
- `deployment-resources.yaml` — app with `resources.requests.cpu` + `limits.cpu/memory` (use a CPU-burnable image, e.g. `hpa-example`/`php-apache` pattern or busybox loop).
- `hpa.yaml` — HPA target the deployment, `minReplicas: 1`, `maxReplicas: 5`, CPU target 50%.
- `load.yaml` — a busybox pod that hammers the service in a loop to generate CPU load.

**Lab acceptance check (the test):**
```bash
# metrics-server (kind): apply components then patch insecure-tls
kubectl top nodes                                 # returns CPU/MEM (proves metrics-server up)
kubectl apply -f manifests/09-scaling/deployment-resources.yaml -f manifests/09-scaling/hpa.yaml
kubectl get hpa                                   # shows TARGETS e.g. 0%/50%, REPLICAS 1
kubectl apply -f manifests/09-scaling/load.yaml
kubectl get hpa -w                                # TARGETS rises, REPLICAS scales up toward 5
```
Pass = `kubectl top` works and the HPA scales replicas up under load (and back down after).

**Cheatsheet additions (📋):** `kubectl top nodes/pods`, `kubectl autoscale deploy ...`, `kubectl get hpa`, requests/limits block, metrics-server kind patch.

**Gotchas (🐛):** HPA needs metrics-server **and** CPU `requests` set, else `<unknown>` targets; over-limit memory → `OOMKilled`; kind's metrics-server needs `--kubelet-insecure-tls`; scale-down is deliberately slow (stabilization window).

---

## Task 10: Module 10 — Debugging (work-readiness core)

**Files:**
- Create: `10-debugging.md`
- Create: `manifests/10-debugging/broken-image.yaml`, `broken-crashloop.yaml`, `broken-pending.yaml`
- Update: `README.md` (row 10), `cheatsheet.md`

**Learning outcomes (🎯):**
- Run the debugging loop: `get` → `describe` → `logs` → events → `exec`.
- Map symptoms to causes: `ImagePullBackOff`, `CrashLoopBackOff`, `Pending` (unschedulable).
- Read crashed-container logs with `--previous`.
- Use `kubectl describe` events and `kubectl get events --sort-by` to find the *why*.

**Docker bridge (🐳):** beyond `docker logs`/`docker inspect` — k8s adds cluster-level **events** and controller status that explain scheduling/restart decisions.

**Diagram (📊):** decision flow: symptom → which command → likely cause → fix.

**Manifests (deliberately broken):**
- `broken-image.yaml` — pod with a non-existent image tag (→ `ImagePullBackOff`).
- `broken-crashloop.yaml` — pod whose command exits 1 immediately (→ `CrashLoopBackOff`).
- `broken-pending.yaml` — pod requesting impossible resources (e.g. `cpu: "1000"`) → `Pending`/Unschedulable.

**Lab acceptance check (the test):**
```bash
kubectl apply -f manifests/10-debugging/broken-image.yaml
kubectl describe pod <p> | findstr /I "Failed Back-off"      # event explains ImagePullBackOff
kubectl apply -f manifests/10-debugging/broken-crashloop.yaml
kubectl logs <p> --previous                                  # logs from the crashed instance
kubectl apply -f manifests/10-debugging/broken-pending.yaml
kubectl describe pod <p> | findstr /I "FailedScheduling Insufficient"   # why it can't schedule
```
Pass = learner independently diagnoses all three from `describe`/`logs`/events and states the fix.

**Cheatsheet additions (📋):** `kubectl describe pod`, `kubectl logs --previous`, `kubectl get events --sort-by=.lastTimestamp`, `kubectl debug` (ephemeral container), `kubectl get pod -o wide`.

**Gotchas (🐛):** logs of a crashed container need `--previous`; events expire (~1h) — look fast; `Pending` is almost always scheduling (resources/taints), shown in `describe`; `ImagePullBackOff` ≠ `CrashLoopBackOff` (pull vs run failure).

---

## Task 11: Module 11 — Health probes

**Files:**
- Create: `11-health-probes.md`
- Create: `manifests/11-health-probes/deployment-probes.yaml`, `svc.yaml`
- Update: `README.md` (row 11), `cheatsheet.md`

**Learning outcomes (🎯):**
- Distinguish **liveness** (restart on fail), **readiness** (gate traffic), **startup** (protect slow boot).
- Configure `httpGet` / `tcpSocket` / `exec` probes and tune `initialDelaySeconds`/`periodSeconds`/thresholds.
- Show readiness controls Service Endpoints membership; liveness drives restarts.

**Docker bridge (🐳):** richer than Docker's `HEALTHCHECK` — readiness actually gates whether the Service sends traffic, and liveness triggers managed restarts.

**Diagram (📊):** probe outcomes → readiness toggles Endpoints in/out; liveness fail → kubelet restarts container.

**Manifests:**
- `deployment-probes.yaml` — `nginx` (or `http-echo`) with `readinessProbe` + `livenessProbe` (httpGet `/`), plus a `startupProbe`.
- `svc.yaml` — ClusterIP selecting the deployment.

**Lab acceptance check (the test):**
```bash
kubectl apply -f manifests/11-health-probes/deployment-probes.yaml -f manifests/11-health-probes/svc.yaml
kubectl get pods                                   # READY 1/1 only after readiness passes
kubectl get endpoints <svc>                        # pod IP appears only when ready
# break liveness (exec to remove the probed path / point probe at bad path) and observe:
kubectl get pods -w                                # RESTARTS increments
```
Pass = learner shows a pod entering Endpoints only when ready, and liveness failures incrementing RESTARTS.

**Cheatsheet additions (📋):** `readinessProbe`/`livenessProbe`/`startupProbe` blocks (httpGet/tcpSocket/exec), threshold fields, `kubectl get pods` READY column meaning.

**Gotchas (🐛):** over-aggressive liveness → restart loops on a healthy-but-slow app (use startupProbe); failing readiness silently removes traffic (no errors, just no Endpoints); wrong probe path/port; missing `initialDelaySeconds` causes early failures.

---

## Task 12: Module 12 — Security & RBAC

**Files:**
- Create: `12-security-rbac.md`
- Create: `manifests/12-security-rbac/namespace.yaml`, `serviceaccount.yaml`, `role.yaml`, `rolebinding.yaml`
- Update: `README.md` (row 12), `cheatsheet.md`

**Learning outcomes (🎯):**
- Use namespaces for isolation/scoping.
- Explain ServiceAccounts (pod identity) vs users.
- Build least-privilege access: Role (namespaced) + RoleBinding; contrast with ClusterRole/ClusterRoleBinding.
- Verify permissions with `kubectl auth can-i`.

**Docker bridge (🐳):** Docker has no built-in multi-tenant authz; k8s RBAC defines *who can do what* on *which resources*, per namespace.

**Diagram (📊):** Subject (ServiceAccount) ⟶ RoleBinding ⟶ Role (verbs × resources) within a Namespace.

**Manifests:**
- `namespace.yaml` — namespace `rbac-lab`.
- `serviceaccount.yaml` — SA `reader` in `rbac-lab`.
- `role.yaml` — Role allowing `get`/`list` on `pods` only.
- `rolebinding.yaml` — binds `reader` SA to the Role.

**Lab acceptance check (the test):**
```bash
kubectl apply -f manifests/12-security-rbac/   # ns, sa, role, rolebinding
kubectl auth can-i list pods --as=system:serviceaccount:rbac-lab:reader -n rbac-lab    # yes
kubectl auth can-i delete pods --as=system:serviceaccount:rbac-lab:reader -n rbac-lab  # no
kubectl auth can-i list pods --as=system:serviceaccount:rbac-lab:reader -n default     # no (namespaced)
```
Pass = learner demonstrates the SA can list pods only in `rbac-lab`, and cannot delete or read other namespaces.

**Cheatsheet additions (📋):** `kubectl auth can-i [--as=]`, Role/RoleBinding vs ClusterRole/ClusterRoleBinding, `kubectl create role/rolebinding ... --dry-run -o yaml`, `kubectl get sa,role,rolebinding -n`.

**Gotchas (🐛):** every pod gets the namespace `default` SA unless set; Role is namespaced, ClusterRole is cluster-wide — easy to over-grant; binding subject `kind`/`name`/`namespace` must match exactly; avoid `cluster-admin` bindings.

---

## Task 13: Module 13 — Observability

**Files:**
- Create: `13-observability.md`
- Create: `manifests/13-observability/load-sample.yaml` (optional reuse from Module 9)
- Update: `README.md` (row 13), `cheatsheet.md`
- Prereq: metrics-server (from Module 9).

**Learning outcomes (🎯):**
- Read live resource usage with `kubectl top nodes/pods`.
- Use events + `describe` for cluster-state visibility over time.
- Understand the gap k8s leaves: logs/metrics are ephemeral → need aggregation (Prometheus/Grafana, Loki) — conceptual intro, no full install.
- Identify resource-pressure signals (evictions, OOM, throttling).

**Docker bridge (🐳):** `docker stats` shows one host; `kubectl top` + events give a cluster-wide view, and production adds external metric/log stores.

**Diagram (📊):** sources (kubelet/cAdvisor → metrics-server; container stdout → logs) → consumers (`kubectl top`, dashboards) → why you add Prometheus/Grafana/Loki.

**Manifests:**
- Optional: reuse `manifests/09-scaling/load.yaml` to create something worth observing; or a small multi-pod deployment to `top`.

**Lab acceptance check (the test):**
```bash
kubectl top nodes                                  # CPU/MEM per node
kubectl top pods -A                                # per-pod usage
kubectl get events -A --sort-by=.lastTimestamp | more   # recent cluster activity
kubectl describe node <node> | findstr /I "Allocated Capacity"   # capacity vs allocated
```
Pass = learner reads node/pod usage and the events timeline, and can explain what Prometheus/Grafana would add.

**Cheatsheet additions (📋):** `kubectl top nodes/pods [-A]`, `kubectl get events --sort-by`, `kubectl describe node`, note on metrics-server vs Prometheus.

**Gotchas (🐛):** `kubectl top` needs metrics-server (else error); events are short-lived; pod logs vanish when the pod is deleted (hence log aggregation); `top` shows *current* usage, not historical trend.

---

## Task 14: Module 14 — Capstone (multi-tier app, end-to-end)

> Capstone is integrative, not a new concept. It runs an **extended** version of the Module Authoring Loop: it composes prior manifests into one app and the acceptance check exercises deploy → expose → config → storage → scale → debug → rollout/rollback together.

**Files:**
- Create: `14-capstone.md`
- Create: `manifests/14-capstone/` — `namespace.yaml`, `db-statefulset.yaml`, `db-svc.yaml` (headless), `backend-deployment.yaml` (+ probes, resources), `backend-svc.yaml`, `frontend-deployment.yaml` (+ probes), `frontend-svc.yaml` (NodePort), `configmap.yaml`, `secret.yaml`, `hpa.yaml`, and a `kustomization.yaml` to apply them together.
- Update: `README.md` (row 14), `cheatsheet.md`

**Learning outcomes (🎯) — demonstrate the success criteria from the spec:**
- Deploy a multi-tier app (frontend → backend → db) into its own namespace with one command.
- Expose it (Service/NodePort), wire config via ConfigMap+Secret, persist db via PVC.
- Scale it (HPA), gate traffic via readiness probes.
- Diagnose an induced failure from logs/events; roll a new version out and roll it back.

**Docker bridge (🐳):** this is your `docker-compose.yml` for a real app — but self-healing, scalable, observable, with rollouts/rollbacks.

**Diagram (📊):** full topology — Namespace containing frontend Deployment+Svc(NodePort) → backend Deployment+Svc → db StatefulSet+PVC+headless Svc, with ConfigMap/Secret/HPA attached.

**Lab acceptance check (the test):**
```bash
kubectl apply -k manifests/14-capstone/
kubectl get all,pvc,cm,secret,hpa -n capstone     # everything created
kubectl wait --for=condition=ready pod --all -n capstone --timeout=120s   # all Ready
# reach the app:
kubectl port-forward -n capstone svc/frontend 8080:80   # app responds in browser/curl
# scale + self-heal:
kubectl scale -n capstone deploy/backend --replicas=4 ; kubectl get pods -n capstone
# induce + diagnose a failure (bad image), then recover:
kubectl set image -n capstone deploy/backend app=bad:tag ; kubectl rollout status -n capstone deploy/backend   # fails
kubectl describe -n capstone pod <backend>        # diagnose ImagePullBackOff
kubectl rollout undo -n capstone deploy/backend ; kubectl rollout status -n capstone deploy/backend            # recovered
# cleanup
kubectl delete -k manifests/14-capstone/
```
Pass = the whole app comes up Ready, is reachable, scales, and the learner diagnoses + rolls back a broken deploy unaided.

**Cheatsheet additions (📋):** `kubectl apply -k`, `kubectl wait --for=condition=ready`, `kubectl get all -n`, end-to-end recovery flow.

**Gotchas (🐛):** start-order/dependencies (backend not ready until db is — readiness probes handle this); db Secret must exist before pods mount it; resource requests must fit the kind node; NodePort vs port-forward on local; cleaning up = delete the namespace to avoid orphaned PVCs.

---

## Self-review (run against the spec)

**1. Spec coverage:**
- 15 modules (spec table, lines 30–46): Module 0 already done; Modules 1–14 = Tasks 1–14. ✅
- Per-lesson 8-section format (lines 48–60): convention **C1** + every task's brief maps to all 8 sections. ✅
- Practice = live lab, chat = instructor (lines 61–62): Loop **Step E** + **C3** verification gate are live, paste-driven. ✅
- File structure (lines 64–87): module files `NN-*.md`, `cheatsheet.md` growth (**C5**), `manifests/` per-module (**C2**), README tracker (**C4**). ✅
- Progression loop (lines 89–95): request → write → live practice → mark done + commit → next = Loop Steps A–H. ✅
- Pacing rule "one at a time, never upfront" (lines 97–98, 105): enforced in **Loop** preface and the reconciliation section; this plan defines contracts, not pre-written lessons. ✅
- Out of scope (lines 100–105): no cert trivia, no cloud-managed clusters (kind local only), no upfront authoring. ✅
- Success criteria (lines 107–111): deploy/expose/scale/diagnose/rollout+rollback/config+storage+jobs all covered across Tasks 3–14, integrated in Task 14's acceptance check. ✅

**2. Placeholder scan:** every task has concrete outcomes, named manifest files, and a runnable acceptance check with expected output. Lesson *prose* is intentionally deferred per the spec's pacing rule (documented, not a placeholder). No "TODO/TBD/handle edge cases".

**3. Type/name consistency:** manifest folders use the `NN-<module-slug>` scheme matching README filenames throughout (`03-deployments`, `12-security-rbac`, etc.); the Authoring Loop step names (A–H) and conventions (C1–C6) are referenced consistently by every task; cluster context is `kind-k8s-course` everywhere.
