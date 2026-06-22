# QeamerWRT — Fase 1: byggemiljø & første test-build

Mål: få et **uendret** AC87U-image til å kompilere og boote *før* vi rører
branding eller herding. Det beviser at toolchain-en din virker, så feil senere
skyldes endringene dine — ikke oppsettet.

> ⚠️ Bygg aldri på prod-maskinen. Bruk en VM eller container. Ikke flash noe
> før recovery-oppsettet i roadmapen (§5) er på plass.

---

## Steg 1 — Vertsmiljø
AC87U bygges fra **legacy-repoet** (`RMerl/asuswrt-merlin`), ikke `.ng`.
Det er gammel kode og krever et eldre bygge-OS.

- Anbefalt: **Ubuntu 18.04 LTS x86_64** (20.04 fungerer ofte; nyere kan kreve patching av gamle build-script).
- Sett opp som VM (VirtualBox/VMware/Hyper-V) eller LXC/Docker-container.
- Gi den ≥ 4 vCPU, 4–8 GB RAM og **≥ 40 GB disk** (kildetre + toolchain er stort).

## Steg 2 — Byggeavhengigheter
```bash
sudo dpkg --add-architecture i386
sudo apt update
sudo apt install -y build-essential libtool-bin autoconf automake \
  bison flex g++ gawk gengetopt gettext git gperf nano \
  libncurses5-dev libncurses5 libssl-dev zlib1g-dev unzip uuid-dev \
  xsltproc cmake pkg-config python3 python-is-python3 \
  lib32z1 lib32stdc++6 libc6-i386 mtd-utils u-boot-tools
```
*(Noen 32-bit i386-pakker trengs fordi deler av den gamle toolchain-en er 32-bit.)*

## Steg 3 — Hent kildekoden
```bash
mkdir -p ~/qeamerwrt && cd ~/qeamerwrt
git clone https://github.com/RMerl/asuswrt-merlin.git
cd asuswrt-merlin

# Bekreft at AC87U-target finnes (BCM4709 / ARM-platform)
ls release/src-rt-6.x.4708/
```
> Verifiser mot repoets egen `README.md` hvilken branch/tag som er siste
> 384.13-base, og bytt til den (`git checkout <tag>`). Repo-strukturen er fasit
> hvis noe her avviker.

## Steg 4 — Toolchain
Toolchain-en følger med i repoet (`tools/`) og pakkes ut ved første `make`.
Hvis den ligger som arkiv:
```bash
cd ~/qeamerwrt/asuswrt-merlin/tools
# følg repoets instruks for å pakke ut/sette PATH til kryss-kompilatoren
```

## Steg 5 — Første uendrede build
```bash
cd ~/qeamerwrt/asuswrt-merlin/release/src-rt-6.x.4708
make rt-ac87u 2>&1 | tee ~/qeamerwrt/build-ac87u.log
```
- Tar typisk **30–90 min** første gang. Følg `build-ac87u.log` ved feil.
- Vanlige snublesteiner: manglende i386-bibliotek, for ny gcc, manglende
  `mtd-utils`/`u-boot-tools`. Logg-linjen rett før "Error" peker på pakken.

## Steg 6 — Finn imaget
```bash
ls -lh ~/qeamerwrt/asuswrt-merlin/release/src-rt-6.x.4708/image/
# forventet: RT-AC87U_3.0.0.4_384.13_*.trx
```
Sammenlign filstørrelse/format med original `RT-AC87U_384.13_10.trx` du allerede har — de bør ligne (samme HDR0/TRX-format).

## Steg 7 — Verifiser FØR flash
1. `sha256sum image/RT-AC87U_*.trx` — noter checksum.
2. Bekreft TRX-header: `xxd -l 4 image/RT-AC87U_*.trx` → skal vise `HDR0`.
3. **Recovery klar:** USB-TTL seriekabel tilkoblet, ASUS Firmware Restoration / CFE TFTP testet, gammel firmware lagret.
4. Flash det uendrede imaget. Booter det og fungerer nett → toolchain-en er bevist. Da, og først da, går vi til Fase 2 (branding) og Fase 3 (herding).

---

## Sjekkliste for Fase 1
- [ ] VM/container med Ubuntu 18.04 satt opp
- [ ] Avhengigheter installert
- [ ] Kilde klonet, riktig 384.13-tag valgt
- [ ] `make rt-ac87u` fullfører uten feil
- [ ] Image produsert med korrekt HDR0/TRX-format
- [ ] Recovery-oppsett (seriekabel + CFE/TFTP) testet
- [ ] Uendret image flashet og booter

Når alle er huket av: si fra, så går vi videre til å bake inn QeamerWRT-brandingen
og deretter herdingen, steg for steg.

> Merknad: kommandoene følger den dokumenterte Merlin-byggeflyten, men det
> nøyaktige avhengighets-settet og branch-navnet kan variere med repoets
> nåværende tilstand. Repoets egen `README` / wiki er fasiten — sjekk den hvis
> et steg avviker, så justerer vi.
