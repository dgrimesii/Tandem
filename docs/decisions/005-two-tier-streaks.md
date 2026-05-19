# ADR 005 — Two-Tier Streak System

**Status:** Accepted

## Context
Streaks are highly motivating for one user specifically because of the fear of breaking them. But a fragile streak (breaks on any missed day) creates shame spirals and app abandonment. A completely unbreakable streak has no motivational stakes.

## Decision
Two streaks run simultaneously:

**Activity Streak (daily)** — consecutive days with any qualifying activity. Has real stakes. Protected by Streak Shields (earned weekly). Includes a 48-hour recovery window before breaking. Can break — resets with dignity, personal best preserved permanently.

**Journey Streak (weekly)** — consecutive weeks hitting minimum weekly goal. Very hard to break. Survives individual bad days. Represents the real long-term pattern.

## Consequences
- Activity Streak preserves motivational stakes (it can break)
- Journey Streak ensures something is always unbroken, always reflecting the bigger picture
- Streak Shields reward consistent weeks and provide calculated protection
- Personal best is permanent — breaking a streak never erases the record
- Recovery window gives a real path forward without silently forgiving the miss
