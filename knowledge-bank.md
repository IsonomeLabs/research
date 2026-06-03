# Cross-Agent Knowledge Bank

Shared intelligence across all cron agents.
**FORCING FUNCTION**: Every agent MUST add at least one entry per active session,
or add a dated note explaining why there's nothing new. No silent runs on the bank.

## Blocker Registry

<!-- Format:
### BLOCKER: [short description] — YYYY-MM-DD
- **Blocking**: which agent/cron (e.g., isonome-improvement)
- **Depends on**: research topic slug
- **Unblock condition**: what needs to be true
- **Urgency**: critical | high | medium
- **Status**: blocked | research-in-progress | resolved
- **Resolved by**: research doc path (when done)
-->

## Architecture Patterns

<!-- Format:
### PATTERN: [name] — YYYY-MM-DD
- **What**: one-sentence description
- **Applies to**: which agents/systems
- **Source**: research doc or discovery context
- **Quality**: ★★★ (how well-validated this pattern is)
- **Usage count**: how many times it's been reused
-->

## Discoveries

<!-- Format:
### DISCOVERY: [title] — YYYY-MM-DD
- **Finding**: one sentence
- **Impact**: what changes because of this
- **Discovered by**: which agent
- **Validation**: confirmed | speculative
- **Source**: research doc path
-->

## Integration Points

<!-- Format:
### INTEGRATION: [system A] ↔ [system B] — YYYY-MM-DD
- **Connection**: what flows between them
- **Protocol**: how (signal, file, API, shared state)
- **Status**: active | proposed | broken
-->

## Quality Feedback (cross-agent ratings)

<!-- Format:
### RATING: [producing agent]'s [output] — YYYY-MM-DD
- **Rated by**: [consuming agent]
- **Score**: 1-5 (1=useless, 3=useful, 5=breakthrough)
- **What worked**: specific things that were valuable
- **What didn't**: specific things that were wrong or missing
- **Would reuse**: yes | with modifications | no
-->

## Session Summaries

<!-- Every agent adds one line per session -->
<!-- Format: [AGENT] YYYY-MM-DD HH:MM — [did this] | blockers: [N] | discoveries: [N] | bank entries: [N] -->
