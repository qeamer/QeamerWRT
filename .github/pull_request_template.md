## What & why
<!-- One logical change. What does it do, and which CVE/measure does it address? -->

## Type
- [ ] security (backport / hardening)
- [ ] feat (branding / UI / feature)
- [ ] fix
- [ ] build
- [ ] docs

## Pre-push checklist
- [ ] `make rt-ac87u` builds with no new errors
- [ ] `shellcheck` / `cppcheck` clean on changed files
- [ ] `git diff` reviewed — no debug leftovers, no hardcoded values
- [ ] No binaries from other firmware images added
- [ ] Changed component reports the intended version
- [ ] **No upstream copyright/license notices removed** (see NOTICE.md)
- [ ] No secrets committed (keys, passwords, NVRAM)
- [ ] `CHANGELOG.md` updated

## Backport source (if applicable)
- Upstream URL:
- Version (from → to):
- Checksum verified:

## On-device verification
- [ ] Not yet tested on hardware (maintainer flashes/tests)
- [ ] Confirmed working on device by maintainer
