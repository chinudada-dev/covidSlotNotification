# CoWIN Vaccine Slot Notifier

A Python script that polls the Indian government's CoWIN API for available COVID-19 vaccination slots across specified districts and sends a WhatsApp alert via Twilio when an 18+ eligible slot is found.

---

## Overview

The script checks vaccine slot availability for the **next 14 days** across a configurable list of districts. When it detects an open session for individuals aged 18 and above, it immediately dispatches a WhatsApp notification with the centre name, address, pincode, and date.

---

## Requirements

| Package | Version |
|---|---|
| Python | ≥ 3.6 |
| requests | ≥ 2.20 |
| twilio | ≥ 7.0 |

Install dependencies:

```bash
pip install requests twilio
```

---

## Configuration

Before running, update the following values directly in the script:

### Districts

```python
districtList = [265, 294]
```

Replace with the CoWIN district IDs you want to monitor. District IDs can be looked up via the [CoWIN public API](https://apisetu.gov.in/public/marketplace/api/cowin).

### Twilio Credentials

```python
account_sid = 'YOUR_ACCOUNT_SID'
auth_token  = 'YOUR_AUTH_TOKEN'
```

Obtain these from your [Twilio Console](https://console.twilio.com/).

### WhatsApp Numbers

```python
from_='whatsapp:+14155238886',   # Twilio sandbox number (or your approved sender)
to='whatsapp:+919901595582'      # Recipient's WhatsApp number with country code
```

> **Note:** The Twilio WhatsApp Sandbox requires the recipient to opt-in by sending a join message to the sandbox number before messages can be received.

---

## Usage

```bash
python vaccine_notifier.py
```

The script runs once, scanning all 14 upcoming dates × all configured districts. For continuous monitoring, schedule it with `cron` (Linux/macOS) or Task Scheduler (Windows):

```bash
# Example: run every 15 minutes
*/15 * * * * /usr/bin/python3 /path/to/vaccine_notifier.py
```

---

## How It Works

```
Generate next 14 dates
       ↓
For each date × each district
       ↓
Call CoWIN findByDistrict API
       ↓
Filter sessions where min_age_limit == 18
       ↓
Send WhatsApp alert via Twilio for each match
```

### Sample WhatsApp Message

```
Your vaccine appointment at Safdarjung Hospital:: Ansari Nagar, New Delhi:: 110029 on 5-5-2021 is now open
```

---

## API Reference

**CoWIN Public API — Find Sessions by District**

```
GET https://cdn-api.co-win.in/api/v2/appointment/sessions/public/findByDistrict
    ?district_id={id}
    &date={DD-M-YYYY}
```

No authentication is required for this public endpoint.

---

## Security Warning

> ⚠️ **Never commit your Twilio `account_sid` or `auth_token` to version control.**

Move credentials to environment variables before sharing or deploying:

```python
import os
account_sid = os.environ['TWILIO_ACCOUNT_SID']
auth_token  = os.environ['TWILIO_AUTH_TOKEN']
```

---

## Known Limitations

- The script runs once per execution — no built-in polling loop.
- A Twilio message is sent per matching session per run, which may result in duplicate alerts if run frequently.
- The CoWIN API may enforce rate limits or return errors during peak load — no retry logic is implemented.
- The date format passed to the API (`D-M-YYYY`) may produce incorrect results for single-digit days/months without zero-padding; verify against the API's expected format.
