# QeamerWRT — project map

One place to see everything.

## Start here
| File | What it is |
|------|-----------|
| [README.md](README.md) | Project overview, badges, security table, phase plan |
| [guides/phase0-hardening.md](guides/phase0-hardening.md) | **Phase 0 — do tonight.** No-risk hardening on stock firmware |

## Plan & security
| File | What it is |
|------|-----------|
| [docs/roadmap.md](docs/roadmap.md) | Master plan: two tracks, phases, recovery |
| [docs/security/threats.md](docs/security/threats.md) | What actually attacks home routers |
| [docs/security/risk-assessment.md](docs/security/risk-assessment.md) | Risk, likelihood, residual risk |
| [docs/security/policy.md](docs/security/policy.md) | The security standard this project follows |

## Build (the QeamerWRT firmware itself)
| File | What it is |
|------|-----------|
| [guides/phase1-build.md](guides/phase1-build.md) | Phase 1 — build the first unmodified image |
| [docs/backport-targets.md](docs/backport-targets.md) | Component list, target versions, CVEs closed |

## Branding & UI
| File | What it is |
|------|-----------|
| [branding/README.md](branding/README.md) | How to bake logo + name into the UI |
| [ui-theme/](ui-theme/) | Visual theme matching the logo (`theme.css`, `preview.html`) |
| [assets/logo/](assets/logo/) | SVG and PNG logo files |

## Project meta
| File | What it is |
|------|-----------|
| [AGENTS.md](AGENTS.md) | Hard rules for AI assistants — read before AI edits |
| [CONTRIBUTING.md](CONTRIBUTING.md) | How to contribute + pre-push checklist |
| [NOTICE.md](NOTICE.md) | Upstream copyright attribution (GPL) |
| [LICENSE](LICENSE) | GPL-2.0-only |
| [CHANGELOG.md](CHANGELOG.md) | Version history |
| [docs/cursor-prompt.md](docs/cursor-prompt.md) | Opening prompt for Cursor |

---

### Where am I in the project?
1. **Tonight:** Phase 0 — stock-firmware hardening (USB stick → amtm → Skynet/Diversion/stubby). Zero risk.
2. **Next:** Phase 1 — set up the build environment, compile one unmodified image. Cursor drives; you flash.
3. **Then:** Phases 2–5 — branding, firewall/DNS hardening, backports, freeze 1.0.

> Not sure about something? You don't need to understand the low-level parts —
> follow the phase you're on, and ask when stuck.
