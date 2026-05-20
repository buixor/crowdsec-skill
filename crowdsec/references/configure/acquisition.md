# Configure — Acquisition (log sources)

Canonical docs: <https://docs.crowdsec.net/docs/next/getting_started/post_installation/acquisition> · datasources index <https://docs.crowdsec.net/docs/next/data_sources/intro>

> STUB. To cover:
> - `acquis.yaml` vs. `acquis.d/*.yaml`
> - File datasource (paths, type/labels, multi-file globs)
> - journald datasource (filters, units)
> - syslog, kinesis, k8s_audit, docker, AppSec — when to pick each
> - Verify a source after editing: `crowdsec -t` (validate config), `cscli metrics show acquisition` (confirm it's read), `cscli explain` (confirm a line parses)
> - Common pitfalls: missing `type:` label (parser won't match), permission denied on log files, journald unit filter typos
