# 🛡️ Email Phishing Detection

> **Real-time phishing analysis for Microsoft Outlook — GPT-4 header analysis with HIGH/CRITICAL-only Slack alerting**

[![Status](https://img.shields.io/badge/Status-Active-green)]()
[![Category](https://img.shields.io/badge/Category-SecOps-red)]()
[![n8n](https://img.shields.io/badge/n8n-v1.121.3-orange)]()
[![License](https://img.shields.io/badge/License-MIT-lightgrey)]()

---

## 📋 Table of Contents
- [Overview](#-overview)
- [Workflow Architecture](#-workflow-architecture)
- [Risk Classification](#-risk-classification)
- [Node Table](#-node-table)
- [Credentials Required](#-credentials-required)
- [Quick Start](#-quick-start)
- [Customisation](#-customisation)
- [Error Handling](#-error-handling)
- [Technologies Used](#️-technologies-used)
- [Known Limitations](#-known-limitations)
- [Contact](#-contact)

---

## 🎯 Overview

Polls Microsoft Outlook every minute for new emails. For each email, retrieves the full message headers via the **Microsoft Graph API**, extracts SPF, DKIM, DMARC results, IP addresses, and Return-Path, and passes this metadata to **GPT-4** for phishing risk assessment. Only **HIGH** and **CRITICAL** risk emails trigger a Slack alert. LOW and MEDIUM are silently ignored.

### Why This Matters
- **Header analysis is complex** — SPF, DKIM, DMARC, spoofing indicators, IP reputation. Delegating this to GPT-4 replaces what would otherwise be a multi-step rules engine with a single intelligent assessment.
- **Alert fatigue is real** — learned from experience. Alerting on MEDIUM risk creates noise that causes teams to ignore the alerts that actually matter. HIGH/CRITICAL only.
- **Metadata only** — email bodies are never sent to OpenAI. Only headers, subject, sender, and recipients. Privacy-by-design.
- **Tested to zero false positives** — validated against KnowBe4 phishing simulation emails before production activation.

### Key Features
- ✅ Minute-level polling of Microsoft Outlook inbox
- ✅ Full header retrieval via Microsoft Graph API
- ✅ GPT-4 four-tier risk classification (LOW / MEDIUM / HIGH / CRITICAL)
- ✅ HIGH and CRITICAL only → Slack alert
- ✅ Email body never sent externally

---

## 🏗️ Workflow Architecture

```
Microsoft Outlook Trigger (polls every 1 min)
              │
              ▼
Retrieve Email Headers (Microsoft Graph API)
              │
              ▼
Format Headers
(extracts SPF / DKIM / DMARC / IPs / Return-Path)
              │
              ▼
Set Email Variables
(subject, sender, recipients, formatted headers)
              │
              ▼
Analyse Email with GPT-4
(returns: LOW / MEDIUM / HIGH / CRITICAL + reasoning)
              │
              ▼
          High Risk?
              │
     YES      │      NO
(HIGH/CRIT)  │   (LOW/MED)
              │
     ▼                ▼
Send Slack         No Action
Security Alert     Required
```

---

## 🚨 Risk Classification

| Rating | Action | Description |
|--------|--------|-------------|
| ✅ LOW | Silent — no alert | Passes authentication, no spoofing indicators |
| 🟡 MEDIUM | Silent — no alert | Minor anomalies, not actionable |
| 🔴 HIGH | Slack alert sent | Failed auth headers or clear spoofing indicators |
| 🚨 CRITICAL | Slack alert sent | Active phishing attempt, impersonation, or malicious indicators |

> **Design principle**: LOW and MEDIUM are the expected baseline. Alerting on them creates noise that causes real threats to be missed. Only escalate what requires human action.

---

## 📊 Node Table

| # | Node | Type | Purpose |
|---|------|------|---------|
| 1 | Microsoft Outlook Trigger | Outlook | Polls every minute for unread emails |
| 2 | Retrieve Email Headers | HTTP Request | Calls Microsoft Graph API for full headers |
| 3 | Format Headers | Code | Extracts SPF, DKIM, DMARC, IPs, Return-Path |
| 4 | Set Email Variables | Set | Maps subject, sender, recipients, headers for GPT-4 prompt |
| 5 | Analyse Email with GPT-4 | OpenAI | Returns LOW / MEDIUM / HIGH / CRITICAL risk verdict |
| 6 | High Risk? | If | Passes only HIGH or CRITICAL to Slack |
| 7 | Send Slack Security Alert | Slack | Posts alert to security channel |
| 8 | No Action Required | No-Op | Silent end for LOW/MEDIUM emails |

---

## 🔑 Credentials Required

| Credential | Type | Used By |
|------------|------|---------|
| Microsoft Outlook OAuth2 | n8n Microsoft OAuth2 | Outlook Trigger + Graph API header retrieval |
| OpenAI API Key | n8n OpenAI credential | Analyse Email with GPT-4 |
| Slack OAuth2 | n8n Slack credential | Send Slack Security Alert |

---

## 🚀 Quick Start

### Prerequisites
- n8n instance (self-hosted or cloud)
- Microsoft 365 account with Outlook access
- OpenAI API key (GPT-4 access required)
- Slack workspace with bot permissions

### Setup Steps

**1. Import the workflow**
```bash
n8n import:workflow --input=email-phishing-detection.json
```

**2. Configure credentials**

In n8n → Credentials, create:
- **Microsoft Outlook OAuth2** — requires Azure app registration with `Mail.Read` and `Mail.ReadWrite` permissions + Graph API access
- **OpenAI** — API key with GPT-4 access
- **Slack OAuth2** — bot token with `chat:write` scope

**3. Set your Slack channel**

In `Send Slack Security Alert`, update `channelId` to your security alerts channel.

**4. Validate against test emails**

Before activating in production, send known phishing test emails (e.g. KnowBe4 simulations) and verify the classification is correct. Tune the GPT-4 system prompt to match your organisation's risk tolerance.

**5. Activate**

Enable the workflow. It begins polling Outlook every minute immediately.

---

## ⚙️ Customisation

| What | Where | How |
|------|-------|-----|
| Poll frequency | Microsoft Outlook Trigger | Change interval — every minute / 5 min / hourly |
| Alert threshold | High Risk? (If node) | Add MEDIUM to the condition to lower the threshold |
| Slack channel | Send Slack Security Alert | Update `channelId` |
| AI model | Analyse Email with GPT-4 | Swap to GPT-4o or Claude if needed |
| Risk prompt | Analyse Email with GPT-4 | Tune the system prompt to your organisation's profile |
| Alert message format | Send Slack Security Alert | Edit the `text` field |

---

## 🛡️ Error Handling

- Graph API call uses `onError: continueRegularOutput` — a header retrieval failure won't stop the workflow
- No retry logic is currently configured — transient Graph API failures will be silently skipped until the next poll cycle
- Add an Error Trigger node connected to Slack for workflow-level failure notification in production

---

## 🛠️ Technologies Used

| Tool | Version | Purpose |
|------|---------|---------|
| n8n | v1.121.3 | Workflow orchestration |
| Microsoft Outlook | - | Email source |
| Microsoft Graph API | v1.0 | Full email header retrieval |
| OpenAI GPT-4 | Latest | Phishing risk classification |
| Slack | API v2 | Security alert delivery |

### Why These Tools?

**Microsoft Graph API** — The Outlook trigger node alone doesn't return full headers. Graph API provides the raw `internetMessageHeaders` array needed for SPF/DKIM/DMARC analysis. One extra HTTP Request node, significant improvement in signal quality.

**GPT-4** — Header analysis involves interpreting combinations of SPF pass/fail, DKIM signatures, DMARC alignment, IP reputation, and Return-Path mismatches. A rules engine would need dozens of conditions. GPT-4 handles this in a single prompt and explains its reasoning — useful for tuning.

**HIGH/CRITICAL only** — Deliberate design. Tested against KnowBe4 simulations. FALSE POSITIVE RATE TARGET: zero. If a legitimate email is misclassified as HIGH, trust in the system breaks down. Alert only when certain.

---

## ⚠️ Known Limitations

- **Poll-based, not real-time** — 1-minute polling means up to 60 seconds between email arrival and alert. For instant response, replace with a Microsoft Graph webhook trigger (requires webhook endpoint accessible from the internet).
- **GPT-4 prompt needs periodic tuning** — phishing tactics evolve. Review the system prompt quarterly.
- **No email screenshot capture** — investigated and deferred. Screenshot API services add cost and complexity with marginal detection improvement given header-only analysis already catches the key indicators.
- **Microsoft 365 OAuth complexity** — Azure app registration with correct Graph API permissions is the most common setup failure. See `.env.example` for required permission scopes.

---

## 📫 Contact

**Tony O'Connor**
- GitHub: [@digitaloconnor](https://github.com/digitaloconnor)
- n8n Instance: [automation.fioslabs.org](https://automation.fioslabs.org)

---

## 🔖 Tags

`#n8n` `#automation` `#phishing` `#secops` `#email-security` `#microsoft-outlook` `#graph-api` `#openai` `#gpt4` `#slack` `#security-automation` `#threat-detection`

---

**⚡ Status**: Active | **🗂️ Category**: SecOps | **📅 Last Updated**: 2026-03-26
