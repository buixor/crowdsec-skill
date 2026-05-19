# Operate — Upgrades, backup, rollback

Canonical docs: <https://docs.crowdsec.net/docs/next/configuration/crowdsec_configuration> · `cscli` reference <https://docs.crowdsec.net/docs/next/cscli/>

> STUB. To cover:
> - Pre-upgrade: backup `/var/lib/crowdsec/data/` (LAPI sqlite/postgres) and `/etc/crowdsec/`
> - Per-env upgrade flow:
>   - bare-metal: `apt upgrade crowdsec` + restart; check `cscli version`
>   - docker: pull new tag, recreate container with same volumes
>   - k8s: helm upgrade with `--reset-then-reuse-values`
> - Hub upgrade: `cscli hub upgrade`
> - Bouncer upgrades (separate package per bouncer)
> - Rollback procedure (snapshot, package downgrade, restore DB)
> - Breaking-change checklist between minor versions (link to release notes)
