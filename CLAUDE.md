# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

**Duo** — an MVP for co-parenting a 3-year-old child, shared between two separated parents. Goal:
reduce friction and conflict in day-to-day custody logistics, keep both parents informed about the
child's daily life, and give them a low-conflict channel to communicate.

Current state: a single-file mobile-first HTML/CSS/JS prototype ([duo.html](duo.html)), published as
a Claude Artifact at https://claude.ai/artifact/3aMEhb1GD5PH5B8DAzEgZ6. All data is mocked/hardcoded
in JS; there is no backend and no real multi-user sync — user-added data persists only in the
current browser's `localStorage`, per viewer.

## Functional scope (5 features)

1. **Shared custody calendar** — alternating weekly custody (Léa / Karim) shown as a colored month
   grid, a "this week" banner naming who has the child, and a list of upcoming handoff days with a
   "request a swap" action (flags the day, no real notification).
2. **Child's journal** — timestamped entries logged by whichever parent had the child that day:
   meal, sleep, health, mood, event. Filterable by type, with an entry counter and an "add entry"
   bottom sheet.
3. **Shared health record** ("Santé" tab) — blood type, allergies, latest growth measurement
   (weight/height), a vaccination list (with a "mark as done" toggle for upcoming ones), and current
   prescriptions (dosage, prescriber). Read-only demo data except the vaccination toggle.
4. **Low-conflict messaging** — a shared thread with quick templates ("Info santé", "Logistique",
   "Demande") meant to keep tone neutral and on-topic, plus a lightweight "Envoyé / Vu" read-status
   on the current user's own messages.
5. **Mediation mode** — before sending, `isTense(text)` flags a message as potentially conflictual
   (repeated `!`, ALL-CAPS shouting, or a small dictionary of tense phrases like "jamais" / "ras le
   bol" / "comme d'habitude"). A banner then offers `mockNeutralRewrite(text)`, a rule-based (not
   LLM-based) softened rewrite, or the option to send the original unchanged.

A parent switcher in the header ("Léa" / "Karim") sets which identity is "you" — it drives message
authorship, journal entry authorship, and read-status logic. There is no real authentication.

## Structure

- [duo.html](duo.html) — the entire app: inline `<style>`, inline `<script>` (IIFE), no build step,
  no dependencies. Open directly in a browser.
  - Mock data: `MOCK_JOURNAL`, `MOCK_MESSAGES`, `MOCK_HEALTH` and `assignedParent(day)` (weekly
    alternation logic) near the top of the script.
  - State: a single `store` object (`{ swaps, journal, messages, health, parent }`) persisted to
    `localStorage` under the key `duo-coparent-mvp` via `loadStore()`/`saveStore()`. Falls back to
    the mock data on first load.
  - Four views (`view-cal`, `view-journal`, `view-sante`, `view-msg`) toggled by the bottom tab bar;
    each has its own `render*()` function re-run after every state change.
  - Mediation mode lives entirely in the messaging composer logic: `isTense()` / `mockNeutralRewrite()`
    plus the `#mediation-banner` element and its two action buttons.

## Known limitations / next steps

- No shared backend: two different parents on two different devices do **not** see the same data —
  this is the main gap before it's usable for real co-parenting (needs a real DB + sync, e.g.
  Postgres/Supabase, once Node.js tooling is available on this machine).
- No authentication — the parent switcher is a UI toggle, not a login.
- Custody alternation is a fixed weekly pattern hardcoded for September 2026; a real version needs a
  configurable custody schedule (weekly, alternating weeks, custom).
- "Swap request" and message "Vu" status are purely local/simulated, no notification to the other
  parent.
- Health record is display-only mock data (no add/edit form yet) apart from the vaccination toggle.
- Mediation rewrite is a simple rule-based heuristic, not a real LLM call — good enough to
  demonstrate the UX, not to catch every tense phrasing.
