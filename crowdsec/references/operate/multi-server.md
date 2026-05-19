# Operate — Multi-server / distributed LAPI

Canonical docs: <https://docs.crowdsec.net/docs/next/local_api/intro>

## Sections to fill

> STUB. To cover:
> - Topologies: single LAPI + many agents; HA LAPI behind LB; per-cluster LAPI
> - Registering agents to a remote LAPI (`cscli lapi register`)
> - mTLS between agents and LAPI (cert generation, trust, rotation)
> - Postgres backend for LAPI (when sqlite stops scaling)
> - Bouncer placement: per-agent vs central
> - Cross-LAPI decision sync (Console / CAPI / blocklists API)
> - Pitfalls: clock skew, NAT, agent IDs colliding after image clone (regenerate `machine_id`)
