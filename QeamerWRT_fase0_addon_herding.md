# QeamerWRT — Fase 0: addon-herding på stock firmware

**Brick-risiko: ingen.** Du gjør dette på stock Asuswrt-Merlin 384.13_10, uten å endre
selve firmware-imaget. Alt kan reverseres ved å avinstallere tillegget.

> Gjør dette i kveld. Det gir deg ~80 % av sikkerhetsgevinsten fra hele prosjektet,
> umiddelbart. Bygg QeamerWRT (Fase 1+) parallelt som eierskaps-/hobbyprosjektet.

---

## Del 1 — Web-UI-innstillinger (gjør disse FØRST, ingen SSH nødvendig)

Logg inn på ruteren (http://192.168.1.1) og sett disse:

### Administration → System
- [ ] **Enable Telnet** → **OFF** (Telnet er ukryptert, aldri bruk)
- [ ] **Enable SSH** → **LAN only** (ikke WAN)
- [ ] **Allow password login** for SSH → kun nøkkel hvis du har satt opp det
- [ ] **HTTPS only** → ON (ingen HTTP)
- [ ] Admin-passord: bytt til sterkt unikt passord hvis ikke gjort

### WAN → Internet Connection
- [ ] **Enable UPnP** → **OFF**
- [ ] **Enable NAT-PMP** → **OFF**

### Administration → System → WAN access
- [ ] **Enable web access from WAN** → **OFF**

### Wireless → Professional (per bånd)
- [ ] **Enable WPS** → **OFF** (begge bånd)

### Administration → System → Miscellaneous
- [ ] Slå av tjenester du ikke bruker: AiCloud, SMB/Samba, FTP, DDNS (hvis ubrukt)

---

## Del 2 — amtm-tillegg via SSH

> **Forutsetning — USB-pinne kreves.** amtm, Skynet, Diversion og stubby bruker
> Entware for lagring. Entware krever en USB-minnepinne (≥ 8 GB, ext4) koblet til
> ruteren. Plugg inn pinnen **før** du starter amtm, og la amtm sette opp Entware
> som første steg (velg **ep** i amtm-menyen). Uten USB-pinne feiler installasjonen.

SSH inn på ruteren:

```bash
ssh admin@192.168.1.1
```

### Installer amtm og Entware (gjør dette først)
```bash
curl -Os https://raw.githubusercontent.com/decoderman/amtm/master/amtm && sh amtm
# Velg 'ep' i menyen for å sette opp Entware på USB-pinnen
```

---

### 2a — Skynet (IPSet-brannmur + DDoS-demping + blokklister)

Fra amtm-menyen: velg **sk**

Skynet gir deg:
- Default-drop på all uoppfordret inngående WAN-trafikk
- Automatiske blokklister (kjente botnett, skannere, malice-IP) lastet ved boot
- SYN-flood-rate-limiting
- Portskann-deteksjon og automatisk banning
- Logg til syslog for innsyn

Etter installasjon, verifiser:
```bash
firewall status
```
Skal vise aktivt antall blokkerte IP-er og regler.

---

### 2b — Diversion (DNS-basert annonse- og malware-blokkering)

Fra amtm-menyen: velg **di**

Diversion bruker dnsmasq (allerede på ruteren) til å blokkere kjente
annonse-/malware-/telemetri-domener for alle enheter på LAN-et.

Anbefalt konfigurasjon under installasjon:
- Blokkliste: **Steven Black** (StevenBlack/hosts — stor, godt vedlikeholdt)
- Update schedule: daglig

---

### 2c — stubby (DNS-over-TLS — kryptert DNS)

Fra amtm-menyen: velg **st**

stubby sender alle DNS-forespørsler kryptert til en upstream-resolver du velger.
Hindrer ISP og mellommenn fra å se hvilke domener du slår opp.

Anbefalte resolvers under installasjon:
- **Quad9** (9.9.9.9 / 9.9.9.9#dns.quad9.net) — blokkerer kjent malware
- eller **Cloudflare** (1.1.1.1 / one.one.one.one)

Etter installasjon, verifiser:
```bash
ps | grep stubby              # skal vise stubby-prosessen som kjører
nslookup google.com           # skal svare uten feil (DNS fungerer)
```

---

### 2d — scMerlin + scribe (valgfritt — service-kontroll og logging)

Fra amtm-menyen: velg **sc** og **sb**

Gir bedre kontroll over hvilke tjenester som kjører og lagrer logg lokalt.
Nyttig for å oppdage anomalier over tid.

---

## Del 3 — Verifisering etter installasjon

Kjør dette fra SSH for å sjekke at alt er på plass:

```bash
# Skynet aktiv?
firewall status | head -5

# DNS-over-TLS aktiv?
ps | grep stubby

# Hvilke porter lytter ruteren på? (bør være minimalt)
netstat -tlnp

# SSH-banneret ditt (verifiser ingen uønskede tjenester)
cat /etc/motd

# OpenSSL-versjon (baseline å sammenligne med etter Fase 4)
openssl version

# dnsmasq-versjon (baseline)
dnsmasq --version | head -1

# Dropbear SSH-versjon (baseline)
dropbear -V 2>&1 | head -1
```

Noter versjonene — dette er **baseline** du skal slå med Fase 4-backportene.

---

## Del 4 — Ekstra sysctl-herding (manuell, valgfri)

> **Forutsetning:** JFFS custom scripts må være aktivert.
> Gå til **Administration → System → Enable JFFS custom scripts and configs → ON**
> og lagre før du oppretter skriptet under.

Disse kan legges inn i `/jffs/scripts/firewall-start` slik at de settes ved boot:

```bash
#!/bin/sh
# QeamerWRT Fase 0 — sysctl-herding
sysctl -w net.ipv4.tcp_syncookies=1
sysctl -w net.ipv4.tcp_max_syn_backlog=2048
sysctl -w net.ipv4.tcp_synack_retries=2
sysctl -w net.ipv4.icmp_echo_ignore_broadcasts=1
sysctl -w net.ipv4.conf.all.rp_filter=1
sysctl -w net.ipv4.conf.all.accept_redirects=0
sysctl -w net.ipv4.conf.all.accept_source_route=0
sysctl -w net.ipv4.conf.all.log_martians=1
```

```bash
# Gjør skriptet kjørbart
chmod +x /jffs/scripts/firewall-start

# Kjør det nå (uten å reboot)
sh /jffs/scripts/firewall-start
```

> Skynet setter allerede mange av disse — sjekk at du ikke duplikerer dem.
> Skynet sin konfigurasjon er fasit; disse er supplement der Skynet ikke dekker.

---

## Sjekkliste — Fase 0 fullført

- [ ] Telnet OFF, SSH kun LAN
- [ ] HTTPS-only WebUI, WAN-tilgang OFF
- [ ] UPnP OFF, NAT-PMP OFF, WPS OFF
- [ ] Sterkt admin-passord satt
- [ ] Skynet installert og aktiv (blokklister lastet)
- [ ] Diversion installert (DNS-blokkering aktiv)
- [ ] stubby installert (DNS-over-TLS verifisert)
- [ ] Baseline-versjoner notert (openssl, dnsmasq, dropbear)
- [ ] sysctl-herding i firewall-start (valgfritt)

Når alle er huket av: du har betydelig bedre sikkerhet enn 99 % av hjemmerutere
på markedet, uten å ha risikert noe. Gå videre til Fase 1 (byggemiljø) i eget tempo.
