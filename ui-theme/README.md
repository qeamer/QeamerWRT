# QeamerWRT — UI theme

A web-interface theme that matches the QeamerWRT logo: dark slate base,
blue→teal gradient accents, clean geometric sans, the Q-ring mark.

## Files
| File | Purpose |
|------|---------|
| `theme.css` | The theme. Drop into the web UI tree; load after the stock stylesheet. |
| `preview.html` | Open in any browser to see the live theme. |
| `preview.svg` / `preview.png` | Static mockup of the themed dashboard. |

## Palette (from the logo)
- Base `#0F172A` · panels `#1E293B` / `#273449` · lines `#334155`
- Text `#F1F5F9` · muted `#94A3B8` · dim `#64748B`
- Primary `#3B82F6` → hover `#60A5FA` · accent `#14B8A6` → `#2DD4BF`
- Gradient `135deg, #3B82F6 → #14B8A6`

## Integration (build phase 2)
1. Copy `theme.css` and the logo PNGs from `assets/logo/` into the web interface directory
   (`release/src/router/www/`).
2. In the page head/template, add **after** the stock stylesheet link:
   ```html
   <link rel="stylesheet" type="text/css" href="theme.css">
   ```
3. The selectors target the stock UI's class names; adjust any that differ in
   your source tree. Keep all colors referencing the `--qw-*` CSS variables so
   the palette stays consistent with the logo.
4. Verify in `preview.html` first, then bake into the build and confirm on the
   device (Kent).

> Note: class names in the stock interface can vary between source revisions.
> Treat `preview.html` as the source of truth for the intended look; map the
> theme onto whatever selectors the actual UI uses.
