# Tandem — Effort Calculation

> This document is the single source of truth for how effort points are derived.
> All pillars that read or write effort data should reference this document.

---

## Purpose

Effort points are the currency of Tandem's quest system. They replace raw minutes
as the unit of progress so that a challenging session for one person is weighted
appropriately relative to the same session for a fitter person, and so that a hard
effort counts more than a casual one.

Effort points must always resolve to a value. The system degrades gracefully when
the best data is unavailable.

---

## The Formula

```
effort_points = duration_minutes × personal_multiplier × effort_multiplier
```

### Personal Multiplier
Owned by Identity. Set from fitness tier during onboarding. Invisible to users.
Evolves over time via milestone-triggered prompts (see #35).

| Fitness Tier | Personal Multiplier |
|---|---|
| Beginner | 1.25x |
| Intermediate | 1.05x |
| Active | 0.90x |
| Athlete | 0.75x |

### Effort Multiplier
Resolved via the three-priority hierarchy below. Stored on each Activity record
alongside `effort_source` (provenance) and `intensity_override` (if user-set).

---

## Three-Priority Resolution Hierarchy

The system evaluates the following in order and uses the first available signal.

---

### Priority 1 — User Override

**When available:** User explicitly selects intensity during or after logging.

**How it works:** The user selects Easy / Moderate / Hard / Maximum. This overrides
any biometric or estimated value. The user knows how an activity felt — that judgment
is trusted.

| User Selection | Effort Multiplier |
|---|---|
| Easy | 0.80x |
| Moderate | 1.00x |
| Hard | 1.20x |
| Maximum | 1.50x |

**Stored as:** `intensity_override` on Activity, `effort_source = "user_override"`

---

### Priority 2 — Biometric (Heart Rate Based)

**When available:**
- `heart_rate_avg` biometric exists on the Activity
- `UserHeartRateProfile` is populated for the user (resting HR + max HR)

**How it works:** Uses Heart Rate Reserve (HRR) method — the standard for
calculating relative exercise intensity.

```
HRR% = (HR_avg - HR_resting) / (HR_max - HR_resting) × 100
```

HRR% maps to zones and multipliers:

| HRR% Range | Zone | Effort Multiplier |
|---|---|---|
| < 50% | Recovery | 0.70x |
| 50–60% | Base | 0.85x |
| 60–70% | Aerobic | 1.00x |
| 70–80% | Threshold | 1.20x |
| 80–90% | VO2 | 1.40x |
| > 90% | Maximum | 1.60x |

**Heart rate profile defaults:**
- `resting_heart_rate`: measured (prompted during onboarding, updatable)
- `max_heart_rate`: 220 - age (calculated default, overridable with tested value)

**Stored as:** effort_multiplier from zone lookup, `effort_source = "biometric"`

**Note:** Over time, as resting heart rate drops with improved fitness, the same
session at the same absolute HR will produce a higher HRR% — correctly reflecting
that it represents a greater relative effort for a deconditioned user. This is
intentional and accurate.

---

### Priority 3 — Estimated

**When available:** Always. This is the fallback when no override and no HR data.

**How it works:** Combines three inputs to produce a directionally correct estimate.

```
effort_multiplier = base_intensity × duration_ratio × tier_factor
```

**base_intensity** — from `activity_type.intensity_default`:
| Intensity Default | Base Value |
|---|---|
| low | 0.75 |
| medium | 1.00 |
| high | 1.20 |

**duration_ratio** — longer sessions are harder:
```
duration_ratio = min(duration_minutes / 45, 1.4)
```
Sessions under 45 min scale proportionally. Sessions above 45 min cap at 1.4x
(diminishing returns — a 90-min session isn't twice as hard as a 45-min one).

**tier_factor** — fitter users find the same activity relatively easier:
| Fitness Tier | Tier Factor |
|---|---|
| Beginner | 1.00 |
| Intermediate | 0.95 |
| Active | 0.90 |
| Athlete | 0.85 |

**Stored as:** calculated value, `effort_source = "estimated"`

---

## Effort Source Transparency

Every Activity stores `effort_source` — the provenance of the effort_multiplier used.
This serves two purposes:

1. **Trust** — the app can optionally show users how their points were calculated
   ("based on your heart rate" vs "estimated from activity type")

2. **Calibration** — when both HR data and estimation are available, the divergence
   between them can be used to improve the estimation model over time

The `effort_source` field is never surfaced negatively. Estimated is not worse —
it's just less precise. The framing, if shown: "estimated from activity" (not
"no HR data available").

---

## Biometric Data Sources

Biometrics attach to Activity via `ActivityBiometric` records. Multiple biometrics
can be stored per session. Heart rate is the most universally useful for effort
calculation; others are captured for completeness and future use.

| Source | What it provides |
|---|---|
| Manual entry | User types average HR after session |
| Heart rate monitor (Bluetooth) | HR avg, HR max, HR zones |
| Strava sync | HR avg, HR max, distance, elevation, pace |
| Health Connect | HR avg, steps, calories |
| Peloton sync | Power avg, cadence avg, calories, HR avg |

For Phase 1 (manual logging), manual HR entry is the primary path. A single
optional field at log time ("What was your average heart rate?") unlocks Priority 2
immediately, before any integrations exist.

---

## Activity Suitability (Body System Awareness)

Effort calculation and activity suitability are separate concerns but both inform
the activity suggestion engine.

Suitability is computed at query time from:
- `ActivityBodySystemImpact` (Library — objective stress per body system)
- `UserCondition` (Identity — active conditions per user)

Result: `suitable / flag / caution / avoid`

This does NOT affect effort points. A "caution" activity has the same effort
calculation as a "suitable" one. Suitability only affects:
- Whether the activity is suggested (Just Start, pain day alternatives)
- Whether a soft warning is shown at log time

See `docs/design/data-model.md` for the full ActivitySuitability calculation.

---

## Cross-Pillar Dependencies

| Pillar | What it provides | What it needs |
|---|---|---|
| **Identity** | personal_multiplier, UserHeartRateProfile | Nothing — source data |
| **Library** | intensity_default (estimation input) | Nothing — source data |
| **Record** | Stores effort_multiplier, effort_source, effort_points, ActivityBiometric | Reads Identity (personal_multiplier, HR profile), Library (intensity_default) |
| **Plans** | frequency_target, typical_days (for projection) | Reads Record (actual effort points logged) |
| **Momentum** | Reads effort_points for quest accumulation and projections | Reads Record (effort_points), Plans (frequency_target for projections) |
