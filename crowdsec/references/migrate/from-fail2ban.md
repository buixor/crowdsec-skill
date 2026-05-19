# Migrate — From fail2ban to CrowdSec

Canonical docs: <https://docs.crowdsec.net/> (search "fail2ban")

> STUB. To cover:
> - Concept map: jail → scenario; filter → parser; action → bouncer; banaction → bouncer backend; findtime/maxretry → leaky bucket params
> - Side-by-side install: keep fail2ban active in observe-only; run CrowdSec in simulation
> - Which fail2ban jails map to existing CrowdSec hub collections (sshd, apache-auth, etc.)
> - Cutover checklist: disable fail2ban jails progressively as CrowdSec equivalents prove they fire
> - Cleanup: remove fail2ban iptables chains before enabling crowdsec-firewall-bouncer to avoid double-blocking / conflicting chains
> - Common gotchas: log rotation differences (CrowdSec re-opens on rename), persistent bans (fail2ban dbfile → CrowdSec decisions in LAPI)
