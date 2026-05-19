# Bouncers — Web servers (nginx, Traefik, Caddy)

Canonical docs: <https://docs.crowdsec.net/u/bouncers/intro> (per-bouncer pages: nginx, traefik, caddy)

> STUB. To cover:
> - nginx: install module/plugin, config snippet, stream-mode
> - Traefik: middleware plugin (Yaegi) + traefik-bouncer container
> - Caddy: caddy-crowdsec-bouncer module
> - Captcha vs ban behavior per bouncer
> - Bouncer key generation and rotation
> - Verification: curl a banned IP through the web server, expect 403/captcha
> - Pitfalls: plugin/module version skew with web server; cache layer in front; X-Forwarded-For trust
