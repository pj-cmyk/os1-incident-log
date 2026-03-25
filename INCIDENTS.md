# Incident Log

## Active Issues

### [2026-03-25] Coywolf DM context churn
- **Status:** In progress
- **Owner:** Coywolf
- **Symptoms:** 5-minute compaction cycles, context loss after each compaction
- **Root causes:** 
  - Large tool payloads (`sessions_list` returning 20K+ tokens)
  - Narration overhead on every tool call
  - ~20K boot context leaving limited conversation space
- **Attempted fixes:** 
  - Skill trim (78→37 skills, ~7,800 tokens recovered)
  - rsync bootstrap (snapshot syncs from Den to VPS every 30 min)
  - AGENTS.md rule to read snapshot first
  - Systemd restart policy patch
- **Next steps:** Sub-agent for heavy debugging tasks; test cold-start bootstrap

### [2026-03-25] Slack Socket Mode inbound routing
- **Status:** Investigating (sub-agent spawned)
- **Owner:** Coywolf
- **Symptoms:** Slack connects but zero inbound messages processed; outbound works fine
- **Root cause:** Unknown — socket mode silently broken since March 24
- **Evidence:** Zero inbound logs, repeated pong timeouts

---

## Resolved Issues

### [2026-03-25] Systemd restart loop — 15,000+ attempts ✅
- **Resolution:** Patched systemd unit with `StartLimitBurst=5`, `StartLimitIntervalSec=300`, `RestartSec=30`
- **Date resolved:** 2026-03-25 20:10 UTC
- **Owner:** Coywolf

### [2026-03-25] Stale gateway process blocking restarts ✅
- **Resolution:** Killed stale PID, clean restart via systemd
- **Date resolved:** 2026-03-25 20:15 UTC
- **Owner:** Coywolf

### [2026-03-25] Anthropic API instability ✅
- **Resolution:** External issue, cleared on its own
- **Date resolved:** ~2026-03-25 19:55 UTC
- **Notes:** API overloaded errors 19:45-19:53 UTC, required 3 retries

### [2026-03-25] DM session outputting NO_REPLY incorrectly ✅
- **Resolution:** Fresh session with context loss was defaulting to NO_REPLY; fixed by sending direct nudge and establishing conversation
- **Date resolved:** 2026-03-25 20:20 UTC
- **Owner:** Coywolf
