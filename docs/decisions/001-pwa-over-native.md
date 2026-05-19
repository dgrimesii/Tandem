# ADR 001 — PWA over Native Android for Phase 1

**Status:** Accepted

## Context
Both users are on Android (Pixel 8 Pro, Fold 10). Native Android gives full Health Connect access and richer notifications but significantly increases build complexity and time-to-prototype.

## Decision
Build Phase 1 as a React PWA hosted on the Raspberry Pi. Validate the core quest/streak experience before committing to native.

## Consequences
- Faster path to a working prototype
- Health Connect deferred to Phase 4
- Strava integration unaffected (OAuth works from PWA)
- Native wrapper remains an option once core UX is validated
