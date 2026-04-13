# Email Phishing Detection

**Version:** 1.0  
**Status:** Production  
**Category:** SecOps  
**Last Updated:** 2026-03-26

> **Impact:** Automated phishing triage on every inbound email. GPT-4 handles the complexity of SPF/DKIM/DMARC analysis. Alerts only on HIGH and CRITICAL risk — silence on everything else. False positive rate reduced to zero against KnowBe4 test suite.

---

## What It Does

Monitors incoming Microsoft Outlook emails in real time, extracts and analyses email headers using GPT-4, and sends a Slack security alert only if the email is rated **HIGH** or **CRITICAL** risk. Low and medium risk emails are silently ignored.

The design principle: alert fatigue is a real cost. A workflow that alerts on everything trains humans to ignore alerts. This one doesn't.

---

## Workflow Diagram

```
Microsoft Outlook Trigger
        │
        ▼
Retrieve Email Headers (Microsoft Graph API)
        │
        ▼
Format Headers (SPF / DKIM / DMARC / IPs)
        │
        ▼
Set Email Variables (subject, sender, recipients, headers)
        │
        ▼
Analyse Email with GPT-4
        │
        ▼
High Risk? (IF node)
   ├── TRUE  (HIGH / CRITICAL) → Send Slack Security Alert
   └── FALSE (LOW / MEDIUM)   → No Action Required
```

---

## Node Reference

| # | Node | Type | Purpose |
|---|------|------|---------|
| 1 | Microsoft Outlook Trigger | Outlook | Polls every minute for unread emails |
| 2 | Retrieve Email Headers | HTTP Request | Calls Microsoft Graph API for full headers |
| 3 | Format Headers | Code | Extracts SPF, DKIM, DMARC, IPs, Return-Path |
| 4 | Set Email Variables | Set | Maps subject, sender, recipients, headers for prompt |
| 5 | Analyse Email with GPT-4 | OpenAI | Assesses phishing risk — returns LOW/MEDIUM/HIGH/CRITICAL |
| 6 | High Risk? | IF | Filters — only passes HIGH or CRITICAL downstream |
| 7 | Send Slack Security Alert | Slack | Posts alert to security channel |
| 8 | No Action Required | No-Op | Silent end for low-risk emails |

14 nodes total including sticky note documentation.

---

## Risk Levels

| Rating | Action | Rationale |
|--------|--------|-----------|
| 🟢 LOW | Silent — no alert | Not actionable |
| 🟡 MEDIUM | Silent — no alert | Not immediately actionable |
| 🔴 HIGH | Slack alert sent | Requires human review |
| 🚨 CRITICAL | Slack alert sent | Requires immediate human action |

LOW and MEDIUM are deliberately ignored. Alerting on every suspicious signal creates noise that causes humans to ignore the real ones.

---

## What GPT-4 Analyses

Header analysis is complex. SPF, DKIM, and DMARC validation, IP reputation, Return-Path mismatches, and spoofing indicators are delegated to GPT-4 — which outputs a plain English risk verdict. No regex. No brittle rule sets.

The prompt is the core of this workflow. Review and tune it periodically as phishing tactics evolve.

---

## Credentials Required

| Credential | Node(s) | Purpose |
|------------|---------|---------|
| Microsoft Outlook OAuth2 | Outlook Trigger, Retrieve Email Headers | Access mailbox and full headers via Graph API |
| OpenAI API Key | Analyse Email with GPT-4 | GPT-4 phishing analysis |
| Slack OAuth | Send Slack Security Alert | Post to security channel |

See `.env.example` for required environment variable names.

---

## Setup

1. **Import** `Email_Phishing_Detection.json` into n8n
2. **Connect credentials** — Outlook OAuth2, OpenAI API key, Slack
3. **Update Slack channel** — default is `#n8n-testing-space`. Change the channel ID in the Slack node to your production security channel
4. **Review the GPT-4 prompt** — in `Analyse Email with GPT-4`, check the system prompt matches your risk tolerance
5. **Test** — use `test-data.json` to verify the IF node routing before going live
6. **Activate** — workflow polls Outlook every minute

---

## Customisation

| What | Where | How |
|------|-------|-----|
| Poll frequency | Microsoft Outlook Trigger | Change interval (every minute / hourly / etc.) |
| Risk threshold | High Risk? (IF node) | Add or remove HIGH/CRITICAL conditions |
| Slack channel | Send Slack Security Alert | Update `channelId` |
| AI model | Analyse Email with GPT-4 | Swap GPT-4 for another model |
| Alert message format | Send Slack Security Alert | Edit the `text` field |
| GPT-4 system prompt | Analyse Email with GPT-4 | Tune for your environment and risk appetite |

---

## Known Limitations

- **Poll-based trigger** — polls every minute, not true real-time. For instant response, replace with a Microsoft Graph webhook trigger
- **Screenshot analysis not implemented** — screenshot services (Urlbox, Screenshotapi) were evaluated but added unreliable complexity with no measurable improvement in detection accuracy
- **Prompt quality matters** — GPT-4 analysis is only as good as the system prompt. Review it periodically as phishing tactics evolve

---

## Related Workflows

| Workflow | Relationship |
|----------|-------------|
| `Analyze & Sort Suspicious Email Contents with ChatGPT.json` | Similar pattern — sorts emails into categories rather than risk levels |
| `Check Suspicious Links via Telegram with GPT-4.json` | Companion — checks suspicious URLs flagged in emails |
