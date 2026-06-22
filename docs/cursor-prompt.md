# Åpningsprompt til Cursor

Kopier teksten under og lim inn som din **første** melding til Cursor i QeamerWRT-repoet.

---

Du jobber i QeamerWRT-repoet — en herdet, om-brandet fork av Asuswrt-Merlin
384.13_10 for ASUS RT-AC87U (BCM4709, ARMv7 32-bit).

Før du gjør noe som helst: les `AGENTS.md` og `README.md` i sin helhet, og følg
reglene i `AGENTS.md` strengt — de hindrer at en fysisk ruter blir brikket.
Bekreft kort at du har forstått de harde begrensningene (ARMv7 ikke ARMv8,
legacy-repo ikke .ng, aldri kopier binærer, ikke rør kjernen/WiFi-driveren,
ikke marker noe som "fungerer" uten bekreftelse fra meg på maskinvaren).

Deretter, ikke skriv kode ennå. Din første oppgave er **Fase 1** fra
`guides/phase1-build.md`: hjelpe meg å sette opp byggemiljøet og få en
**uendret** `make rt-ac87u` til å bygge og produsere et TRX-image. Vi rører
hverken branding eller herding før det uendrede imaget bygger og booter.

Start med å gå gjennom Fase 1 steg for steg med meg, og still meg spørsmål om
mitt oppsett (OS, VM eller container, ressurser) før du gir kommandoer. Hold
deg til git-/kvalitetsreglene i AGENTS.md: små commits, bygg og bug-sjekk før
hver push, `main` skal alltid kunne bygge.

---

## Tips for økten
- La Cursor jobbe i **én fase om gangen**. Ikke be om "bygg hele firmwaren" — det fører til hallusinasjon.
- Når en byggefeil dukker opp: lim `build-ac87u.log`-linjene rundt feilen inn i Cursor.
- Be Cursor lage en branch per oppgave (`feat/branding`, `backport/dnsmasq`) og merge til `main` først når det bygger.
- Du gjør all flashing/recovery selv. Cursor verifiserer kun at det *bygger*.
