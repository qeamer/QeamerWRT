# QeamerWRT — byggeplan & sikkerhets-roadmap

**Eier:** Kent "Qeamer" Nygjerdet
**Mål-maskinvare:** ASUS RT-AC87U (BCM4709, ARMv7 32-bit, Quantenna 5GHz)
**Base:** Asuswrt-Merlin 384.13_10 (siste utgivelse for AC87U, GPL)
**Fokus:** anti-hacking, DDoS, brannmur, DNS, oppdaterte komponenter
*(Ikke WiFi-kryptering / WPA3 — utenfor scope per ønske, og blokkert av lukket driver.)*

---

## 0. Ærlig forventningsstyring

Hva dette prosjektet **kan** levere:
- En herdet, om-brandet firmware med oppdaterte sikkerhetskomponenter i userspace
- Betydelig redusert angrepsflate mot admin-planet og WAN
- DDoS-demping, streng brannmur, kryptert DNS, blokklister
- Ditt eget navn og logo i UI, login og SSH-banner

Hva det **ikke** kan levere (harde vegger på AC87U):
- **Ny Linux-kjerne** — den lukkede Broadcom `wl.ko`-driveren er bundet til kjerne-ABI ~2.6/3.x. Vi sitter på den kjernen.
- **WPA3** — krever driver/kjerne-støtte vi ikke har. (Du bryr deg ikke om dette — greit.)
- **Quantenna 5GHz-fiks** — lukket firmware, ingen kilde. Radioen forblir som den er.

Konsekvens: vi backporter **userspace**-pakker (OpenSSL, dnsmasq, dropbear, curl osv.) og strammer **konfigurasjon** — ikke kjernen.

---

## 1. To spor — velg bevisst

**Spor A — Addon-herding på stock 384.13_10 (anbefalt å gjøre uansett, null brick-risiko).**
Mesteparten av sikkerhetsmålene dine finnes allerede som modne tillegg via `amtm`:
- **Skynet** — IPSet-brannmur, automatiske blokklister, DDoS-/portskann-demping
- **Diversion** — DNS-basert annonse-/malware-blokkering
- **stubby** — DNS-over-TLS (kryptert DNS)
- **scMerlin / scribe** — service-kontroll og syslog

Dette gir deg ~80 % av sikkerhetsgevinsten på en kveld. **Gjør dette først** — det er også et perfekt referansepunkt for hva den egne firmwaren skal innbake permanent.

**Spor B — QeamerWRT egen firmware-build.** Bake herdingen + brandingen permanent inn i et image du flasher. Mer arbeid, brick-risiko, men det er "eierskaps"-prosjektet du er ute etter. Resten av dokumentet dekker dette.

---

## 2. Kildekode & toolchain

```
# Merlin legacy-repo (AC87U ligger her, IKKE i .ng-repoet)
git clone https://github.com/RMerl/asuswrt-merlin.git
cd asuswrt-merlin
git checkout 384.13      # tag/branch for AC87U-basen
```

- Toolchain (kryss-kompilator for ARM) følger med i `tools/` i repoet.
- Byggemiljø: **Ubuntu 18.04/20.04 x86_64** (nyere kan kreve patching av gamle build-script). Bruk en VM eller container — ikke prod-maskinen din.
- Target-katalog for AC87U: `release/src-rt-6.x.4708/` med modellprofil `RT-AC87U`.

```
sudo apt install -y build-essential libtool-bin autoconf automake \
  bison flex gawk gengetopt git gperf nano libncurses5-dev \
  libssl-dev cmake zlib1g-dev unzip uuid-dev xsltproc gettext
cd release/src-rt-6.x.4708 && make rt-ac87u
# Output-image: image/RT-AC87U_3.0.0.4_384.XX_QeamerWRT.trx
```

---

## 3. Branding-integrasjon
Se `branding/README.md`. Kort: kopier logo-PNG-ene fra `assets/logo/` og
`branding/branding.css` inn i `release/src/router/www/`, koble inn stilarket,
sett `motd`/versjonsstreng. Navn + logo vises da i UI, login og SSH.

---

## 4. Sikkerhets-herding — mappet til dine prioriteringer

### 4.1 Anti-hacking (admin-planet — viktigste angrepsflate)
- **Steng admin mot WAN** fullstendig (`misc_http_x=0`, ingen fjern-WebUI).
- **Kun HTTPS-UI**, TLS 1.2+; fjern HTTP-redirect-lekkasjer.
- **Bytt ut Dropbear → oppdatert versjon**, kun nøkkel-basert SSH, slå av passord-login, ikke-standard port.
- **Backport OpenSSL/wolfSSL** til siste støttede gren for å lukke kjente CVE-er i TLS-stacken.
- **Slå av angrepsvektorer:** Telnet AV, WPS AV, UPnP AV, NAT-PMP AV, "Enable web access from WAN" AV.
- **Login-throttling:** innebygd forsøksbegrensning + lockout (mot brute force).
- **Slå av tjenester du ikke bruker:** SMB/Samba, FTP, AiCloud, DDNS hvis ubrukt — hver av disse er en CVE-flate.
- Sett **sterkt unikt admin-passord** og endre standard brukernavn.

### 4.2 DDoS-demping (`sysctl` + iptables)
Bakes inn i `release/src/router/etc/` init-script:
```
net.ipv4.tcp_syncookies = 1
net.ipv4.tcp_max_syn_backlog = 2048
net.ipv4.tcp_synack_retries = 2
net.ipv4.icmp_echo_ignore_broadcasts = 1
net.ipv4.conf.all.rp_filter = 1
net.ipv4.conf.all.accept_redirects = 0
net.ipv4.conf.all.accept_source_route = 0
```
iptables-regler: SYN-rate-limit, ICMP-rate-limit, drop INVALID-pakker,
drop fragmenterte/anomale pakker, conntrack-tuning. (Skynet leverer mye av dette ferdig — kopier logikken inn permanent.)

### 4.3 Brannmur
- **Default-drop** på all uoppfordret inngående WAN-trafikk.
- **IPSet-blokklister** (kjente botnett/skannere/malice-IP) lastet ved boot.
- **Drop INVALID + spoofede** pakker, anti-bogon.
- Valgfri **egress-filtrering** (begrens utgående til kjente porter).
- Logg dropp til syslog for innsyn.

### 4.4 DNS
- **DNS-over-TLS (stubby/unbound)** mot f.eks. Quad9 (9.9.9.9, malware-blokkerende) eller Cloudflare.
- **DNSSEC-validering** i dnsmasq.
- **DNS-rebind-beskyttelse** PÅ (mot lokal-IP-injeksjon).
- **Tving all LAN-DNS gjennom ruteren** (avskjær klienter som hardkoder andre DNS).
- **Blokkliste-basert filtrering** (annonser/malware/telemetri) à la Diversion.
- **Backport dnsmasq** til siste versjon (historisk mange CVE-er — viktig).

### 4.5 Oppdaterte komponenter (firmware-aktualitet)
Backport og rekompiler de mest sårbare pakkene mot nyeste støttede versjon:
`openssl`, `dnsmasq`, `curl`, `wget`, `dropbear`, `busybox`, `nettle`, `zlib`.
Dette er kjernen i "ikke gammel firmware"-følelsen — kjernen forblir gammel,
men de internett-vendte komponentene blir aktuelle.

---

## 5. Test- & recovery-plan (KRITISK — gjør dette før første flash)
1. **Sikkerhetskopier nåværende firmware/NVRAM:** `nvram show > backup.txt`, og dump mtd-partisjoner via SSH før du rører noe.
2. **Skaff USB-TTL seriekabel** (3.3V) og finn UART-pinnene på AC87U — eneste sikre vei tilbake fra en hard brick.
3. **Lær CFE recovery-modus:** hold Reset under oppstart → ASUS Firmware Restoration-verktøyet flasher via TFTP på 192.168.1.1.
4. **Flash først et uendret selvbygget image** for å bekrefte at toolchain-en din lager et bootbart image — *før* du legger til endringer.
5. Inkrementelt: branding → DNS → brannmur → DDoS → pakke-backports. Test og lagre fungerende image mellom hvert steg.

---

## 6. Faseplan / milepæler
| Fase | Leveranse | Brick-risiko |
|------|-----------|--------------|
| 0 | Spor A: Skynet + Diversion + stubby på stock | Ingen |
| 1 | Sett opp byggemiljø, bygg uendret AC87U-image, flash & boot | Lav-middels |
| 2 | Legg inn QeamerWRT-branding, bygg, flash | Lav |
| 3 | Brannmur + DDoS + DNS-herding bakt inn | Middels |
| 4 | Backport openssl/dnsmasq/dropbear/curl | Middels |
| 5 | Frys "QeamerWRT 1.0", dokumenter, publiser GPL-kilde | — |

---

## 7. Min ærlige anbefaling
Start **Spor A i kveld** — du får DDoS-demping, kryptert DNS, blokklister og
brannmur-herding umiddelbart, uten risiko, på den firmwaren du har. Det dekker
nesten hele sikkerhetsmålet ditt. Bygg så **QeamerWRT (Spor B)** som
eierskaps-/hobbyprosjektet det er — der jeg kan være med på kode, patcher,
branding og feilsøking hele veien. Bare ikke gjør AC87U til din
internett-vendte hovedbrannmur på lang sikt; maskinvaren er 11 år og
kjernen kan vi ikke modernisere.
