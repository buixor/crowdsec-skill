# Configure — Notifications (http/email/slack)

Canonical docs: <https://docs.crowdsec.net/docs/next/local_api/notification_plugins/intro>

> STUB. To cover:
> - Plugin layout: `/etc/crowdsec/notifications/*.yaml` + binary in `/usr/lib/crowdsec/plugins/`
> - Wiring a plugin into a profile (`notifications:`)
> - Templating with go-template (alert/decision fields)
> - HTTP: webhook patterns (Slack incoming, Discord, generic)
> - Email: SMTP, auth, TLS
> - Testing a plugin: `cscli notifications test`
> - Pitfalls: plugin binary missing exec bit; templating errors swallow notification silently
