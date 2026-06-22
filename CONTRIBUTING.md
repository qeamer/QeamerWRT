# Contributing to QeamerWRT

Thanks for helping harden QeamerWRT. This project targets **old, end-of-life
router hardware**, so a few rules are strict — they protect a physical device and
respect the licenses of the code we build on.

Read `AGENTS.md` first; it holds the hard hardware constraints.

## Golden rules
1. **Preserve all upstream copyright and license notices.** Never remove or alter
   the original authors' notices (ASUS, Asuswrt-Merlin / Eric Sauvageau, Linux,
   GNU, and component authors). See `NOTICE.md`. QeamerWRT branding is added
   *alongside* them, never instead of them.
2. **Never copy binaries** from newer / different-architecture firmware images.
   Take source/patches only and recompile for the ARMv7 target.
3. **Do not touch the kernel or the closed wireless driver.**
4. **Never commit secrets** — keys, passwords, NVRAM dumps. See `.gitignore`.
5. **Don't mark anything "works" without on-device confirmation** (the maintainer flashes/tests).

## Workflow
- Branch per change: `feat/...`, `backport/...`, `fix/...`. Keep `main` buildable.
- One logical change per commit. Conventional commits (`feat:`, `fix:`,
  `security:`, `build:`, `docs:`, `chore:`), explaining *why* and citing the
  CVE/measure where relevant.
- Open a PR using the template; fill in the pre-push checklist.

## Pre-push checklist (also in the PR template)
- [ ] `make rt-ac87u` builds with no new errors
- [ ] `shellcheck` / `cppcheck` clean on changed files
- [ ] `git diff` reviewed — no debug leftovers, no hardcoded values
- [ ] No binaries from other firmware added
- [ ] Changed component reports the intended version
- [ ] No upstream copyright/license notices removed
- [ ] No secrets committed
- [ ] `CHANGELOG.md` updated

## Backports
- One component per PR. Fetch from the official upstream source; verify
  checksum/signature. Record source URL + checksum in the commit or `CHANGELOG.md`.
