---
name: telegram-ping
description: Ping Mohamed on Telegram when you need them to act — work blocked, a decision or credential only they can provide, or you're waiting. Manual /telegram-ping <message>. Once invoked in a session, arms a session-long duty — ping at every stop: needing Mohamed, end of a round of work, or session end.
---

# Telegram ping

## Once invoked, armed for the whole session

The first invocation turns this duty ON until the session ends. From then on, ping at every:

1. **Needs Mohamed** — input, decision, credential, or an action only he can do. Ping the moment it appears, not at the end of the turn. Tag `Needs you:` / `Decision needed:` / `Blocked:`.
2. **Round of work ends** — your turn finishes and control returns to him. Tag `Done:` plus a one-line outcome.
3. **Session ends** — final ping, tag `Ending:`.

Nothing switches the duty off. If unsure whether a stop counts, it counts.

## Message format

Multi-line, four blocks, sent with `parse_mode=HTML`:

```
<b>🔴 [task name]</b>
<i>one-line description</i>

<u>Tag:</u> headline sentence
payload…
```

- **Line 1 — header:** `<b>` + severity emoji + `[task name]`, the decision or task this session is working on (issue, branch, or brief name). Several agents ping Mohamed at once, and the chat-list preview shows only this line — it must tell him who pinged and how urgent, alone.
- **Line 2 — description:** `<i>` one line on what this session is doing. Omit it when the header already says it.
- **Blank line, then the stop:** `<u>Tag:</u>` plus a one-line headline of the stop and its cost.
- **Payload** (only when it carries information): `<code>` marks identifiers in line (branch, file, port, command); `<blockquote>` quotes an error or message; `<blockquote expandable>` when the payload runs past ~4 lines — Telegram renders it collapsed, tap to expand; `<pre><code class="language-…">` for a stack trace or log tail, the only real text colors Telegram offers. Inline ⚠️ / ℹ️ markers when the payload mixes severities.

Severity emoji on line 1, by tag:

| Tag | Emoji |
| --- | --- |
| `Blocked:` | 🔴 |
| `Needs you:` / `Decision needed:` | 🟠 |
| `Done:` | 🟢 |
| `Ending:` | 🔵 |

Escape literal `& < >` as `&amp; &lt; &gt;`. Header + description + headline ≤ ~300 chars (the glance cost); total ≤ ~1500 (Telegram hard limit 4096). No secrets, tokens, or connection strings.

## Setup — once per machine

If `~/.config/telegram-ping/chat-id` doesn't exist, ask the user for their chat ID, then save it locally (never commit it):

```bash
mkdir -p ~/.config/telegram-ping && printf '%s\n' '<their chat id>' > ~/.config/telegram-ping/chat-id
```

## Send it

One shell command, `MSG` set first. If the context-mode hook redirects `curl` away from Bash, run the same snippet through the `ctx_execute` tool (language `shell`).

```bash
set -a; source /root/.claude/channels/telegram/.env; set +a
CHAT_ID=$(cat ~/.config/telegram-ping/chat-id)
curl -sf --max-time 10 -X POST "https://api.telegram.org/bot$TELEGRAM_BOT_TOKEN/sendMessage" \
  -d chat_id="$CHAT_ID" -d parse_mode=HTML --data-urlencode text="$MSG" >/dev/null && echo "telegram: sent" || echo "telegram: SEND FAILED"
```

If the chat-id file is missing, run Setup before sending — never guess an ID.

## Rules

- One ping per stop moment. Never skip one, never batch several into one.
- If the send fails, say so — never claim a ping that didn't happen. On HTTP 400 (malformed HTML) retry once with every tag stripped — a degraded ping beats a missing one; any other failure, retry once as-is, then report.
