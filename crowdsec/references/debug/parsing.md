# Debug — Logs not being parsed

Canonical docs: <https://docs.crowdsec.net/docs/next/troubleshooting/intro> · `cscli metrics` <https://docs.crowdsec.net/docs/next/observability/cscli_metrics>

Commands below are written for **bare-metal** (`sudo cscli …`). In docker,
prefix with `docker exec <name>`; in k8s, `kubectl exec -n <ns> <pod> --`.

The single diagnostic that answers "is parsing working" is **`cscli metrics
show acquisition`**. It prints,
per source: Lines read / parsed / unparsed / poured to bucket / whitelisted.

## Read the table first

Verified example from a live box:

```
| Source                 | Lines read | Lines parsed | Lines unparsed |
| appsec:appsec          | 4          | 4            | -              |
| file:/var/log/auth.log | 49         | -            | 49             |
| file:/var/log/syslog   | 1          | -            | 1              |
```

Map the symptom to the cause:

| What the row shows | Meaning | Where to look |
|---|---|---|
| **Source absent entirely** | acquisition never matched the file/journald unit | acquisition config / perms (below) |
| **0 lines read** | source matched but nothing arrives — file empty, wrong path, journald perms, container can't see the path | reachability (below) |
| **Lines read, 0 parsed, all unparsed** | lines arrive but no parser claims them | **usually normal noise** — or a `type:` mismatch (below) |
| **Read = parsed** | parsing fine; if no alerts the problem is downstream → [no-alerts.md](./no-alerts.md) |

The 49/49 row above is the typical case — `auth.log` carries `sudo`/`cron`/
`systemd-logind` lines no parser claims. Treat unparsed as a fault only when
**lines you know should match** (a real `sshd: Failed password …`, an nginx
access line) don't move the parsed counter.

## Confirm with `cscli explain` (read-only, no traffic needed)

This is the fastest way to prove a parser claims a line and which `type:` it
needs. Verified:

```bash
LINE='May 18 10:00:00 host sshd[123]: Failed password for root from 203.0.113.5 port 22 ssh2'

sudo cscli explain --log "$LINE" --type syslog     # → green ssh-bf scenarios
sudo cscli explain --log "$LINE" --type nginx      # → 🔴 parser failure
```

Same line, different `--type`: `syslog` parses to the ssh scenarios; `nginx` is
a hard parser failure. **The `type:` label in your acquisition must match the
parser family**, exactly as `--type` does here. Use `--file <path>` to replay a
whole log file; add `--only-successful-parsers` when replaying a whole file to
hide the long list of parsers that legitimately reject each line and surface
only the ones that matched (the default output is very noisy on a real log).

## The #1 real cause: `type:` label vs installed parser

Acquisition files set `labels: { type: <x> }`. A parser only runs on events
whose `type` it expects. Mismatch = lines read, 0 parsed.

```bash
sudo grep -r 'type:' /etc/crowdsec/acquis.d/        # what you declared
sudo cscli parsers list                              # what's installed/enabled
```

- nginx/apache access logs → need `crowdsecurity/nginx` / `apache2` collection,
  acquisition `type: nginx` (or the right syslog program tag).
- sshd → `crowdsecurity/sshd`, `type: syslog` reading `/var/log/auth.log`.
- A source with `type: syslog` but only file-format lines (no syslog prefix)
  won't match the syslog parser — check with `cscli explain`.

## Reachability (when 0 lines read)

- **File perms**: the `crowdsec` user must read the file. `sudo -u crowdsec head
  <path>` — if that fails, it's perms (or SELinux/AppArmor — see
  [common-errors.md](./common-errors.md)).
- **journald**: source must be in a group that can read the journal; a
  file-source migration to journald silently reads nothing otherwise.
- **Containers/k8s**: the log path must be *mounted into* the crowdsec
  container/pod. A path that exists on the host but not in the container reads
  zero. Verify inside: `docker exec crowdsec ls -l <path>` /
  `kubectl exec -n <ns> <pod> -- ls -l <path>`.

## Multi-stage chains

Parsing is staged (`s00` → `s01` → `s02-enrich`). A line can be parsed at s01
then dropped/whitelisted at `s02-enrich` (e.g. `crowdsecurity/whitelists`,
verified at `/etc/crowdsec/parsers/s02-enrich/whitelists.yaml`). If "parsed" is
non-zero but "whitelisted" is also non-zero and alerts are missing, the issue is
a whitelist, not parsing → [no-alerts.md](./no-alerts.md).

## Fix path

- Missing parser for a real source → install the hub collection
  (`cscli collections install crowdsecurity/<svc>`), then
  `sudo systemctl reload crowdsec`. Authoring a custom parser is **out of scope
  for this skill**.
- Wrong `type:` → correct the acquisition file, reload, re-check
  `cscli metrics show acquisition`.
