# Sikkerhetspolicy — Kent "Qeamer" Nygjerdet sitt hjemmenettverk

En praktisk standard du faktisk kan følge og holde deg til. Ikke en bedrifts-
policy med jus og roller — en personlig sjekkliste-standard for ett hjem, én
administrator, AC87U/QeamerWRT som ruter.

**Versjon:** 1.0 · **Eier/admin:** Kent "Qeamer" Nygjerdet · **Gjennomgås:** hver 6. mnd

---

## 1. Prinsipper
1. **Lukk alt inn som standard.** Ingen tjeneste eksponeres mot internett uten en bevisst, dokumentert grunn.
2. **Hold det som snakker med nettet oppdatert.** DNS- og TLS-komponenter har førsteprioritet.
3. **Minste privilegium.** Tjenester og enheter får bare den tilgangen de faktisk trenger.
4. **Anta at gammelt = sårbart.** Kjernen kan ikke fikses, så kompensér med konfigurasjon.

## 2. Ruter-administrasjon
- Admin-WebUI: **kun fra LAN**, **HTTPS-only**, ikke-standard port.
- SSH: **kun nøkkel-basert**, passord-login av, ikke-standard port, kun fra LAN.
- Telnet: **alltid av**.
- Admin-passord: langt og unikt, lagret i passordmanager, ikke gjenbrukt.
- Logg ut av WebUI etter bruk (mot CSRF).
- Fjern­administrasjon / "access from WAN": **av**.

## 3. Tjenester på ruteren
- UPnP og NAT-PMP: **av**. Porter åpnes manuelt og dokumenteres (se §7).
- AiCloud, FTP, Samba/SMB, mediaserver: **av** med mindre i aktiv bruk.
- DDNS: kun hvis nødvendig; ellers av.

## 4. DNS
- DNS-over-TLS aktivert (f.eks. Quad9 9.9.9.9 — malware-blokkerende, eller Cloudflare).
- DNSSEC-validering: på. DNS-rebind-vern: på.
- All LAN-DNS tvinges gjennom ruteren (klienter kan ikke omgå).
- Blokkliste for annonser/malware aktiv (Diversion / bakt inn i QeamerWRT).

## 5. Brannmur
- Default-drop på all uoppfordret inngående WAN-trafikk.
- IPSet-blokklister (botnett/skannere) lastet ved boot.
- Drop INVALID/spoofede pakker; SYN- og ICMP-rate-limiting.
- Dropp logges til syslog; gjennomgå loggen ved mistanke.

## 6. Nettverkssegmentering
- IoT-enheter (kameraer, smartplugger, TV) på **isolert gjeste-/IoT-nett**, uten tilgang til hovednettet.
- Gjester på eget gjestenett med klient-isolasjon.

## 7. Endrings- og portlogg
Før en enkel logg (tekstfil) over:
- Hver port du åpner inn: port, enhet, grunn, dato. Lukk når ubrukt.
- Hver firmware-/backport-endring: dato, versjon, hva som ble endret.

## 8. Oppdatering & vedlikehold
- Sjekk Merlin/oppstrøms for nye CVE-er i dnsmasq, openssl, dropbear, curl **hver 1–3. mnd**.
- Backport sikkerhetsfikser til QeamerWRT når de kommer (du er nå "leverandøren").
- Ta NVRAM-/config-backup før hver endring.

## 9. Sikkerhetskopi & gjenoppretting
- Behold siste kjente fungerende firmware-image lagret utenfor ruteren.
- Ha USB-TTL-seriekabel + CFE-recovery-prosedyre dokumentert og testet **før** du trenger den.
- Eksporter ruter-config etter hver bekreftet fungerende endring.

## 10. Hendelseshåndtering (lettvekt, for ett hjem)
Ved mistanke om kompromittering:
1. Koble ruteren fra WAN.
2. Sjekk DNS-innstillinger, åpne porter, ukjente admin-sesjoner, syslog.
3. Flash siste kjente rene image på nytt; bytt admin-passord og SSH-nøkler.
4. Sjekk øvrige enheter for sidebevegelse.
5. Noter hva som skjedde i endringsloggen.

---

### Etterlevelse
Denne policyen er oppfylt når §2–§6 er konfigurert som beskrevet og §8-gjennomgang
er gjort innen fristen. Mål: ingen åpen dør inn, oppdatert userspace, isolert IoT,
testet recovery. Det er "god nok"-standarden for et lavt-trussel-hjem på eldre maskinvare.
