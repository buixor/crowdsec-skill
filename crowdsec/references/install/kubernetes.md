# Install — Kubernetes (Helm)

Canonical docs: <https://docs.crowdsec.net/docs/next/getting_started/installation/kubernetes> · chart values <https://github.com/crowdsecurity/helm-charts/tree/main/charts/crowdsec>

Operational layer over the canonical chart docs. Verified against the
`crowdsec/crowdsec` Helm chart **0.24.0 (app v1.7.8)** on a kind cluster.

## Architecture (what the chart deploys)

| Component | Workload | Role |
|---|---|---|
| **LAPI** | Deployment (`lapi`) | Local API + DB; bouncers and agents talk to it. Optional dashboard. |
| **Agent** | DaemonSet (`agent`) | One pod per node, reads other pods' logs and ships to LAPI. |
| **AppSec** | Deployment (`appsec`, *disabled by default*) | WAF listener; separate Service for bouncers to forward to. |

Bouncers are **not** in this chart — they live with the thing they protect
(ingress controller / web bouncer, or a node firewall bouncer).

## Install

```bash
helm repo add crowdsec https://crowdsecurity.github.io/helm-charts
helm repo update
kubectl create namespace crowdsec
helm install crowdsec crowdsec/crowdsec -n crowdsec -f values.yaml
```

Minimal `values.yaml` that actually works on kind/k3d (verified shape):

```yaml
container_runtime: containerd          # SEE GOTCHA 1 — chart default is "docker"
lapi:
  persistentVolume:
    data:   { enabled: false }         # dev only; keep enabled in prod (GOTCHA 3)
    config: { enabled: false }
  dashboard:
    enabled: false
agent:
  acquisition:
    - namespace: kube-system
      podName: kube-apiserver-*
      program: kube-apiserver
appsec:
  enabled: true
  acquisitions:
    - source: appsec
      listen_addr: "0.0.0.0:7422"
      path: /
      appsec_config: crowdsecurity/appsec-default
```

## The gotchas that actually bite

### 1. `container_runtime` default is `docker`; modern clusters use `containerd`

Verified: the chart ships `container_runtime: docker`. kind, k3d, and most
managed clusters (EKS/GKE/AKS recent) run **containerd**. With the wrong value
the agent reads pod logs in the wrong format → lines read, **0 parsed**, no
alerts (the [parsing.md](../debug/parsing.md) symptom). Set
`container_runtime: containerd` unless your nodes genuinely use the Docker
runtime. Confirm with `kubectl get nodes -o wide` → CONTAINER-RUNTIME column.

### 2. Acquisition is pod-selector based, not file paths

Unlike bare-metal/Docker, `agent.acquisition` selects **pods** by
`namespace` + `podName` (glob) + `program` (which parser to apply). There is no
`/var/log/...` path. To protect an ingress controller you point it at that
controller's namespace/pod and set `program` to the matching parser (e.g.
`nginx`). `agent.additionalAcquisition` takes the classic datasource shapes
(syslog listener, kinesis, etc.) for non-pod sources.

### 3. LAPI PVCs default ON and need a StorageClass

`lapi.persistentVolume.data` (1Gi) and `.config` (100Mi) are **enabled by
default** and store registered **bouncer API keys** and LAPI credentials. With
no default StorageClass the LAPI pod stays `Pending` on an unbound PVC. kind
ships a `standard` (local-path) default SC so it works there; many bare clusters
do not — set `storageClassName` or provision a default SC. Disabling them (as
in the dev values above) means **bouncer keys reset on every LAPI restart** —
fine for dev, wrong for prod.

### 4. AppSec is a separate Deployment + Service

`appsec.enabled: true` adds an AppSec Deployment and its own Service. Bouncers
forward to the **AppSec Service DNS** (`appsec.lapiURL`/`lapiHost`/`lapiPort`
control how AppSec itself reaches LAPI, default the internal LAPI service).
`appsec.acquisitions` is the in-cluster equivalent of the bare-metal
`acquis.d/appsec.yaml` — same `source: appsec` / `listen_addr` / `appsec_config`
shape; use `crowdsecurity/appsec-default` so the health-check rule is present
(same reasoning as [../appsec/deploy.md](../appsec/deploy.md)).

### 5. RBAC / PSA

The agent needs RBAC to read pod logs cluster-wide (the chart creates the
ClusterRole/Binding). On clusters with restricted Pod Security Admission, the
DaemonSet may need a relaxed namespace label
(`pod-security.kubernetes.io/enforce: privileged|baseline`) — symptom is the
agent DaemonSet pods rejected at admission.

### 6. kind/k3d dev: disk pressure is real (verified failure)

kind/k3d nodes pull CrowdSec images into the node's overlay FS; a small VM
fills up and the apiserver wedges with `net/http: TLS handshake timeout`
mid-rollout — not an obvious CrowdSec error. Check `docker exec <node> df -h /`
and budget several GB free.

## Validate

`cscli` runs inside the LAPI pod. The bundled helper supports this:

```bash
~/.claude/skills/crowdsec/scripts/diagnose.sh --env k8s --namespace crowdsec --pod <lapi-pod>
# manual equivalent:
LAPI=$(kubectl get pod -n crowdsec -l type=lapi -o name | head -1)
kubectl exec -n crowdsec $LAPI -- cscli lapi status
kubectl exec -n crowdsec $LAPI -- cscli metrics
```

WAF smoke test: `kubectl port-forward -n crowdsec svc/crowdsec-appsec-service
7422:7422` then run the `curl` allow/block probe from
[../appsec/deploy.md](../appsec/deploy.md) against `127.0.0.1:7422`.

## Teardown

```bash
helm uninstall crowdsec -n crowdsec
kubectl delete namespace crowdsec
kind delete cluster --name <name>     # dev: also reclaims the node image's disk
```

## Next step

Run the probes in [../operate/health-check.md](../operate/health-check.md)
(use the `kubectl exec … cscli` row in its per-environment table).
