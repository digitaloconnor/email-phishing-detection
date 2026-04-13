# Security

## Reporting a Vulnerability

If you discover a security issue in this workflow, please report it via [GitHub Issues](../../issues) or directly through LinkedIn: [linkedin.com/in/tony0connor](https://www.linkedin.com/in/tony0connor/).

Do not include credentials, email content, or header data in any issue report.

---

## Data This Workflow Touches

| Data | Source | Destination | Retained |
|------|--------|-------------|---------|
| Email subject | Microsoft Outlook | OpenAI API (prompt) | No — passed in request only |
| Email sender / recipients | Microsoft Outlook | OpenAI API (prompt) | No |
| Raw email headers | Microsoft Graph API | OpenAI API (prompt) | No |
| GPT-4 risk verdict | OpenAI API | Slack (alert only on HIGH/CRITICAL) | In Slack only |
| Email body content | Not accessed | — | Never read |

**Email body is never accessed.** Analysis is limited to headers only — subject, sender, recipients, SPF/DKIM/DMARC results, IP addresses, Return-Path.

**PII note:** Email metadata (sender, recipients, subject) is forwarded to the OpenAI API for analysis. Ensure your OpenAI data processing terms are acceptable for your organisation and comply with applicable data protection obligations (GDPR, etc.). Consider using the OpenAI API with zero data retention if available on your plan.

---

## Credential Security

- All credentials are stored in n8n's encrypted credential store or as environment variables
- API keys and OAuth tokens are **never** hardcoded in workflow JSON
- The `.env` file is **never** committed to version control — see `.gitignore`
- Rotate all credentials immediately if any are exposed
- Microsoft OAuth2 token refresh is handled by n8n automatically

---

## Alert Design

This workflow is designed to **not alert on LOW and MEDIUM risk emails**. This is intentional — alert fatigue is a security risk in itself. A security team that ignores alerts because there are too many of them is more vulnerable than one that receives fewer, higher-confidence alerts.

Only HIGH and CRITICAL risk emails produce a Slack alert requiring human review and action.

---

## Network

- Outbound calls: Microsoft Graph API, OpenAI API, Slack API
- Inbound: Microsoft Outlook trigger polls outbound — no inbound webhook exposure
- No external IP exposure from this workflow
