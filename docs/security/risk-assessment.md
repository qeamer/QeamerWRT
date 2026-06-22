# Konsekvensvurdering — AC87U / QeamerWRT som hovedruter

**System:** ASUS RT-AC87U, brukt som hovedruter/brannmur for ett hjemmenettverk
**Bruker:** Kent "Qeamer" Nygjerdet (én administrator, lavt trusselnivå-miljø)
**Firmware:** stock 384.13_10 i dag → planlagt QeamerWRT (herdet fork)

Formål: gi deg et nøkternt bilde av hva du faktisk risikerer, hvor sannsynlig
det er, og hvor mye QeamerWRT flytter nålen — så valget om å beholde AC87U som
hovedruter er informert.

---

## 1. Sårbarheter

| # | Sårbarhet | Kan patches? | Restrisiko etter QeamerWRT |
|---|-----------|--------------|----------------------------|
| V1 | Gammel Linux-kjerne (~2.6/3.x), lukket Broadcom ABI | ❌ Nei | Forblir — men sjelden nåbar bak default-drop |
| V2 | dnsmasq 2.81 (DNSpooq) | ✅ Ja (backport ≥2.90) | Eliminert |
| V3 | OpenSSL 1.0.2u EOL | ✅ Ja (→1.1.1w) | Eliminert |
| V4 | Eksponerte tjenester (WAN-admin, UPnP, AiCloud, Telnet) | ✅ Ja (konfig) | Eliminert hvis stengt |
| V5 | curl/busybox/nettle/zlib CVE-er | ✅ Ja (backport) | Sterkt redusert |
| V6 | Quantenna 5GHz lukket firmware | ❌ Nei | Forblir (men du bruker ikke 5GHz-sikkerhet som mål) |
| V7 | Ingen leverandør-sikkerhetspatcher fremover | ⚠️ Delvis (du blir leverandøren) | Avhenger av at du holder backports ved like |

## 2. Sannsynlighet for angrep

- **Automatiserte, ikke-målrettede angrep (botnett, masseskann): HØY og kontinuerlig.** Hele internett skannes døgnet rundt. Dette er den realistiske trusselen mot et hjem.
- **Målrettet angrep mot deg spesifikt: LAV.** Du beskriver et lavt-trussel-miljø uten spesielle motstandere. Ingen bruker en 0-dag på en privatpersons ruter.

Konklusjon: risikoen din er nesten utelukkende **automatisert og opportunistisk**
— og den typen angrep stoppes effektivt av nettopp det QeamerWRT gjør (lukke
eksponering + oppdatere userspace). Det er gode nyheter for planen din.

## 3. Konsekvens ved vellykket angrep

- **Konfidensialitet:** Avlytting/omdirigering av trafikk (særlig via DNS-hijack). Middels-høy konsekvens — bank, e-post, passord kan eksponeres.
- **Integritet:** Manipulert DNS/innhold, injiserte annonser, endrede ruter-innstillinger.
- **Tilgjengelighet:** Ruteren vervet til DDoS-botnett → ustabilt nett, ISP-svartelisting.
- **Sidebevegelse:** Kompromittert ruter = ståsted for å angripe alle enheter i hjemmet (PC, NAS, IoT).
- **Omdømme/økonomi:** For en privatperson lavt, men identitetstyveri/banksvindel via DNS-hijack er den reelle økonomiske faren.

## 4. Risikomatrise (etter QeamerWRT-herding)

| Trussel | Sannsynlighet | Konsekvens | Restrisiko |
|---------|---------------|------------|-----------|
| Botnett-rekruttering | Lav (lukket eksponering) | Høy | **Lav** |
| DNS-hijack | Lav (DoT+backport) | Høy | **Lav** |
| Eksponert admin | Svært lav (kun LAN) | Høy | **Svært lav** |
| Kjerne-0-dag utenfra | Svært lav (ikke nåbar) | Høy | **Lav** |
| IoT-svakt-ledd | Middels | Middels | **Lav-middels** (isoler IoT) |

## 5. Samlet vurdering

QeamerWRT, riktig konfigurert, flytter AC87U fra **"utdatert og delvis eksponert"**
til **"herdet, lukket angrepsflate, oppdatert userspace"**. Mot den realistiske
trusselen din — automatiserte masseskann — er det en stor og reell forbedring.

**Den ærlige restrisikoen** ligger i V1 (kjernen) og V7 (du må selv vedlikeholde
backports). Så lenge ruteren ikke eksponerer noe inn og du holder userspace
oppdatert, er det et forsvarlig valg å bruke den som hovedruter i et lavt-trussel-hjem.

**Anbefalt grense:** ikke bruk den i et høyrisikomiljø (hjemmekontor med
sensitive bedriftsdata, kjente motstandere). For vanlig privat bruk: ja, med
herdingen på plass. Vurder utskifting den dagen kravene dine endrer seg.
