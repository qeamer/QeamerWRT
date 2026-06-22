# QeamerWRT — backport-mål (komponentliste)

**Base:** Asuswrt-Merlin 384.13_10 (siste AC87U-build, juni 2020)
**Kilde å låne fra:** 388.x / 3004.388.x (asuswrt-merlin.ng) og oppstrøms-prosjektene
**Regel:** lån *kildekode + patcher*, rekompiler for ARMv7. Aldri kopier binærfiler.

Prioritert etter internett-eksponering: DNS og TLS-stacken først (de snakker
direkte med nettet), deretter SSH og resten.

---

## P0 — kritisk, gjør først

### dnsmasq  `2.81` → `≥ 2.90` (helst 2.91)
Den viktigste enkeltbackporten. 384.13_10 sin 2.81 er sårbar for **DNSpooq**-klyngen.
- Lukker: **CVE-2020-25681 … 25687** (DNSpooq — cache-poisoning + heap buffer overflow, fikset i 2.83), pluss senere DNSSEC-/parsing-fikser.
- Konsekvens i dag: DNS-cache-forgiftning er en reell ekstern angrepsvektor mot en hovedruter. Dette er grunnen til at DNS står øverst.
- Hentes fra: 388.x bruker 2.90/2.91 — løft kilden direkte.

### openssl  `1.0.2u` (+ `1.1.1g`) → `1.1.1w`
1.0.2-grenen er **EOL siden 2019** — null sikkerhetsstøtte. Dropp den helt; bygg alt mot 1.1.1w (siste 1.1.1).
- Lukker bl.a.: **CVE-2022-0778** (BN_mod_sqrt uendelig løkke → DoS via ondt sertifikat), **CVE-2021-3711** (SM2-dekrypterings-buffer-overflow), **CVE-2021-3712** (ASN.1-streng-overlesing), **CVE-2023-0286** (X.400 type-forveksling).
- Merknad: noen gamle AC87U-komponenter er lenket mot 1.0.2 — disse må rekompileres mot 1.1.1. Hovedjobben i hele prosjektet.

---

## P1 — høy

### dropbear (SSH)  `2020.80` → `≥ 2022.83` (helst 2025.x)
- Lukker: **CVE-2021-36369** (svakhet i auth-håndtering) m.fl.
- Ta samtidig med herdet cipher/kex-liste fra nyere bygg (fjern svake algoritmer).

### curl / libcurl  `~7.66` → `8.x`
- 2020→2026 er dusinvis av CVE-er i URL-parsing, TLS og protokollhåndtering.
- Brukes av oppdaterings-/DDNS-/AiCloud-funksjoner — verdt å holde aktuell.

### busybox  (gammel) → nyere stabil
- Lukker bl.a.: **CVE-2021-42373…42386** (DoS/use-after-free-klynge), **CVE-2022-28391** (terminal-escape-injeksjon via `ip`/`telnet`-output).

---

## P2 — middels

### nettle  → `3.10.x`
- Lukker: **CVE-2021-3580** (RSA-PSS verifisering DoS) m.fl. Brukes av GnuTLS/DNSSEC-stien.

### CA-rotbunt (`cacert.pem`)  → fersk Mozilla-bunt
- Ingen CVE, men gamle/utgåtte rot-sertifikater = stille TLS-svakhet. Erstatt med dagens bunt fra 388.x.

### zlib  → `≥ 1.2.12`
- Lukker: **CVE-2022-37434** (heap-overlesing i inflate via ondt gzip-felt).

---

## Konfigurasjon å "stjele" (ikke pakker, men defaults fra 388.x)
- **DNS-over-TLS / stubby**-integrasjon og GUI-plumbingen.
- **DNSSEC**-validering på som standard + **DNS-rebind-vern**.
- Hardere **sysctl**-defaults (syncookies, rp_filter, ignore broadcasts).
- Nyere **brannmur-/iptables-logikk** (drop INVALID, rate-limit).
- **HTTPS-only WebUI** med TLS 1.2+ og moderne cipher-liste.

---

## Arbeidsflyt per backport
1. `git diff` pakken mellom 384.13-tag og 388.x-tag i .ng-repoet.
2. Hent oppstrøms-tarball for målversjonen, legg ASUS/Merlin sine patcher oppå.
3. Rekompiler mot openssl 1.1.1w for ARMv7; løs lenke-/ABI-feil.
4. Test isolert (Spor A-ruter eller chroot) før det går inn i firmware-imaget.
5. Verifiser versjon på enheten: `dnsmasq --version`, `openssl version`, `dropbear -V`.

---

## Realitetssjekk
Dette moderniserer hele den **internett-vendte userspace-stacken** — som er der
nesten alle praktiske angrep mot hjemmerutere lander. Kjernen og WiFi-blobbene
forblir gamle og kan ikke patches; derfor: ingen åpne porter inn, default-drop
brannmur, og ikke regn den som like sikker som en støttet 2026-ruter — men klart
sikrere enn 384.13_10 slik den står i dag.

*Versjonsnumre per Merlin-changelog (388.x / 3006) og oppstrøms; verifiser nyeste
stabile før bygg.*
