# Changelog

All notable changes to this workflow are documented here.

Format: `[Version] — YYYY-MM-DD`

---

## [1.0] — 2026-03-26

### Production release

- Microsoft Outlook trigger polling every minute via Graph API
- Full header extraction: SPF, DKIM, DMARC, IPs, Return-Path
- GPT-4 phishing risk analysis with LOW/MEDIUM/HIGH/CRITICAL output
- IF node routing — HIGH and CRITICAL only proceed to Slack alert
- LOW and MEDIUM silently terminated (No Action Required node)
- Slack alert posted to security channel on HIGH/CRITICAL
- 6 sticky notes covering all workflow sections
- Tested against KnowBe4 phishing simulation suite — false positive rate: zero

---

## [0.5] — 2026-02-20

### Beta

- Core header extraction and GPT-4 analysis working
- Alert threshold testing — initial version alerted on MEDIUM and above
- Tuned down to HIGH/CRITICAL only after alert fatigue testing
- Screenshot capture investigated and ruled out

---

## [0.1] — 2026-01-15

### Initial build

- Outlook trigger + basic GPT-4 prompt
- Single-path — no risk level routing
- All emails produced an alert (unsustainable, replaced in 0.5)
