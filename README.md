# CrowdSec skill for Claude Code

An operational [Claude Code skill](https://docs.claude.com/en/docs/claude-code/skills)
for installing, configuring, operating, and debugging
[CrowdSec](https://www.crowdsec.net) — `cscli`, LAPI/CAPI, hub collections,
parsers/scenarios/whitelists deployment, bouncers (firewall, nginx, traefik,
caddy), the WAF (AppSec component), profiles, notifications, upgrades, and
fail2ban migration. Covers bare-metal/systemd, Docker, and Kubernetes/Helm.

> **Scope:** this is an *operational* skill. It does **not** author WAF rules,
> scenarios, or parsers — only deploys, configures, and debugs them.

## Install

```text
/plugin marketplace add buixor/crowdsec-skill
/plugin install crowdsec@buixor
```

Update later with:

```text
/plugin marketplace update buixor
```

## What it does

Once installed, Claude Code automatically loads this skill when your prompt
involves CrowdSec operations. Typical triggers:

- "Install CrowdSec on this server" / "deploy CrowdSec in my Kubernetes cluster"
- "Set up the nginx / traefik / caddy / firewall bouncer"
- "Enable the WAF / AppSec component"
- "Enroll this engine in the Console"
- "Migrate from fail2ban"
- "Logs aren't being parsed" / "no alerts are firing" / "bouncer isn't blocking"

## Repository layout

```
crowdsec-skill/
├── .claude-plugin/         # marketplace + plugin manifests
├── crowdsec/
│   ├── SKILL.md            # skill entry point (auto-loaded by Claude Code)
│   ├── references/         # topic-specific reference docs
│   └── scripts/            # collection and parsing helpers
└── CHANGELOG.md
```

## License

MIT — see [LICENSE](LICENSE).

## Links

- CrowdSec: https://www.crowdsec.net
- Documentation: https://docs.crowdsec.net
- Hub: https://hub.crowdsec.net
- Console: https://app.crowdsec.net
