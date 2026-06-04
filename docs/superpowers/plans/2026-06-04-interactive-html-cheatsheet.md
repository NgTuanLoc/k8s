# Interactive HTML Cheatsheet Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Build `cheatsheet.html` — a single, self-contained, offline interactive version of `cheatsheet.md` with live search, collapsible modules, copy-to-clipboard, and auto dark/light theming.

**Architecture:** One HTML file with inline `<style>` and inline vanilla JS. Content (Modules 0–14) is transcribed verbatim from `cheatsheet.md` into native `<details>` cards. CSS custom properties drive two themes; vanilla JS handles search, expand/collapse, copy, and a persisted theme toggle. No build step, no dependencies, no network.

**Tech Stack:** HTML5, CSS custom properties + `prefers-color-scheme`, vanilla JavaScript (`<details>`/`<summary>`, `navigator.clipboard` with `execCommand` fallback, `localStorage`).

**Verification model:** No JS test runner exists in this repo and the deliverable is a static offline file. Each task is verified with the spec's manual browser acceptance checks (concrete expected outcomes given per task). Commit after each task.

---

## File Structure

```
cheatsheet.html   # the single deliverable, repo root — created in Task 1, extended by every later task
```

Internal regions of the file (all tasks edit this one file):

```
<!doctype html>
<html lang="en">
  <head> … <style> (Task 1) … </head>
  <body>
    <header> … title, search, theme toggle, expand/collapse (Task 1) … </header>
    <main id="modules"> … 15 <details> cards (Task 2) … </main>
    <p id="no-results"> … (Task 5) … </p>
    <script> … copy (Task 3), expand/collapse (Task 4), search (Task 5), theme (Task 6) … </script>
  </body>
</html>
```

`cheatsheet.md` is the verbatim content source for Task 2 and is NOT modified.

---

## Task 1: HTML skeleton + themes + header shell

**Files:**
- Create: `cheatsheet.html`

- [ ] **Step 1: Create the file with skeleton, CSS themes, and header**

Create `cheatsheet.html` with exactly this content:

```html
<!doctype html>
<html lang="en">
<head>
<meta charset="utf-8">
<meta name="viewport" content="width=device-width, initial-scale=1">
<title>Kubernetes Master Cheatsheet</title>
<style>
  :root {
    --bg: #0d1117; --card: #161b22; --text: #e6edf3; --muted: #9da7b3;
    --accent: #39d353; --accent2: #56d4dd; --border: #30363d;
    --pre-bg: #010409; --mark: #ffd33d; --mark-text: #1c1c00;
  }
  :root[data-theme="light"] {
    --bg: #f6f8fa; --card: #ffffff; --text: #1f2328; --muted: #59636e;
    --accent: #326CE5; --accent2: #0969da; --border: #d0d7de;
    --pre-bg: #f6f8fa; --mark: #fff3b0; --mark-text: #1c1c00;
  }
  @media (prefers-color-scheme: light) {
    :root:not([data-theme="dark"]) {
      --bg: #f6f8fa; --card: #ffffff; --text: #1f2328; --muted: #59636e;
      --accent: #326CE5; --accent2: #0969da; --border: #d0d7de;
      --pre-bg: #f6f8fa; --mark: #fff3b0; --mark-text: #1c1c00;
    }
  }
  * { box-sizing: border-box; }
  body {
    margin: 0; background: var(--bg); color: var(--text);
    font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, sans-serif;
    line-height: 1.5;
  }
  header {
    position: sticky; top: 0; z-index: 10; background: var(--card);
    border-bottom: 1px solid var(--border); padding: 12px 16px;
    display: flex; flex-wrap: wrap; gap: 8px; align-items: center;
  }
  header h1 { font-size: 1.1rem; margin: 0; flex: 1 1 auto; }
  #search {
    flex: 2 1 220px; min-width: 160px; padding: 8px 10px;
    background: var(--bg); color: var(--text);
    border: 1px solid var(--border); border-radius: 6px; font-size: 0.95rem;
  }
  header button {
    padding: 8px 10px; background: var(--bg); color: var(--text);
    border: 1px solid var(--border); border-radius: 6px; cursor: pointer;
    font-size: 0.85rem;
  }
  header button:hover { border-color: var(--accent); }
  main { max-width: 960px; margin: 0 auto; padding: 16px; }
  details {
    background: var(--card); border: 1px solid var(--border);
    border-radius: 8px; margin-bottom: 12px; padding: 0 14px;
  }
  details[open] { padding-bottom: 12px; }
  summary {
    cursor: pointer; padding: 12px 0; font-weight: 600;
    font-size: 1.02rem; color: var(--accent);
  }
  summary::marker { color: var(--muted); }
  .concept { color: var(--muted); margin: 4px 0 12px; font-size: 0.92rem; }
  .block { position: relative; margin: 0 0 12px; }
  pre {
    background: var(--pre-bg); border: 1px solid var(--border);
    border-radius: 6px; padding: 12px 14px; overflow-x: auto; margin: 0;
    font-family: "SFMono-Regular", Consolas, "Liberation Mono", monospace;
    font-size: 0.86rem; line-height: 1.45;
  }
  pre code { color: var(--text); }
  .copy-btn {
    position: absolute; top: 6px; right: 6px;
    padding: 3px 8px; font-size: 0.72rem; cursor: pointer;
    background: var(--card); color: var(--muted);
    border: 1px solid var(--border); border-radius: 5px; opacity: 0.85;
  }
  .copy-btn:hover { color: var(--accent); border-color: var(--accent); }
  mark { background: var(--mark); color: var(--mark-text); border-radius: 2px; }
  #no-results { display: none; text-align: center; color: var(--muted); padding: 24px; }
  #no-results.show { display: block; }
</style>
</head>
<body>
<header>
  <h1>📋 Kubernetes Master Cheatsheet</h1>
  <input id="search" type="search" placeholder="Search commands…" aria-label="Search commands">
  <button id="expand-all" type="button">Expand all</button>
  <button id="collapse-all" type="button">Collapse all</button>
  <button id="theme-toggle" type="button" aria-label="Toggle dark/light theme">🌙</button>
</header>
<main id="modules">
  <!-- Module cards inserted in Task 2 -->
</main>
<p id="no-results">No commands match your search.</p>
<script>
  // Behavior wired up in Tasks 3–6
</script>
</body>
</html>
```

- [ ] **Step 2: Verify it opens and themes work**

Open `cheatsheet.html` by double-clicking (file:// in your browser).
Expected:
- Page loads with the sticky header (title, search box, three buttons).
- On an OS set to dark mode → dark terminal palette; on light mode → light/blue palette.
- No console errors.

- [ ] **Step 3: Commit**

```bash
git add cheatsheet.html
git commit -m "feat: scaffold interactive cheatsheet HTML with themes and header"
```

---

## Task 2: Transcribe all 15 module cards from cheatsheet.md

**Files:**
- Modify: `cheatsheet.html` (replace the `<!-- Module cards inserted in Task 2 -->` comment inside `<main id="modules">`)
- Read (source, do not modify): `cheatsheet.md`

Each module in `cheatsheet.md` becomes one `<details>` card following this exact template:

```html
<details open>
  <summary>MODULE TITLE</summary>
  <p class="concept">CONCEPT SUMMARY LINE</p>
  <div class="block">
    <button class="copy-btn" type="button">Copy</button>
    <pre><code>COMMAND BLOCK TEXT (verbatim, HTML-escaped)</code></pre>
  </div>
  <!-- one .block per fenced code block in that module -->
</details>
```

Rules for transcription:
- Use the section heading from `cheatsheet.md` as the `<summary>` (e.g. `Pods: logs & exec (Module 2)`).
- Put the prose sentence(s) that follow the code blocks into `<p class="concept">`.
- Create one `.block` (button + `<pre><code>`) per fenced code block. A module with two fenced blocks (e.g. Module 0, Module 9) gets two `.block` elements.
- Copy command text **verbatim**, including inline `#` comments.
- HTML-escape inside `<code>`: `&` → `&amp;`, `<` → `&lt;`, `>` → `&gt;`. (Several commands contain `>`, `<`, `&`, e.g. `base64 -d`, jsonpath `>`, `&&`.)
- Start every card with `<details open>` (default expanded; Collapse-all is available).

- [ ] **Step 1: Add Module 0 and Module 1 cards (worked examples)**

Replace the `<!-- Module cards inserted in Task 2 -->` comment with these two cards first (note Module 0 has two code blocks: contexts/clusters and pods intro):

```html
  <details open>
    <summary>Contexts &amp; clusters (Module 0)</summary>
    <p class="concept">Switch between clusters and inspect node/pod state.</p>
    <div class="block">
      <button class="copy-btn" type="button">Copy</button>
      <pre><code>kubectl config get-contexts        # list reachable clusters
kubectl config current-context     # active cluster
kubectl config use-context NAME    # switch active cluster
kubectl cluster-info               # control-plane URL
kubectl get nodes                  # nodes (want STATUS=Ready)
kubectl get pods -A                # pods in ALL namespaces</code></pre>
    </div>
    <div class="block">
      <button class="copy-btn" type="button">Copy</button>
      <pre><code>kind create cluster --name k8s-course   # create kind cluster
kind delete cluster --name k8s-course   # delete it
minikube start / minikube stop          # minikube lifecycle</code></pre>
    </div>
  </details>

  <details open>
    <summary>Pods (Module 0 intro)</summary>
    <p class="concept">Smallest unit you can run directly.</p>
    <div class="block">
      <button class="copy-btn" type="button">Copy</button>
      <pre><code>kubectl run NAME --image=IMG --restart=Never   # create a bare pod
kubectl get pods                               # list pods
kubectl delete pod NAME                         # delete a pod</code></pre>
    </div>
  </details>

  <details open>
    <summary>Mental model: spec vs status (Module 1)</summary>
    <p class="concept">Reconcile loop = controller runs forever: observe → compare spec vs status → act.</p>
    <div class="block">
      <button class="copy-btn" type="button">Copy</button>
      <pre><code>kubectl apply -f file.yaml        # declarative: create OR update to desired state
kubectl delete -f file.yaml       # remove what the file declares

kubectl get pod NAME -o yaml      # full object: see spec: (desired) and status: (actual)
kubectl get pod NAME -o jsonpath='{.spec.containers[0].image}{"\n"}'   # desired image
kubectl get pod NAME -o jsonpath='{.status.phase}{"\n"}'               # actual phase

kubectl api-resources             # every kind you can create (+ short names)
kubectl explain pod               # docs for a kind
kubectl explain pod.spec.containers   # drill into any field</code></pre>
    </div>
  </details>
```

- [ ] **Step 2: Transcribe the remaining modules (2–14)**

Append cards for Modules 2 through 14, in order, using the same template and the verbatim content from `cheatsheet.md`. Mapping of each `<summary>` to its concept line and number of `.block`s:

| Summary (from cheatsheet.md heading) | Concept line (the prose under the code) | # blocks |
|---|---|---|
| `Pods: logs &amp; exec (Module 2)` | Pod = smallest unit; 1+ containers sharing one IP (`localhost`) + volumes (sidecar pattern). | 1 |
| `Deployments: scale, rollout, rollback (Module 3)` | Deployment → ReplicaSet → Pods. The ReplicaSet self-heals; `selector` must match `template.labels`. | 1 |
| `Deploy strategies (Module 4)` | Recreate = downtime; RollingUpdate = zero-downtime. Safe rolls need readiness probes (Module 11). | 1 |
| `Services &amp; DNS (Module 5)` | Types: ClusterIP (in-cluster) · NodePort (node port) · LoadBalancer (cloud). Empty endpoints = selector/label mismatch. | 1 |
| `Config: ConfigMaps &amp; Secrets (Module 6)` | Inject as env (`valueFrom`/`envFrom`, cached at start) or as files (volume mount, updates live). Secrets are base64, not encrypted. | 1 |
| `Storage: PVC/PV, StatefulSets (Module 7)` | PVC (request) → StorageClass provisions → PV (disk). Data on a PVC survives pod deletion. STS = stable names + PVC per pod. | 1 |
| `Jobs &amp; CronJobs (Module 8)` | Job = run-to-completion (restartPolicy Never/OnFailure). backoffLimit=retries, ttlSecondsAfterFinished=auto-clean. CronJob = scheduled Job; concurrencyPolicy Forbid/Replace avoids overlap. | 1 |
| `Scaling: requests/limits &amp; HPA (Module 9)` | requests = scheduler reserves + HPA baseline; limits = hard cap (CPU throttle, mem OOMKill). HPA needs metrics-server + CPU requests. | 2 |
| `Debugging loop (Module 10)` | Pending = scheduling (describe). CrashLoopBackOff = app (logs). ImagePullBackOff = image/registry (describe). | 1 |
| `Health probes (Module 11)` | liveness = restart wedged container; readiness = gate traffic (Endpoints); startup = protect slow boot. Don't put dependency checks in liveness. | 1 |
| `Security &amp; RBAC (Module 12)` | Role (namespaced) + RoleBinding → subject. ClusterRole/Binding = cluster-wide. Least privilege; every pod runs as a ServiceAccount. | 1 |
| `Observability (Module 13)` | Built-ins show CURRENT state only (top/events/logs are ephemeral). Production adds Prometheus (metrics history) + Grafana (dashboards) + Loki/ELK (logs). | 1 |
| `Capstone: multi-tier with Kustomize (Module 14)` | Combine: Namespace + Secret + ConfigMap + StatefulSet(DB) + Deployments + Services + probes + HPA. initContainer (`nc -z db 5432`) enforces start ordering. | 1 |

For Module 9 the two blocks are (a) the metrics-server install+patch block and (b) the `top`/`autoscale`/`get hpa` block — copy both verbatim from `cheatsheet.md`. Module 11's block is YAML (the three probes) — transcribe it verbatim into `<pre><code>`, HTML-escaping `{`, `}` are fine but escape `<`/`>`/`&` if present (none are in that block).

- [ ] **Step 3: Verify all 15 modules render**

Reload `cheatsheet.html`.
Expected:
- 15 `<details>` cards, all expanded, titles matching the table above + Modules 0/1.
- Each command block shows verbatim text with no broken HTML entities (check Module 1 jsonpath `>`/`<`, Module 6 `base64 -d` `|`, Module 9 `&&`/`>` render as literal characters, not entities or broken tags).
- A "Copy" button sits in the top-right of every block (not yet wired).

- [ ] **Step 4: Commit**

```bash
git add cheatsheet.html
git commit -m "feat: transcribe all 15 cheatsheet modules into HTML cards"
```

---

## Task 3: Copy-to-clipboard

**Files:**
- Modify: `cheatsheet.html` (the `<script>` block)

- [ ] **Step 1: Add the copy handler**

Replace `// Behavior wired up in Tasks 3–6` with:

```javascript
function copyText(text, btn) {
  const done = () => {
    const original = btn.textContent;
    btn.textContent = "Copied ✓";
    setTimeout(() => { btn.textContent = original; }, 1500);
  };
  if (navigator.clipboard && window.isSecureContext) {
    navigator.clipboard.writeText(text).then(done).catch(() => fallbackCopy(text, done));
  } else {
    fallbackCopy(text, done);
  }
}
function fallbackCopy(text, done) {
  const ta = document.createElement("textarea");
  ta.value = text;
  ta.style.position = "fixed";
  ta.style.opacity = "0";
  document.body.appendChild(ta);
  ta.select();
  try { document.execCommand("copy"); done(); } finally { document.body.removeChild(ta); }
}
document.querySelectorAll(".copy-btn").forEach(btn => {
  btn.addEventListener("click", () => {
    const code = btn.parentElement.querySelector("code");
    copyText(code.innerText, btn);
  });
});
```

- [ ] **Step 2: Verify copy works**

Reload. Click the Copy button on Module 0's first block. Paste into a text editor.
Expected:
- Clipboard contains the exact 6 `kubectl config …` lines (with `#` comments), no HTML entities.
- Button briefly shows "Copied ✓" then reverts to "Copy".
- Works over `file://` (fallback path) with no network and no console error.

- [ ] **Step 3: Commit**

```bash
git add cheatsheet.html
git commit -m "feat: add copy-to-clipboard with offline fallback"
```

---

## Task 4: Expand-all / Collapse-all

**Files:**
- Modify: `cheatsheet.html` (the `<script>` block)

- [ ] **Step 1: Add the expand/collapse handlers**

Append to the `<script>`:

```javascript
const allDetails = () => document.querySelectorAll("#modules details");
document.getElementById("expand-all").addEventListener("click", () => {
  allDetails().forEach(d => { d.open = true; });
});
document.getElementById("collapse-all").addEventListener("click", () => {
  allDetails().forEach(d => { d.open = false; });
});
```

- [ ] **Step 2: Verify**

Reload. Click "Collapse all", then "Expand all".
Expected:
- Collapse all → every module closes (only titles visible).
- Expand all → every module opens.
- No console errors.

- [ ] **Step 3: Commit**

```bash
git add cheatsheet.html
git commit -m "feat: add expand-all/collapse-all controls"
```

---

## Task 5: Live search / filter with highlight

**Files:**
- Modify: `cheatsheet.html` (the `<script>` block)

- [ ] **Step 1: Add the search handler**

Append to the `<script>`:

```javascript
const searchInput = document.getElementById("search");
const noResults = document.getElementById("no-results");

// Remember each module's open state before the first search, restore on clear.
let preSearchState = null;

function clearHighlights() {
  document.querySelectorAll("#modules mark").forEach(m => {
    const parent = m.parentNode;
    parent.replaceChild(document.createTextNode(m.textContent), m);
    parent.normalize();
  });
}

function highlight(root, term) {
  const walker = document.createTreeWalker(root, NodeFilter.SHOW_TEXT, null);
  const targets = [];
  while (walker.nextNode()) {
    if (walker.currentNode.nodeValue.toLowerCase().includes(term)) targets.push(walker.currentNode);
  }
  targets.forEach(node => {
    const frag = document.createDocumentFragment();
    const text = node.nodeValue;
    const lower = text.toLowerCase();
    let i = 0, idx;
    while ((idx = lower.indexOf(term, i)) !== -1) {
      if (idx > i) frag.appendChild(document.createTextNode(text.slice(i, idx)));
      const mark = document.createElement("mark");
      mark.textContent = text.slice(idx, idx + term.length);
      frag.appendChild(mark);
      i = idx + term.length;
    }
    if (i < text.length) frag.appendChild(document.createTextNode(text.slice(i)));
    node.parentNode.replaceChild(frag, node);
  });
}

searchInput.addEventListener("input", () => {
  const term = searchInput.value.trim().toLowerCase();
  if (preSearchState === null && term !== "") {
    preSearchState = [...allDetails()].map(d => d.open);
  }
  clearHighlights();

  if (term === "") {
    if (preSearchState) {
      [...allDetails()].forEach((d, i) => { d.open = preSearchState[i]; d.hidden = false; });
      preSearchState = null;
    } else {
      allDetails().forEach(d => { d.hidden = false; });
    }
    noResults.classList.remove("show");
    return;
  }

  let anyMatch = false;
  allDetails().forEach(d => {
    const match = d.textContent.toLowerCase().includes(term);
    d.hidden = !match;
    d.open = match;
    if (match) { highlight(d, term); anyMatch = true; }
  });
  noResults.classList.toggle("show", !anyMatch);
});
```

- [ ] **Step 2: Verify search**

Reload. Type `rollout` in the search box.
Expected:
- Only modules containing "rollout" remain visible (Module 3 Deployments at least), expanded, with "rollout" highlighted.
- Type gibberish `zzzzz` → all cards hidden, "No commands match your search." shows.
- Clear the box → all 15 cards return; their open/closed state matches what it was before searching.

- [ ] **Step 3: Commit**

```bash
git add cheatsheet.html
git commit -m "feat: add live search with highlight and no-results state"
```

---

## Task 6: Theme toggle with persistence

**Files:**
- Modify: `cheatsheet.html` (the `<script>` block)

- [ ] **Step 1: Add the theme toggle handler**

Append to the `<script>`:

```javascript
const root = document.documentElement;
const themeBtn = document.getElementById("theme-toggle");

function applyTheme(theme) {
  // theme: "dark" | "light" | null (null = follow OS)
  if (theme) root.setAttribute("data-theme", theme);
  else root.removeAttribute("data-theme");
  const effectiveDark = theme
    ? theme === "dark"
    : !window.matchMedia("(prefers-color-scheme: light)").matches;
  themeBtn.textContent = effectiveDark ? "☀️" : "🌙";
}

applyTheme(localStorage.getItem("cheatsheet-theme"));

themeBtn.addEventListener("click", () => {
  const current = root.getAttribute("data-theme")
    || (window.matchMedia("(prefers-color-scheme: light)").matches ? "light" : "dark");
  const next = current === "dark" ? "light" : "dark";
  localStorage.setItem("cheatsheet-theme", next);
  applyTheme(next);
});
```

- [ ] **Step 2: Verify theming**

Reload.
Expected:
- With no stored choice, the page matches your OS theme; the toggle icon shows ☀️ in dark mode / 🌙 in light mode.
- Click the toggle → palette flips (dark terminal ↔ light/blue), icon updates.
- Reload → the chosen theme persists (localStorage).
- Clear localStorage (`localStorage.removeItem("cheatsheet-theme")`) and reload → back to following OS.

- [ ] **Step 3: Commit**

```bash
git add cheatsheet.html
git commit -m "feat: add persisted dark/light theme toggle"
```

---

## Task 7: Final acceptance pass

**Files:**
- Modify (only if a check fails): `cheatsheet.html`

- [ ] **Step 1: Run the full spec verification checklist**

Open `cheatsheet.html` via `file://` with networking off and confirm all six spec checks:

1. All 15 modules render with correct titles, concept lines, and command blocks.
2. Search "rollout" → only matching modules remain, expanded, highlighted; clearing restores all (and prior open/closed state).
3. Collapse all → every module closed; Expand all → every module open.
4. Copy button → clipboard holds the exact command; button shows "Copied ✓".
5. Theme toggle → palette switches and persists across reload; no stored choice → matches OS.
6. Offline `file://` → everything works including copy.

Also spot-check special characters render literally (not as entities or broken tags): Module 1 jsonpath `{"\n"}` and `>`, Module 6 `| base64 -d`, Module 9 `&&` and `\`.

- [ ] **Step 2: Commit any fixes (skip if none)**

```bash
git add cheatsheet.html
git commit -m "fix: address final cheatsheet acceptance findings"
```

---

## Self-Review (completed by plan author)

**Spec coverage:**
- Single self-contained file, offline, no deps → Task 1 (skeleton, inline style/script) + verified offline in Tasks 3 & 7. ✅
- All 15 modules, verbatim content → Task 2. ✅
- Live search + highlight + no-results → Task 5. ✅
- Collapsible modules + expand/collapse all → Task 1 (`<details>`) + Task 4. ✅
- Copy-to-clipboard + offline fallback → Task 3. ✅
- Auto dark/light + manual toggle + localStorage persistence → Task 1 (CSS) + Task 6 (JS). ✅
- Non-goals (no progress checkboxes, no regeneration tooling, no frameworks) → respected. ✅
- Accessibility (focusable `<details>`/buttons, `aria-label` on icon buttons) → Task 1 markup. ✅

**Placeholder scan:** None — every code step contains the exact, final code to use.

**Type/name consistency:** `allDetails()` defined in Task 4 and reused in Task 5; `applyTheme()` defined once in Task 6; `copyText`/`fallbackCopy` defined in Task 3; element IDs (`search`, `expand-all`, `collapse-all`, `theme-toggle`, `modules`, `no-results`) match between Task 1 markup and Tasks 3–6 scripts. Consistent.
