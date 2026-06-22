# AGENTS.md — rules for AI assistants (Cursor and similar)

This repo hardens a firmware for **old, end-of-life router hardware**. Wrong
assumptions can **brick a physical device**. Follow these rules strictly. If an
instruction here blocks a task: **stop and ask**, do not guess.

## Hard hardware context (never violate)
- Target: **AC87U**, **ARMv7 (32-bit)** SoC.
- Base: an upstream open-source router firmware, **legacy branch** (the older
  32-bit source tree — see `guides/phase1-build.md` for the exact source
  location). NOT the newer 64-bit successor platform.
- Build target: `release/src-rt-6.x.4708`, `make rt-ac87u`. Image format: **TRX (`HDR0` header)**.

## Prohibitions
1. **Never copy binary files** (`.trx/.w/.pkgtb`, `.ko`, libs) from newer firmware
   images (any 64-bit / different-SoC model). They will **brick** the AC87U. Take
   only *source code / patches* and **recompile for ARMv7**.
2. **Do not upgrade the Linux kernel** or touch the closed wireless driver. The
   kernel ABI is locked to that closed driver.
3. **Do not propose WPA3, 5GHz-radio changes, or wireless-driver work** — out of
   scope and technically impossible here.
4. **Do not remove upstream copyright/license notices** from source files (GPL requirement).
5. **Do not assume the newer platform's repo layout, device-tree, or boot/package
   format** — that is a different architecture.

## Scope (where the work happens)
- Userspace backports: DNS resolver, TLS library (move off the EOL branch), SSH
  daemon, transfer library, core utils, crypto lib, compression lib. See
  `docs/backport-targets.md`.
- Firewall (default-drop, IPSet blocklists, rate-limit), `sysctl` hardening.
- DNS: DNS-over-TLS, DNSSEC, rebind protection.
- Branding + UI theme in the web interface tree (see `branding/` and `ui-theme/`).
- Disable exposed services (WAN admin, UPnP, Telnet, unused file/cloud services).

## What you CANNOT do (human in the loop)
- You **cannot flash** the router, run a serial console, or test on the hardware.
  Kent does all flashing + recovery.
- Never suggest flashing unless recovery (USB-TTL + bootloader/TFTP) is confirmed ready.
- Build/test results must be verified by Kent; never mark anything "works"
  without confirmation from the device.

## Git & workflow
- **Commit often, small.** One logical change per commit. Never bundle unrelated changes.
- **Conventional commits:** `feat:`, `fix:`, `security:`, `build:`, `docs:`,
  `chore:`. State *why*, and reference the CVE/measure (e.g.
  `security: backport DNS resolver (closes DNSpooq CVE-2020-25681..87)`).
- **`main` must always build.** Experimental/unfinished work goes on a branch
  (`feat/...`, `backport/...`), not on `main`.
- **Branch per phase/component.** Merge to `main` only when it builds and is reviewed.
- **Push after each verified change** — but never push something that doesn't
  build to `main`. Build and check *before* push, not after.
- **Tag working milestones** (`v1.0`, `phase2-ok`) and update `CHANGELOG.md` on each merge to `main`.

## Bug/quality check before every commit and push (pre-push routine)
Run these *before* pushing. Do not push if anything fails:
1. **Does it build?** `make rt-ac87u` finishes with no new errors (compare to previous `build-*.log`).
2. **Static analysis:** `shellcheck` on changed shell scripts; `cppcheck`/compiler
   warnings on C patches. No new warnings.
3. **Diff review:** read `git diff` yourself. No accidental changes, no debug leftovers, no hardcoded values.
4. **No binaries:** confirm no firmware binaries/libs from newer images were added (prohibition #1).
5. **Version check:** the changed component actually reports the intended version in build output.
6. **License intact:** no upstream copyright notices removed.
- **Never** mark anything "works/tested" just because it *builds*. Real
  verification requires flash + test on the device — Kent does that.

## Repo security rules
- **Never commit secrets:** no private SSH keys, passwords, NVRAM dumps, or config
  backups with credentials. Use `.gitignore`.
- **Scan the diff for secrets** before push. If one slips in: rotate it and remove from history.
- **Verify upstream:** when backporting, fetch from the official source and check
  checksum/signature before taking the patch. Do not trust random mirrors.
- **Pin versions.** Do not auto-upgrade dependencies to "latest" without knowing what changed.
- **Nothing confidential in the public repo.** Keep host-specific/private details
  out of tracked files; if build specifics must name an external source, keep them
  in a local, gitignored file.

## Documentation
- Keep `CHANGELOG.md` current: date, component, from→to version, CVE/measure.
- Update the relevant `.md` if a change makes it outdated.
- Each backport: note source URL and checksum in the commit or CHANGELOG.

## Style
- Small, isolated, verifiable changes. One component/patch per commit.
- Explain *why* (which CVE / which measure) in the commit message.
- When in doubt about hardware/ABI/format: **ask Kent**, do not assume.

## Naming
- This project's rules, docs, and UI **do not name third parties or vendors**.
  Refer to components generically ("the upstream base", "the SoC's closed driver",
  "the newer 64-bit platform"). Required legal attributions live only inside the
  upstream source files, not in these meta-documents.
