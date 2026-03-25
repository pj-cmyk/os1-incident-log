# Known Failure Patterns

Recurring operational failure modes observed across OS-1 agents, with mitigations.

---

## 1. Context Overflow Cascade

**Pattern:** Heavy tool usage → context bloat → compaction → context loss → re-orientation tool calls → more bloat → repeat

**Triggers:**
- `sessions_list` without limits (10-40K tokens per call)
- `sessions_history` on active sessions
- Large code file reads without truncation
- Screenshot analysis in rapid succession

**Mitigations:**
- Always use `limit` parameter on list commands
- Pipe exec output through `head -n 50` or similar
- Use sub-agents for heavy debugging
- Read Den cache snapshot instead of calling session tools

---

## 2. Bootstrap Failure After Restart

**Pattern:** Gateway restart → fresh session with no context → model defaults to NO_REPLY or asks "what were we discussing?"

**Triggers:**
- Gateway restart (manual or crash recovery)
- Compaction with lossy summarization
- Session timeout/expiry

**Mitigations:**
- Den cache snapshot (rsync to VPS every 30 min)
- AGENTS.md hard rule: read snapshot before any tool calls
- HEARTBEAT.md bootstrap instruction
- `memory_search` for lightweight orientation (~200 tokens vs 20K)

---

## 3. Systemd Restart Loop

**Pattern:** Gateway crashes → systemd restarts → new instance fails (lock held) → counted as failure → another restart → infinite loop

**Triggers:**
- `Restart=on-failure` without `StartLimitBurst`
- Short `RestartSec` (10s) not allowing lock release

**Fix:**
```ini
StartLimitBurst=5
StartLimitIntervalSec=300
RestartSec=30
```

---

## 4. Stale Process Blocking

**Pattern:** Old gateway process survives config reload, holds port/lock, new processes can't start

**Triggers:**
- `systemctl daemon-reload` without process restart
- Partial crash leaving orphan process

**Fix:**
- Kill stale PID manually: `kill <pid>`
- Then `systemctl start clawdbot-gateway`

---

## 5. Cross-Session Error Contamination

**Pattern:** Error message from one session gets included in another session's context, causing confusion or cascading failures

**Triggers:**
- `sessions_list` pulling error states from other sessions
- Compaction summaries including error text
- Tool outputs containing stack traces

**Mitigations:**
- Session isolation (don't query other sessions' history)
- Clean error handling in tool outputs
- Sub-agents for risky operations

---

## 6. Silent Channel Disconnection

**Pattern:** Channel (Slack, WhatsApp) appears connected but stops receiving inbound messages

**Triggers:**
- Socket mode timeout without reconnect
- Gateway restart not re-establishing all channels
- Auth token expiry

**Diagnosis:**
- Check gateway logs for inbound vs outbound activity
- Verify socket/webhook health
- Look for "pong wasn't received" or timeout errors

**Fix:** Gateway restart usually resolves; may need channel re-auth
