# QeamerWRT — branding-integrasjon

Eier / build owner: **Kent "Qeamer" Nygjerdet**

Web-grensesnittet til Asuswrt-Merlin ligger i kildetreet under
`release/src/router/www/`. All branding nedenfor legges inn der før du bygger.

## 1. Logo-filer

Kopier disse inn i `release/src/router/www/`:

| Fil | Bruk |
|-----|------|
| `qeamerwrt_logo_dark.png` | Banner-logo (mørk bakgrunn i UI-et) |
| `qeamerwrt_logo.png` | Lys variant / dokumentasjon |
| `qeamer_branding.css` | Override-stilark |

## 2. Koble inn stilarket

I `release/src/router/www/Main_Login.asp` og i hovedmalen
`require/require.com.js` (eller `index.asp` head-seksjon), legg til
**etter** det eksisterende `<link ... NM_style.css>`:

```html
<link rel="stylesheet" type="text/css" href="qeamer_branding.css">
```

## 3. Eier-linje i banneret

I banner-malen (`as.asp` / topbanner-blokken i `index.asp`), rett etter
logo-div-en, legg til:

```html
<div id="qeamer-owner">QeamerWRT · build by Kent &quot;Qeamer&quot; Nygjerdet</div>
```

## 4. Login-side undertekst

I `Main_Login.asp`, under påloggingsboksen:

```html
<div style="color:#2DD4BF;font-size:13px;text-align:center;margin-top:10px;">
  QeamerWRT — Secure Router Firmware<br>
  <span style="color:#64748B;">Maintained by Kent &quot;Qeamer&quot; Nygjerdet</span>
</div>
```

## 5. Modell-/versjonsstreng

Sett en gjenkjennelig build-streng så den vises i *Administration → System*
og i SSH-banneret. I `release/src/router/shared/version` (eller
`SDK/.../build_no`), bruk f.eks.:

```
QeamerWRT 1.0  (AC87U · hardened Merlin 384.13_10 base)
```

og i `release/src/router/shared/etc/motd`:

```
  QeamerWRT  —  Secure Router Firmware
  Owner: Kent "Qeamer" Nygjerdet
  Base: Asuswrt-Merlin 384.13_10 (RT-AC87U)
```

## Merknad
Endring av navn/logo er tillatt under GPL så lenge kildekoden til
endringene gjøres tilgjengelig. Ikke fjern ASUS/Merlin sine GPL-merknader
i `LICENSE`/`README` — legg din branding *på toppen*, ikke i stedet for.
