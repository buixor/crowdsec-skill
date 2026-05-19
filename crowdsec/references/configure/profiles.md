# Configure — Profiles (decisions, durations, simulation)

Canonical docs: <https://docs.crowdsec.net/docs/next/local_api/profiles> · post-install profiles <https://docs.crowdsec.net/docs/next/getting_started/post_installation/profiles>

> STUB. To cover:
> - `profiles.yaml` structure (filters, decisions, on_success)
> - Decision types: ban / captcha / throttle
> - Duration syntax + escalation patterns
> - Simulation mode for safe rollout (`cscli simulation enable`)
> - Notification triggers from profiles
> - Interaction with allowlists: even if a profile matches, an allowlisted target IP causes the decision to be silently dropped at LAPI write time. To exempt specific IPs/ranges, prefer an allowlist (see [allowlists.md](./allowlists.md)) over a profile filter expression.
> - Pitfalls: filter expression typos silently no-op; profile order matters
