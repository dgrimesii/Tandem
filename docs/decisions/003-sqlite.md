# ADR 003 — SQLite as Initial Database

**Status:** Accepted

## Context
Two-user private app on a Raspberry Pi. Simplicity and zero-config operation matter more than scale.

## Decision
Use SQLite for Phase 1. Migrate to PostgreSQL if write concurrency or data volume becomes an issue.

## Consequences
- Zero configuration, runs anywhere
- Sufficient for two users indefinitely
- Backup is a single file copy
- Migration path to PostgreSQL is straightforward if ever needed
