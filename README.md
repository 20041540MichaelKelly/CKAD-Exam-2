# CKAD Mock Exam — Paper 2

**Duration:** 120 minutes  **Tasks:** 19  **Pass mark:** 66% (unofficial; the LF doesn't publish one)

This paper covers the v1.37 curriculum items that aren't in Paper 1: Kustomize, CRDs, RBAC, admission control, API deprecations, volumes, init/sidecar containers, DaemonSets, `kubectl top`, and debugging.

Use context `kind-ckad` from your own terminal, unless a task says **Host**. For those, run `./ssh.sh` to open a shell on that host (like `ssh ckad-control-plane` in the real exam), do the work there, then `exit`. Task files are in this kit's `paper2/` folder.

---

### Task 1 (6%) — Namespace `kz-prod`
A Kustomize base is at `paper2/kustomize/base`. Create an overlay at `paper2/kustomize/overlays/prod/kustomization.yaml` that:
- puts everything in namespace `kz-prod`
- adds the name prefix `prod-`
- sets the `shop` Deployment to **3** replicas
- adds the label `env=prod` to all resources

Apply the overlay to the cluster. Do not edit the base.

---

### Task 2 (5%) — Namespace `crd-ns` · **Host `ckad-control-plane`** for part 2
1. A CRD for `Backup` resources exists in the cluster. Create a Backup named `nightly` in namespace `crd-ns` with `schedule: "0 2 * * *"`, `target: postgres` and `retention: 7`.
2. On host `ckad-control-plane`, write the CRD's **short name** (only the short name) into `/opt/course/2/shortname.txt`.

---

### Task 3 (6%) — Namespace `rbac-ns`
ServiceAccount `pod-reader` exists. Create a Role `pod-read` that allows `get`, `list` and `watch` on Pods, and a RoleBinding `pod-read-bind` that grants it to `pod-reader`. The ServiceAccount must not be able to delete Pods.

---

### Task 4 (5%) — Namespace `rbac-ns`
ServiceAccount `client` should be able to list ConfigMaps through the existing Role `cm-reader`, but it can't. Find and fix the problem. `client` must not get access to Secrets.

---

### Task 5 (6%) — Namespace `legacy`
`paper2/legacy/app.yaml` was written for a very old cluster and is rejected by the API server. Fix the file **in place** so it uses current API versions and is valid, then apply it. The Deployment `legacy-app` must end up with 2 ready replicas, and the HPA `legacy-hpa` must target it.

---

### Task 6 (6%) — Namespace `storage`
Create a PersistentVolumeClaim `data-pvc` requesting `100Mi`, access mode `ReadWriteOnce`, using the cluster's default StorageClass. Create a Pod `data-pod` (image `busybox:1.36`) that mounts it at `/data` and runs:
```
sh -c 'echo hello > /data/hello.txt && sleep 36000'
```
The PVC must be `Bound` and the Pod `Running`.

---

### Task 7 (5%) — Namespace `storage`
Create a PersistentVolume `pv-local` with capacity `200Mi`, access mode `ReadWriteOnce`, `storageClassName: manual` and a `hostPath` of `/mnt/ckad`. Then create a PVC `pvc-local` in namespace `storage` that binds to this PV.

---

### Task 8 (5%) — Namespace `init-ns`
Create a Pod `web-init` running `nginx:1.27`. Add an init container `setup` (image `busybox:1.36`) that writes the text `initialized` to `index.html` in an `emptyDir` volume named `html`. nginx must serve that file from `/usr/share/nginx/html`.

---

### Task 9 (4%) — Namespace `init-ns`
Create a Pod `native-sidecar` with:
- main container `app`: image `nginx:1.27`
- a **native sidecar** (an init container that keeps running for the life of the Pod) named `log-shipper`: image `busybox:1.36`, command `sleep 36000`

The Pod must be Running and Ready.

---

### Task 10 (5%) — Namespace `ds-ns`
Create a DaemonSet `node-agent` (image `busybox:1.36`, command `sleep 36000`) that runs one Pod on **every** node in the cluster, including the control-plane node.

---

### Task 11 (6%) — Namespace `bluegreen`
Deployment `app-blue` sits behind Service `myapp`. Do a blue/green switch:
1. Create Deployment `app-green` with 2 replicas, image `nginx:1.27-alpine`, and labels `app=myapp`, `version=green`.
2. Once it's ready, switch Service `myapp` to send all traffic to green.
3. Scale `app-blue` to 0 (don't delete it).

---

### Task 12 (5%) — Namespace `metrics` · **Host `ckad-control-plane`**
One of the Pods in namespace `metrics` is using far more CPU than the others. On host `ckad-control-plane`, write **only that Pod's name** to `/opt/course/12/pod.txt`.

---

### Task 13 (6%) — Namespace `debug`
Deployment `payments` keeps crashing. Find out why and fix it so the Pod stays running. Don't change the container's command.

---

### Task 14 (5%) — Namespace `debug`
Deployment `reports` has a Pod stuck in `Pending`. Make it schedule onto node `ckad-worker` **without modifying the Deployment**.

---

### Task 15 (5%) — Namespace `netpol`
The namespace has a default-deny policy. NetworkPolicy `db-allow` is meant to let Pods labelled `app=backend` reach the `app=db` Pods on TCP port `5432`, but it's wrong. Fix `db-allow`. Leave `default-deny` in place.

---

### Task 16 (5%) — Namespace `svc-debug`
Service `checkout` should send traffic to the `checkout` Deployment's nginx Pods, but requests fail. Fix the **Service** (not the Deployment) so it works on port 80.

---

### Task 17 (5%) — Namespace `limits`
`paper2/limits/pod.yaml` is rejected when you apply it. Find out why, then change only its resource values so it's admitted, with a memory **limit** set to the **maximum** this namespace allows and a memory request of `128Mi`. The Pod `limited` must be Running.

---

### Task 18 (5%) — Namespace `restricted`
Create a Pod `psa-pod` (image `busybox:1.36`, command `sleep 36000`) in namespace `restricted`. The namespace enforces a Pod Security Standard. The Pod must be admitted and Running.

---

### Task 19 (5%) — Local machine + namespace `images`
1. Edit `paper2/build/Dockerfile` so it sets environment variable `APP_VERSION=v2` and writes `ckad mock app v2` to `index.html`.
2. Build it as `ckad-app:v2` and load it into the kind cluster `ckad`.
3. Create Pod `app-v2` in namespace `images` using that image. It must never try to pull from a registry.

---

**End of Paper 2.** Run `./grade2.sh` to see your score.

