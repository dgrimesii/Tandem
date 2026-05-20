# Tandem — Five Pillars

Tandem is organized around five core pillars. Each pillar owns a coherent slice of the domain — its own data, logic, and UI surface. Every feature, issue, and design decision belongs to exactly one pillar.

---

## The Five Pillars

```
┌─────────────────┐  ┌─────────────────┐  ┌─────────────────┐
│   IDENTITY      │  │    LIBRARY      │  │     RECORD      │
│                 │  │                 │  │                 │
│ Who you are     │  │ What you can do │  │ What you did    │
└────────┬────────┘  └────────┬────────┘  └────────┬────────┘
         │                    │                     │
         └────────────────────┼─────────────────────┘
                              │
              ┌───────────────┴───────────────┐
              │                               │
   ┌──────────┴──────────┐       ┌────────────┴────────┐
   │       PLANS         │       │      MOMENTUM        │
   │                     │       │                      │
   │ Where you're going  │       │ What keeps you going │
   └─────────────────────┘       └──────────────────────┘
```

---

## Data Flow

```
LIBRARY ──────────────────────────────────────────┐
  provides vocabulary to all pillars               │
                                                   ▼
IDENTITY ──► PLANS ──► RECORD ──► MOMENTUM
  context     intent    activity    motivation
                                       │
                                       └──► PLANS
                                       (multiplier evolution,
                                        plan activation signals)
```

---

## Pillar Definitions

---

### 1. Identity
> *Who you are in Tandem*

The context layer. Every other pillar personalizes itself using Identity data. Without Identity, the app is generic. With it, everything — suggestions, quest difficulty, effort point calculation, activity suitability, language — is calibrated to the real person.

**Owns:**
- User profiles (display name, fitness tier, personal multiplier)
- Physical conditions (`UserCondition` — body system, concern level, active flag)
- Heart rate profile (`UserHeartRateProfile` — resting HR, max HR, zones)
- Goals and motivations (muscle retention, longevity, weight loss, consistency)
- Fitness tier and personal multiplier (initial assignment + evolution over time)
- Multiplier history (audit trail of tier changes)
- Partner relationship (who your Tandem partner is)
- Shared settings and opt-in visibility preferences
- Onboarding flow

**Does not own:**
- Plan targets (Plans)
- Activity history (Record)
- Quest/streak state (Momentum)
- Body system impact definitions (Library)

**Key design constraints:**
- Personal multipliers are never shown to either user — invisible infrastructure
- Physical conditions are private — never shared with partner
- Each user's conditions are independent — A's knee concern does not appear in D's profile
- Conditions evolve: `active = false` when resolved, new conditions added as needed
- Heart rate profile defaults to 220-age for max HR; user can override with tested value
- Fitness tier framing: "where are you starting from" — never a judgment

---

### 2. Library
> *The vocabulary of movement*

The master catalog of activities. The shared language the entire app speaks. Every logged activity is an instance of a Library entry. Every plan targets Library entries. Every quest matches Library tags. The Library is objective — it describes what activities *are*, not what they mean to any particular user.

**Owns:**
- ActivityType definitions (the catalog)
- Category and hierarchy structure (how activities are organized)
- Broad tag taxonomy (`cardio`, `strength`, `flexibility`, `mindfulness`, `low-impact`, `balance`)
- Strength subtype tags (`strength:functional`, `strength:core`, `strength:resistance`, `strength:mobility`)
- Activity flags — `indoor`, `outdoor`, `equipment-free` (stored); `back-safe`, `knee-safe`, `pf-safe` (computed from body system impacts)
- Body system impact data (`ActivityBodySystemImpact` — objective stress per body system per activity)
- Body system vocabulary (closed set: `lumbar_spine`, `hip`, `knee`, `ankle_foot`, `shoulder`, `wrist_hand`, `neck_cervical`, `cardiovascular`, `general_joint`)
- `intensity_default` per activity (low/medium/high — used as estimation input only)
- `min_qualifying_minutes` per activity
- Equipment requirements per activity type
- Seed library (~50 curated activity types at launch)
- Custom activity creation

**Does not own:**
- Logged activity instances (Record)
- How tags map to quest eligibility (Momentum)
- Plan-level activity targets (Plans)
- User-specific condition interpretation (Identity)

**Key design constraints:**
- Body system impacts are objective facts about activities — independent of any user
- `back_stress_level` as a single field is replaced by `ActivityBodySystemImpact[]` — multiple systems, per activity
- Safety flags (`back-safe`, `knee-safe`, `pf-safe`) are computed at query time by joining `ActivityBodySystemImpact` with `UserCondition` — never manually curated
- `intensity_default` is an estimation fallback — it does not directly multiply into effort points
- Tags and flags live on ActivityType — never manually assigned at log time
- The Library is read-only to all other pillars

**ActivityType data shape:**
```
ActivityType
├── id
├── name                      "Resistance Band Circuit"
├── display_name              "Resistance Bands"
├── category                  "Strength"
├── broad_tags[]              ["strength", "low-impact"]
├── subtype_tag               "strength:functional"
├── flags[]                   ["indoor", "equipment-free"]  ← stored flags only
├── intensity_default         "medium"                      ← estimation input, not multiplier
├── min_qualifying_minutes    15
├── equipment_required[]      ["resistance bands"]
├── suggested_for[]           ["beginner", "intermediate"]
└── is_custom                 false

ActivityBodySystemImpact (related — one per system)
├── activity_type_id
├── body_system               "knee" | "lumbar_spine" | "hip" | etc.
├── stress_level              "low" | "medium" | "high"
└── notes                     optional
```

---

### 3. Record
> *What you did*

The primary daily interaction surface. Everything either user does in the app on a typical day lives here. Fast, personal, friction-free on the surface — rich and structured underneath. Feeds every other pillar.

**Owns:**
- Activity logging (manual entry)
- Quick log flow (one-tap, minimum required fields)
- "Just Start" flow (lowest-barrier entry point, inertia breaking)
- Body comfort signal capture (Good / Neutral / Discomfort — optional, any body system)
- Pain day logging and modified activity suggestions
- Activity history and personal timeline
- Biometric capture (`ActivityBiometric` — HR, power, cadence, distance, steps, etc.)
- Effort level resolution (three-priority: user override → biometric → estimated)
- Effort point calculation and storage (executes here at log time)
- Future: automated ingestion from Strava, Health Connect, Peloton (Phase 2+)

**Does not own:**
- ActivityType definitions (Library)
- Effort multiplier values or personal multipliers (Library + Identity supply inputs)
- Quest progress updates triggered by logging (Momentum reads Record)
- Streak updates triggered by logging (Momentum reads Record)

**Key design constraints:**
- Friction in logging is the enemy — required fields are activity type and duration only
- All other fields (intensity, body comfort, notes, biometrics) are always optional
- Effort points are calculated once at log time and stored — never recalculated on read
- `effort_source` tracks provenance: `user_override` / `biometric` / `estimated`
- `body_comfort` replaces `back_comfort` — applicable to any body system for either user
- Biometrics stored as flexible key-value via `ActivityBiometric` — new types require no schema change
- Pain day flag is trusted — never questioned, never challenged
- Comfort log is private — never visible to partner, exportable for medical use
- Confirmation language is always warm and specific: "You rode today. That counts."

**Activity data shape:**
```
Activity (logged instance)
├── id
├── user_id
├── activity_type_id          → ActivityType (Library)
├── plan_id                   → Plan (optional)
├── duration_minutes
├── effort_multiplier         resolved value (stored)
├── effort_source             user_override / biometric / estimated
├── effort_points             duration × personal_multiplier × effort_multiplier
├── intensity_override        Easy / Moderate / Hard / Maximum (optional)
├── body_comfort              Good / Neutral / Discomfort (optional)
├── body_system_flagged       nullable — which system the comfort signal refers to
├── pain_day                  boolean
├── notes                     free text (optional)
├── source                    manual / strava / peloton / health_connect
├── logged_at
└── created_at

ActivityBiometric (related — one per biometric type captured)
├── activity_id
├── biometric_type            heart_rate_avg | heart_rate_max | steps |
│                             cadence_avg | power_avg | calories | distance |
│                             elevation_gain | pace_avg | spo2_avg | hrv | etc.
├── value                     numeric
├── unit                      "bpm" | "steps" | "kcal" | "meters" | "watts" | etc.
├── aggregation               avg / max / min / total
└── source                    manual / heart_rate_monitor / strava / health_connect / peloton
```

**Effort resolution reference:** See `docs/design/effort-calculation.md`

---

### 4. Plans
> *Where you're going*

The intentional, forward-looking layer. Goals structured into trackable commitments. Plans give Record entries meaning and direction — without Plans, you're logging into a void. Plans define the intention; Record captures the execution; Momentum tracks the gap.

**Owns:**
- Plan creation and management
- Activity targets (frequency per week, minimum duration)
- Typical day patterns (`typical_days[]` — for Awareness Engine projection accuracy)
- Plan status lifecycle (active / on_hold / retired)
- Progression logic (plans that evolve targets over time)
- Plan events (activation, retirement, progression — consumed by Momentum milestones)
- Weather contingency mapping (outdoor activity → indoor fallback)
- Exploration track management (open-ended discovery phase)
- Plan activation milestones (the "chapter opening" moment when on_hold → active)
- Plan-to-quest contribution rules (which plans feed which quest tags)

**Does not own:**
- Progress tracking against plan targets (Momentum reads Record against Plan definitions)
- Activity type definitions (Library)
- Quest definitions (Momentum)

**Key design constraints:**
- Plans are private — partner sees shared quest progress, not individual plan details
- On-hold plans remain visible but generate no quest contributions or suggestions
- `typical_days[]` enables the Awareness Engine to project realistically — not just mathematically
- Plan activation (on_hold → active) should feel like a meaningful moment, not a settings toggle
- Every active plan should have a defined indoor fallback for weather-dependent activities

---

### 5. Momentum
> *What keeps you going*

The motivational engine. Everything that creates forward motion, celebrates progress, and sustains engagement over time. Momentum is the consumer of all other pillars — it reads Identity for context, Library for vocabulary, Record for data, and Plans for intent.

A Momentum object is defined by the intersection of three things:
- **What it tracks** — Library (tags, activity types, classifications)
- **Who is doing it** — Identity (user, multiplier, conditions, partnership)
- **What actually happened** — Record (logged activities, effort points, biometrics)

Plans add a fourth dimension — **what was intended** — enabling projections beyond simple pace tracking.

**Owns:**
- Quest engine (definitions, types, progress accumulation, completion, miss handling)
- Quest Awareness Engine (pace calculation, risk states, projections informed by Plans)
- Activity Streak (daily, with Streak Shield mechanic and recovery window)
- Journey Streak (weekly, hard to break, long-arc confidence builder)
- Shared quest progress view (combined progress, both contributions visible)
- Notifications and nudges (warm, forward-looking, never punitive)
- Weekly wins summary (Sunday, wins only, no gaps surfaced)
- Celebration moments (completions, comebacks, personal bests, milestones)
- Longevity milestones (long-arc, non-time-bound shared achievements)
- Consistency scoring (rolling 4-week activity rate)
- Partner awareness (opt-in visibility into partner's activity and quest risk)

**Does not own:**
- Activity definitions (Library)
- Body system impacts or suitability calculation (Library + Identity)
- Plan targets (Plans — Momentum reads them but doesn't define them)
- The actual activity records (Record — Momentum reads them)
- User multipliers or HR profiles (Identity — Momentum uses them)

**Key design constraints:**
- Streaks have real stakes — they can break. Removing all risk removes the motivation.
- One bad day never ends the story — Shields, recovery windows, Journey Streak ensure continuity
- The app never surfaces failure — only forward-looking messages, always
- Quest risk state UI emphasizes path forward, never the gap
- Wins-only weekly summary — zero mention of missed targets or gaps
- Awareness Engine projections use `Plans.typical_days[]` for realistic (not just mathematical) pace
- All Momentum copy reviewed against the "never say" list in vision.md
- Partner awareness is opt-in, default off — never assumed

---

## Pillar Interaction Rules

1. **Library is read-only to all other pillars** — no pillar redefines or overrides Library classifications
2. **Record is the single source of activity truth** — Momentum and Plans read from Record, never duplicate it
3. **Identity context flows down, never up** — pillars use Identity data but don't write back to it (except multiplier evolution, which is a deliberate, user-initiated action)
4. **Plans define intent; Momentum measures it** — a Plan says "3x/week"; Momentum reads Record entries and evaluates whether that intent was met
5. **Momentum closes the loop** — it's the only pillar that influences future behavior by feeding signals back toward Record (via nudges, suggestions, "Just Start")
6. **Body system suitability is a runtime join** — Library provides objective impact facts; Identity provides active conditions; the join happens at query time, never stored

---

## Future Pillar: Insights

As data accumulates, a sixth pillar may emerge:

**Insights** — the read-only synthesis layer. Quarterly functional fitness check-ins, pain day trend analysis (by body system), HR trend analysis (fitness improvement marker), weight loss correlation with activity type, longevity marker tracking. Currently small enough to live within Momentum, but worth naming now so it has a home when the time comes.

---

## Issue Labels

Every GitHub issue is tagged with its pillar:

| Label | Color |
|---|---|
| `pillar:identity` | #7B68EE (medium slate blue) |
| `pillar:library` | #20B2AA (light sea green) |
| `pillar:record` | #FF8C00 (dark orange) |
| `pillar:plans` | #9370DB (medium purple) |
| `pillar:momentum` | #FF6347 (tomato) |
| `pillar:infrastructure` | #708090 (slate grey) |
