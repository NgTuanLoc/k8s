# Kubernetes Tutoring Curriculum — Design Spec

**Date:** 2026-06-03
**Owner:** tuanloc
**Status:** Approved

## Goal

A guided, hands-on Kubernetes learning course taking the learner from
"knows Docker, new to k8s" to **work-ready**: able to deploy, configure,
expose, scale, debug, secure, and operate applications on a real cluster.

## Learner profile

- **Level:** Comfortable with Docker/containers; new to Kubernetes.
- **Goal:** Job / work readiness (practical operation over cert trivia).
- **Practice env:** Needs setup help. Windows 11 Pro local cluster
  (Docker Desktop Kubernetes or kind/minikube — decided in Module 0).
- **Delivery:** Saved `.md` lesson files **plus** live, coached practice in chat.

## Approach

Hybrid curriculum: a short foundation arc (mental model + core objects,
bridged from Docker knowledge) followed by project-driven modules that
mirror real work (deploy → configure → expose → scale → self-heal → debug →
secure → observe). Cert-only trivia is skipped.

## Curriculum (15 modules)

| # | Module | Focus |
|---|--------|-------|
| 0  | Setup            | Windows local cluster, kubectl, verify |
| 1  | Mental model     | What k8s is, control loop, Docker → k8s bridge |
| 2  | Pods             | The atom; why you rarely create them directly |
| 3  | Deployments      | ReplicaSets, rollouts, rollbacks, self-healing |
| 4  | Deploy strategies| RollingUpdate vs Recreate, maxSurge/maxUnavailable, blue-green & canary |
| 5  | Services         | Networking, ClusterIP/NodePort/LoadBalancer, DNS |
| 6  | Config           | ConfigMaps, Secrets, env & volumes |
| 7  | Storage          | Volumes, PV/PVC, StatefulSets (intro) |
| 8  | Jobs & CronJobs  | One-off batch + scheduled tasks, backoff, history limits |
| 9  | Scaling          | replicas, resource requests/limits, HPA |
| 10 | Debugging        | logs, exec, events, describe (work-readiness core) |
| 11 | Health           | liveness/readiness/startup probes |
| 12 | Security         | namespaces, RBAC, ServiceAccounts (practical basics) |
| 13 | Observability    | metrics, kubectl top, intro to monitoring |
| 14 | Capstone         | deploy a multi-tier app end-to-end |

## Per-lesson format

Each module is one self-contained markdown file with a consistent shape:

1. **🎯 What you'll learn** — 3–5 outcome bullets
2. **🐳 Docker bridge** — map concept to existing Docker knowledge
3. **🧠 Concept** — plain-language explanation
4. **📊 Diagram** — mermaid (graph / flow / sequence)
5. **📋 Cheatsheet** — copy-paste kubectl + YAML daily drivers
6. **🧪 Practice** — step-by-step lab on the learner's own cluster
7. **✅ Check yourself** — outcomes + a small challenge
8. **🐛 Common gotchas** — real-world traps

The **Practice** section is the lab sheet; the live chat is the instructor —
the learner runs commands, pastes output, and gets coached/debugged in real time.

## File structure

```
D:/Study/k8s/
├── README.md                  # course home: index + how-to + progress tracker
├── 00-setup.md
├── 01-mental-model.md
├── 02-pods.md
├── 03-deployments.md
├── 04-deploy-strategies.md
├── 05-services.md
├── 06-config.md
├── 07-storage.md
├── 08-jobs-cronjobs.md
├── 09-scaling.md
├── 10-debugging.md
├── 11-health-probes.md
├── 12-security-rbac.md
├── 13-observability.md
├── 14-capstone.md
├── cheatsheet.md              # master quick-ref, grows over time
├── manifests/                 # reusable YAML for labs
└── docs/superpowers/specs/    # this spec
```

## Progression loop

1. Learner requests a module ("let's do Module N" / "next").
2. Tutor writes that module's `.md`, then summarizes in chat.
3. Learner reads it; **Practice** is done live (run → paste → coach → debug).
4. On clearing **Check yourself**, mark done in README progress tracker and `git commit`.
5. Proceed to next module.

**Pacing rule:** modules are written **one at a time, on demand** — never all
upfront — so each lesson stays fresh and adapts to where the learner struggles.

## Out of scope

- Cert-exam-only trivia and speed drills (goal is work readiness, not CKA/CKAD).
- Cloud-provider-managed cluster setup (EKS/GKE/AKS) — local cluster only,
  though concepts transfer.
- Writing all modules upfront.

## Success criteria

Learner can independently deploy a multi-tier app to a cluster, expose it,
scale it, diagnose a failing pod from logs/events, roll out and roll back a
change, and reason about config, storage, and scheduled jobs.
