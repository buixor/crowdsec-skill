# Install — Docker / docker-compose

Canonical docs: <https://docs.crowdsec.net/docs/next/getting_started/installation/docker> · image reference <https://docs.crowdsec.net/u/getting_started/installation/#docker>

Operational layer over the canonical image docs. Verified with
`crowdsecurity/crowdsec:latest` (engine v1.7.8) via compose.

## Minimal working compose (verified)

```yaml
services:
  crowdsec:
    image: crowdsecurity/crowdsec:latest      # pin to a minor in prod, e.g. :v1.7
    container_name: crowdsec
    environment:
      COLLECTIONS: "crowdsecurity/sshd crowdsecurity/appsec-virtual-patching crowdsecurity/appsec-generic-rules"
      GID: "${GID:-1000}"
    volumes:
      - cs-config:/etc/crowdsec
      - cs-data:/var/lib/crowdsec/data
      - /var/log/auth.log:/logs/auth.log:ro          # host log, read-only
      - ./acquis.d:/etc/crowdsec/acquis.d:ro          # your acquisition
    ports:
      - "8081:8080"      # LAPI (see port-conflict note)
      - "7423:7422"      # AppSec (only if you run the WAF)
    restart: unless-stopped
volumes:
  cs-config:
  cs-data:
```

Bring up: `docker compose up -d`. The image installs the `COLLECTIONS` on
first boot (verified: `sshd`, `appsec-virtual-patching`, `appsec-generic-rules`
all `enabled` after startup).

## The gotchas that actually bite

### 1. Acquisition paths are *container* paths, not host paths

This is the #1 Docker mistake. You mount `/var/log/auth.log` to
`/logs/auth.log` — your acquisition file must reference the **in-container**
path:

```yaml
# acquis.d/sshd.yaml
filenames:
  - /logs/auth.log        # NOT /var/log/auth.log
labels: { type: syslog }
source: file
```

A path that exists on the host but isn't mounted reads **0 lines** silently —
verify with `docker exec crowdsec cscli metrics show acquisition`. To read
*other containers'* logs instead of files, use the built-in Docker datasource
(`datasource_docker` is compiled in) with `source: docker` and the docker
socket mounted — but that's a different acquisition shape; start with file
mounts.

### 2. `COLLECTIONS` only applies to a *fresh* config volume

`COLLECTIONS`/`PARSERS`/`SCENARIOS` env vars run on first boot **when
`/etc/crowdsec` is empty**. Because the compose above persists
`cs-config:/etc/crowdsec`, editing the env var later does nothing — the volume
already has a config. After first boot, manage the hub with
`docker exec crowdsec cscli collections install …` (and `docker compose restart`
or `cscli` reload), or recreate the config volume. Don't expect changing the
env var to retro-install.

### 3. Port conflict with a host-installed engine

If a bare-metal CrowdSec already owns `8080`/`7422` (verified — the container
won't bind them), map to free host ports as above (`8081:8080`,
`7423:7422`). Bouncers and `cscli -u` then target the mapped host port. This is
the normal coexistence pattern while migrating host→container.

### 4. AppSec must listen on `0.0.0.0` inside the container

The AppSec acquisition must set `listen_addr: 0.0.0.0:7422` (not `127.0.0.1`)
or the published port reaches nothing. Verified with `7423:7422` + a host
`curl http://127.0.0.1:7423/` smoke test (`allow: 200` / `block: 403`).

### 5. Other env-in-container realities

- **`GID`**: set it so the container user can read mounted journald
  sockets/group-readable logs; mismatch = 0 lines read despite a correct mount.
- **Time skew**: a container with a wrong clock fails CAPI TLS
  (`cscli capi status` errors). Containers normally inherit host time — only an
  issue with custom runtimes.
- **IPv6**: the AppSec/firewall behaviour mirrors bare-metal; container
  networking is v4 by default unless you enable v6 on the daemon/network.

## Bouncer key bootstrap

```bash
# create a key for an external bouncer (web server, firewall, AppSec)
docker exec crowdsec cscli bouncers add my-bouncer -o raw
```

Use that key in the bouncer's config (`api_url` → the mapped host port, e.g.
`http://<host>:8081/`). For declarative bootstrap, the image also honours
`BOUNCER_KEY_<name>` env vars; `cscli bouncers add` post-hoc is simplest for
one-offs.

## Management & diagnostics

Every `cscli` command works via `docker exec crowdsec cscli …`. The bundled
helper supports this directly:

```bash
~/.claude/skills/crowdsec/scripts/diagnose.sh --env docker --container crowdsec
```

(Verified: detects `Environment: docker`, captures version + the full forensic
support archive from inside the container.)

## Teardown

```bash
docker compose down -v        # -v removes the named config+data volumes
```

## Next step

Run the probes in [../operate/health-check.md](../operate/health-check.md)
(use the `docker exec crowdsec cscli …` row in its per-environment table) before
trusting the deployment.
