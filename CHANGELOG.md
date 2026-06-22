# Changelog — QeamerWRT

Format: dato · type · komponent fra→til · CVE/tiltak · kilde+checksum.
Hold denne oppdatert ved hver merge til `main`.

## [Uutgitt]
### Planlagt (se docs/backport-targets.md)
- security: dnsmasq 2.81 → ≥2.90 (lukker DNSpooq CVE-2020-25681..25687)
- security: openssl 1.0.2u (EOL) → 1.1.1w (CVE-2022-0778, 2021-3711 m.fl.)
- security: dropbear 2020.80 → ≥2022.83 (CVE-2021-36369)
- security: curl ~7.66 → 8.x
- security: busybox → nyere (CVE-2021-42373..42386, 2022-28391)
- security: nettle → 3.10.x (CVE-2021-3580)
- security: zlib → ≥1.2.12 (CVE-2022-37434)
- feat: QeamerWRT-branding (logo + Kent "Qeamer" Nygjerdet i UI/login/SSH)
- feat: default-drop brannmur + IPSet-blokklister + sysctl-herding
- feat: DNS-over-TLS (stubby) + DNSSEC + rebind-vern

## [0.0.0] — prosjektstart
- docs: roadmap, trusselbilde, konsekvensvurdering, sikkerhetspolicy, backport-mål, fase 1, branding, AGENTS.md
