# Changelog

All notable changes to this skill are documented here.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

## [0.1.0] - 2026-05-19

### Added
- Initial release of the CrowdSec operational skill for Claude Code.
- `crowdsec/SKILL.md` covering install, configure, operate, and debug flows for
  bare-metal/systemd, Docker, and Kubernetes/Helm.
- Reference docs:
  - `references/install/` — Docker and Kubernetes install notes.
  - `references/operate/` — health check and audit guidance.
  - `references/appsec/` — AppSec/WAF troubleshooting.
  - `references/debug/` — triage and common errors.
- Helper scripts:
  - `scripts/collect-live.sh` — collect live diagnostics from a running engine.
  - `scripts/parse-dump.sh` — parse a collected diagnostic dump.
- Marketplace + plugin manifests under `.claude-plugin/`.

[Unreleased]: https://github.com/buixor/crowdsec-skill/compare/v0.1.0...HEAD
[0.1.0]: https://github.com/buixor/crowdsec-skill/releases/tag/v0.1.0
