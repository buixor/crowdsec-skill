# Debug — Platform-specific gotchas

Canonical docs: <https://docs.crowdsec.net/docs/next/troubleshooting/intro>

Most CrowdSec troubleshooting is platform-agnostic — the symptom docs
([parsing](../symptoms/parsing.md), [no-alerts](../symptoms/no-alerts.md),
[not-blocked](../symptoms/not-blocked.md)) apply everywhere and only the command
prefix changes (`sudo cscli …` → `docker exec <name> …` → `kubectl exec -n <ns>
<pod> -- …`). This page collects the failures that are genuinely **specific to
how the engine is deployed** — the ones that don't reduce to a prefix.

Reach here from the symptom docs when a check says "0 lines read" or "permission
denied" and the cause turns out to be the platform, not the config.

## Docker / docker-compose — host log path not mounted in

The single most common containerised failure: acquisition points at a path that
exists **on the host** but was never bind-mounted into the container, so the
engine reads zero lines.

Symptom: `cscli metrics show acquisition` shows the source with **0 lines read**
(or the row absent), even though `filenames:`/`type:` look correct.

Confirm from *inside* the container — the host view lies:

```bash
docker exec <name> ls -l /var/log/nginx/access.log    # No such file ⇒ not mounted
```

Fix: add the host log directory to the crowdsec service's `volumes:` (read-only
is fine), e.g. `- /var/log/nginx:/var/log/nginx:ro`, then recreate the
container. The acquisition path inside the container must match the mount target.
See [../../configure/acquisition.md](../../configure/acquisition.md).

## Kubernetes — mount + container runtime

Two distinct k8s-only causes:

- **Path not mounted into the pod** (same class as Docker above). Verify inside:
  ```bash
  kubectl exec -n <ns> <pod> -- ls -l <path>
  ```
  Pod/container logs live under the node's `/var/log/pods` or
  `/var/log/containers`; the agent DaemonSet must hostPath-mount that directory.

- **Wrong `container_runtime`** → lines read, **0 parsed**. Managed clusters
  (recent EKS/GKE/AKS) run **containerd**, not Docker; with the wrong value the
  agent reads pod logs in the wrong format and no parser claims them. Set
  `container_runtime: containerd` unless nodes genuinely use the Docker runtime;
  confirm with `kubectl get nodes -o wide` (CONTAINER-RUNTIME column). See
  [../../install/kubernetes.md](../../install/kubernetes.md).

## systemd / bare-metal — SELinux / AppArmor denials

When the `crowdsec` user *can* read a log file by hand but the engine still gets
**0 lines read** or `permission denied`, mandatory-access-control is blocking the
service even though POSIX permissions allow it.

```bash
sudo -u crowdsec head <path>          # succeeds ⇒ not a POSIX-perms problem
sudo ausearch -m avc -ts recent       # SELinux denials (RHEL/Fedora/Alma/Rocky)
sudo dmesg | grep -i denied           # AppArmor denials (Debian/Ubuntu)
```

Fix by relabelling / adding policy for the path the engine reads — **do not
disable enforcement** to "make it work". For a non-standard log location, apply
the log file context (e.g. `var_log_t` on SELinux) or extend the crowdsec
AppArmor profile, then retry.

## journald — group access

A file-source → journald migration silently reads nothing if the `crowdsec` user
isn't in a group permitted to read the journal (`systemd-journal`, or the unit's
`SupplementaryGroups`). This is a systemd-specific variant of the perms check in
[../symptoms/parsing.md](../symptoms/parsing.md) § Reachability.
