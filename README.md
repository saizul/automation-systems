# automation-systems

Importable [n8n](https://n8n.io) workflows for everyday automation, with the description and a screenshot of each one.

[![Validate workflows](https://github.com/saizul/automation-systems/actions/workflows/validate.yml/badge.svg)](https://github.com/saizul/automation-systems/actions/workflows/validate.yml)

| Workflow | File | What it does |
|----------|------|--------------|
| Daily Email Summary | [`Daily Email Summary.json`](Daily%20Email%20Summary.json) | Every morning, reads the last 24 hours of unread Gmail, has an AI rank each email by priority and sends a short Bengali summary to Telegram |

## Daily Email Summary

![Daily Email Summary workflow](Workflow%20Screenshot%20.png)

**What it does.** At 10:00 (in the timezone set on your n8n instance) it fetches up to 20 Gmail messages matching `is:unread newer_than:1d`, formats sender, subject and a 200-character preview of each into a numbered list, and asks an LLM to:

1. give each email a priority (urgent, important or normal),
2. summarise only the urgent and important ones, in Bengali,
3. say which emails need a reply today,
4. open with a three-line overview.

The result is sent to a Telegram chat with the total email count.

**Flow.** `Schedule Trigger` -> `Get many messages` (Gmail) -> `Code` (builds the email list and count) -> `AI Agent` (Groq Chat Model, `llama-3.3-70b-versatile`) -> `Send a text message` (Telegram).

**Nodes used.** Schedule Trigger, Gmail, Code, AI Agent, Groq Chat Model, Telegram.

**Credentials needed** (create them in n8n, then select them on each node after import):

| Credential type | Used by | Notes |
|-----------------|---------|-------|
| Gmail OAuth2 | Get many messages | Read access to the mailbox |
| Groq API | Groq Chat Model | API key from the Groq console |
| Telegram API | Send a text message | Bot token from [@BotFather](https://t.me/BotFather) |

**Placeholders to replace.**

- `YOUR_TELEGRAM_CHAT_ID` in the Telegram node: the chat that should receive the summary.
- Optionally the schedule hour, the Gmail filter (`filters.q`) and the `limit` in the Gmail node.

**Setup.**

1. In n8n choose **Workflows -> Import from file** and select `Daily Email Summary.json`.
2. Attach your Gmail, Groq and Telegram credentials.
3. Set `YOUR_TELEGRAM_CHAT_ID` in the Telegram node.
4. Run the workflow once manually to check the output, then activate it.

Notes: the Telegram node sends with `parse_mode` HTML, so unusual characters in the AI output can occasionally be rejected by Telegram. The Code node has a branch for an empty list, but n8n normally stops a run when the Gmail node returns no items.

See also [`daily email summary descriptions`](daily%20email%20summary%20descriptions) for the original write-up and ideas for future improvements.

## Validation

`scripts/validate-workflows.js` checks every workflow JSON in this repo (valid JSON, well-formed nodes, connections to existing nodes, no orphaned nodes, no embedded secrets). It runs in GitHub Actions on every push and pull request and locally with:

```bash
node scripts/validate-workflows.js
```

## Author

[Saizul Amin](https://saizul.com): web developer, SEO and automation specialist.
