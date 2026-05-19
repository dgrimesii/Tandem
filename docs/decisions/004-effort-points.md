# ADR 004 — Effort Points as Quest Currency

**Status:** Accepted

## Context
Two users with significantly different fitness baselines. Raw minutes as quest currency would make shared quests inequitable — the higher-baseline user would always contribute more without trying harder.

## Decision
Convert activity duration to effort points using two multipliers:
- **Personal multiplier** — reflects baseline fitness tier (e.g. 0.75 for experienced athlete, 1.25 for beginner)
- **Activity multiplier** — reflects relative intensity of the activity type

Quests are denominated in effort points, not minutes. The math is invisible to users.

## Consequences
- Shared quests feel genuinely equitable
- Neither user sees the other's multiplier — only their effort point contributions
- Multipliers should evolve as fitness baselines change
- Initial multiplier values need calibration — treat as configurable constants, not hardcoded values
