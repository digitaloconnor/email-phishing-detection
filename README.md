# email-phishing-detection
Real-time phishing detection for Microsoft Outlook using n8n and GPT-4. Extracts email headers, analyses SPF/DKIM/DMARC signals, and sends a Slack alert only on HIGH or CRITICAL risk. LOW and MEDIUM are silently ignored. Tested against KnowBe4 — false positive rate: zero.
