# Trusselbilde — hjemme-/SOHO-rutere (tilpasset Kent / AC87U)

Hva som faktisk angriper en hjemmeruter på internett i dag, hvorfor det treffer
deg spesielt på gammel maskinvare, og hvordan QeamerWRT-herdingen demper hver
trussel. Sortert etter hvor sannsynlig det er at det rammer *deg*.

---

## 1. Botnett-rekruttering (Mirai-varianter og etterkommere)
**Type:** Automatiserte ormer som skanner hele internett etter rutere med åpne
admin-tjenester (Telnet/SSH/WebUI mot WAN) og kjente sårbarheter, logger inn med
standard-/svake passord eller exploit, og verver enheten inn i et DDoS-botnett.
**Hvorfor du:** Eldre Broadcom-rutere er primærmål — Mirai-familien har innebygde
exploits mot nettopp ASUS/Broadcom-modeller. En AC87U med eksponert admin er lavthengende frukt.
**Konsekvens:** Ruteren din brukes til DDoS-angrep mot andre, trafikken din avlyttes/omdirigeres, ustabil ytelse, og du kan svartelistes hos ISP.
**Demping (QeamerWRT):** Ingen admin mot WAN, Telnet av, kun nøkkel-SSH på ikke-standard port, default-drop brannmur. Botnett finner da ingen dør å banke på.

## 2. DNS-hijacking / cache-poisoning
**Type:** Angriper endrer ruterens DNS-innstillinger (via CSRF, svakt passord
eller en dnsmasq-sårbarhet) så all nettlesing dirigeres gjennom ondsinnede
servere — phishing-sider, falske bank-innlogginger, annonse-injeksjon.
**Hvorfor du:** 384.13_10 kjører **dnsmasq 2.81**, sårbar for DNSpooq-klyngen (cache-poisoning). Dette er den mest konkrete tekniske svakheten din akkurat nå.
**Konsekvens:** Usynlig omdirigering av *all* nettrafikk i husholdningen. Vanskelig å oppdage.
**Demping:** Backport dnsmasq ≥ 2.90 (lukker DNSpooq), DNS-over-TLS (stubby), DNSSEC-validering, DNS-rebind-vern, og tving all LAN-DNS gjennom ruteren.

## 3. Eksponert admin-panel / fjernadministrasjon
**Type:** WebUI eller SSH tilgjengelig fra internett — enten bevisst påslått
"fjernadgang" eller via UPnP-port-åpning. Angripes med brute force eller CSRF.
**Hvorfor du:** Mange lar "Enable access from WAN" eller AiCloud stå på uten å vite det. Hver eksponert tjeneste er en CVE-flate på en gammel kjerne.
**Konsekvens:** Full overtakelse av ruteren.
**Demping:** Alt admin kun fra LAN, HTTPS-only, AiCloud/FTP/Samba av hvis ubrukt, login-throttling.

## 4. UPnP- og NAT-PMP-misbruk
**Type:** Skadevare på en PC/IoT-enhet i nettet ber ruteren åpne porter inn mot
seg selv via UPnP — uten din viten. Eksponerer interne enheter direkte mot nettet.
**Konsekvens:** IoT-kameraer, NAS, PC-er blir nåbare utenfra.
**Demping:** UPnP og NAT-PMP **av** som standard i QeamerWRT; åpne porter manuelt og bevisst hvis nødvendig.

## 5. Utdaterte krypto-/TLS-komponenter
**Type:** Angrep mot kjente sårbarheter i gamle biblioteker (OpenSSL 1.0.2 er EOL,
curl/busybox/nettle med CVE-er). Kan utnyttes via ondsinnede sertifikater eller
manipulerte svar fra tjenester ruteren snakker med.
**Konsekvens:** DoS av ruteren, i verste fall kjøring av kode i en tjeneste.
**Demping:** Backport-lista (openssl 1.1.1w, curl 8.x, nettle 3.10, busybox, zlib).

## 6. CSRF mot WebUI fra en fane i nettleseren
**Type:** Du besøker en ond nettside mens du er innlogget i ruteren; siden sender
skjulte forespørsler som endrer DNS, åpner porter eller bytter passord.
**Demping:** Logg alltid ut av WebUI, HTTPS-only, ikke-standard admin-port, og hold WebUI utilgjengelig fra WAN.

## 7. IoT-enheter som svakt ledd
**Type:** Ikke ruteren selv, men en billig IoT-dings (kamera, smartplugg) med hull
som blir inngangsport til nettet ditt, deretter sidebevegelse.
**Demping:** Sett IoT på **gjeste-/isolert nett** (VLAN-aktig isolasjon i Merlin), blokker deres internett-tilgang der det går.

---

## Prioritert tiltaksrekkefølge for deg
1. Steng admin mot WAN + slå av UPnP/Telnet/ubrukte tjenester *(i dag, på stock)*
2. Kryptert DNS + dnsmasq-backport *(DNSpooq er din mest konkrete svakhet)*
3. Default-drop brannmur + blokklister (Skynet → bakt inn i QeamerWRT)
4. Backport TLS-stacken
5. Isoler IoT på eget nett

> Rød tråd: nesten alt over handler om **eksponerte tjenester** og **utdatert
> userspace** — begge deler kan du fjerne. Ingen av de alvorlige truslene krever
> at du fikser kjernen eller WiFi-blobbene.
