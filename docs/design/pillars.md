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

The context layer. Every other pillar personalizes itself using Identity data. Without Identity, the app is generic. With it, everything — suggestions, quest difficulty, effort point calculation, language — is calibrated to the real person.

**Owns:**
- User profiles (display name, fitness tier, personal multiplier)
- Physical context (back/hip status, knee pain, plantar fasciitis, medical flags)
- Goals and motivations (muscle retention, longevity, weight loss, consistency)
- Fitness tier and personal multiplier (initial assignment + evolution over time)
- Partner relationship (who your Tandem partner is)
- Shared settings and opt-in visibility preferences
- Onboarding flow

**Does not own:**
- Plan targets (Plans)
- Activity history (Record)
- Quest/streak state (Momentum)

**Key design constraints:**
- Personal multipliers are never shown to either user — invisible infrastructure
- Physical context (back issues, pain flags) is private — never shared with partner
- Fitness tier framing: "where are you starting from" — never a judgment

---

### 2. Library
> *The vocabulary of movement*

The master catalog of activities. The shared language the entire app speaks. Every logged activity is an instance of a Library entry. Every plan targets Library entries. Every quest matches Library tags. The Library is the foundation everything else is written on.

**Owns:**
- ActivityType definitions (the catalog)
- Category and hierarchy structure (how activities are organized)
- Broad tag taxonomy (`cardio`, `strength`, `flexibility`, `mindfulness`, `low-impact`, `balance`)
- Strength subtype tags (`strength:functional`, `strength:core`, `strength:resistance`, `strength:mobility`)
- Activity flags (`back-safe`, `knee-safe`, `pf-safe`, `indoor`, `outdoor`, `equipment-free`)
- Classification rules (what tags and flags each ActivityType carries)
- Default effort multipliers per activity type
- Back stress levels per activity type (`low`, `medium`, `high`)
- Equipment requirements per activity type
- Seed library (~50 curated activity types at launch)
- Custom activity creation (user-defined activities beyond the seed library)

**Does not own:**
- Logged activity instances (Record)
- How tags map to quest eligibility (Momentum)
- Plan-level activity targets (Plans)

**Key design constraints:**
- Tags and flags live on ActivityType — never manually assigned at log time
- The seed library is treated as curated content, not just test data
- Custom activities inherit category defaults until calibrated
- The Library is the source of truth for classification — other pillars read from it, never redefine it

**Data shape:**
```
ActivityType
├── id
├── name                      "Resistance Band Circuit"
├── display_name              "Resistance Bands"
├── category                  "Strength"
├── broad_tags[]              ["strength", "low-impact"]
├── subtype_tag               "strength:functional"
├── flags[]                   ["back-safe", "knee-safe", "equipment-free"]
├── default_multiplier        1.0
├── min_qualifying_minutes    15
├── back_stress_level         "low"
├── intensity_default         "medium"
├── equipment_required[]      ["resistance bands"]
├── suggested_for[]           ["beginner", "intermediate"]
└── is_custom                 false
```

---

### 3. Record
> *What you did*

The primary daily interaction surface. Everything either user does in the app on a typical day lives here. Fast, personal, friction-free on the surface — rich and structured underneath. Feeds every other pillar.

**Owns:**
- Activity logging (manual entry)
- Quick log flow (one-tap, minimum required fields)
- "Just Start" flow (lowest-barrier entry point, inertia breaking)
- Back/hip comfort signal capture (Good / Neutral / Discomfort — optional)
- Pain day logging and modified activity suggestions
- Activity history and personal timeline
- Effort point calculation (executes here; formula and multipliers defined in Identity and Library)
- Future: automated ingestion from Strava, Health Connect, Peloton (Phase 2+)

**Does not own:**
- ActivityType definitions (Library)
- Effort point formula or multiplier values (Identity + Library)
- Quest progress updates triggered by logging (Momentum reads Record)
- Streak updates triggered by logging (Momentum reads Record)

**Key design constraints:**
- Friction in logging is the enemy — required fields are activity type and duration only
- All other fields (intensity, back comfort, notes) are always optional
- The "Just Start" flow shows one suggestion, starts a timer, never shows a target or goal
- Pain day flag is trusted — never questioned, never challenged
- Back comfort log is private — never visible to partner, exportable for medical use
- Confirmation language is always warm and specific: "You rode today. That counts."

**Data shape:**
```
Activity (logged instance)
├── id
├── user_id
├── activity_type_id          → ActivityType (Library)
├── plan_id                   → Plan (optional)
├── duration_minutes
├── effort_points             (calculated at log time)
├── intensity_override        Easy / Moderate / Hard (optional)
├── back_comfort              Good / Neutral / Discomfort (optional)
├── pain_day                  boolean
├── notes                     free text (optional)
├── source                    manual / strava / peloton / health_connect
├── logged_at
└── created_at
```

---

### 4. Plans
> *Where you're going*

The intentional, forward-looking layer. Goals structured into trackable commitments. Plans give Record entries meaning and direction — without Plans, you're logging into a void. Plans define the intention; Record captures the execution; Momentum tracks the gap.

**Owns:**
- Plan creation and management
- Activity targets (frequency per week, minimum duration)
- Plan status lifecycle (active / on_hold / retired)
- Progression logic (plans that evolve targets over time)
- Medical context per plan (back-aware flags, cleared/excluded activity types)
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
- Plan activation (on_hold → active) should feel like a meaningful moment, not a settings toggle
- Medical context in plans (back/hip notes) is private and never surfaced to partner
- Every active plan should have a defined indoor fallback for weather-dependent activities

**Current plans (D):**
- Bike Commute — active, 2-4x/week, min 20 min, `cardio` + `low-impact`
- Strength Training — active, 2x/week, min 20 min, `strength`, back-aware
- Endurance Exploration — on_hold pending diagnosis

**Current plans (A):**
- Peloton — active, 2-3x/week, min 15 min, `cardio`
- Yoga — active, 1-2x/week, min 20 min, `flexibility` + `balance` + `mindfulness`
- Walking — active, as needed, min 15 min, `cardio` + `low-impact`

---

### 5. Momentum
> *What keeps you going*

The motivational engine. Everything that creates forward motion, celebrates progress, and sustains engagement over time. Without Momentum, Tandem is a fitness tracker. With it, Tandem is a companion. This pillar is the emotional core of the product — and the one most directly designed around Amy's psychology.

**Owns:**
- Quest engine (definitions, types, progress accumulation, completion, miss handling)
- Quest Awareness Engine (pace calculation, risk states, proactive surfacing)
- Activity Streak (daily, with Streak Shield mechanic and recovery window)
- Journey Streak (weekly, hard to break, long-arc confidence builder)
- Shared quest progress view (combined progress, both contributions visible)
- Notifications and nudges (warm, forward-looking, never punitive)
- Weekly wins summary (Sunday, wins only, no gaps surfaced)
- Celebration moments (completions, comebacks, personal bests, milestones)
- Longevity milestones (long-arc, non-time-bound shared achievements)
- Partner awareness (opt-in visibility into partner's activity and quest risk)
- Consistency scoring (rolling 4-week activity rate)
- Quest difficulty calibration (Easy / Medium / Hard tiers)

**Does not own:**
- Activity definitions (Library)
- Plan targets (Plans — Momentum reads them but doesn't define them)
- The actual activity records (Record — Momentum reads them)
- User multipliers (Identity — Momentum uses them for effort point aggregation)

**Key design constraints:**
- Streaks have real stakes — they can break. Removing all risk removes the motivation.
- One bad day never ends the story — Shields, recovery windows, Journey Streak ensure continuity
- The app never surfaces failure — only forward-looking messages, always
- Quest risk state UI emphasizes path forward, never the gap
- Wins-only weekly summary — zero mention of missed targets or gaps
- All Momentum copy reviewed against the "never say" list in vision.md
- Partner awareness is opt-in, default off — never assumed

---

## Pillar Interaction Rules

1. **Library is read-only to all other pillars** — no pillar redefines or overrides Library classifications
2. **Record is the single source of activity truth** — Momentum and Plans read from Record, never duplicate it
3. **Identity context flows down, never up** — pillars use Identity data but don't write back to it (except multiplier evolution, which is a deliberate, user-initiated action)
4. **Plans define intent; Momentum measures it** — a Plan says "3x/week"; Momentum reads Record entries and evaluates whether that intent was met
5. **Momentum closes the loop** — it's the only pillar that influences future behavior by feeding signals back toward Record (via nudges, suggestions, "Just Start")

---

## Future Pillar: Insights

As data accumulates, a sixth pillar may emerge:

**Insights** — the read-only synthesis layer. Quarterly functional fitness check-ins, pain day trend analysis, weight loss correlation with activity type, longevity marker tracking. Currently small enough to live within Momentum, but worth naming now so it has a home when the time comes.

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
