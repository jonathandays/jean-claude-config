---
name: twilio-api
description: Use when working with the Twilio REST API — creating subaccounts, purchasing phone numbers, sending SMS/MMS, making calls, managing messaging services, addresses, usage records, A2P 10DLC registration, or any Twilio HTTP API call. Trigger on any mention of Twilio API, Twilio REST, Twilio SDK, twilio-python, twilio-node, subaccount creation, phone number provisioning, A2P registration, 10DLC, brand registration, campaign registration, Twilio webhooks, or "how do I call the Twilio API for X". Use this skill even for simple questions about Twilio API authentication or endpoint structure.
---

# Twilio REST API

Twilio's APIs are organized around REST. Each product (Messaging, Voice, Phone Numbers) has its own API, but they share common authentication, pagination, and error handling patterns.

## Authentication

All requests use HTTP Basic Auth: `AccountSid:AuthToken` or `ApiKeySid:ApiKeySecret`.

```bash
# Basic Auth with Account SID + Auth Token
curl -X GET "https://api.twilio.com/2010-04-01/Accounts/ACXXXXXXXX.json" \
  -u "ACXXXXXXXX:your_auth_token"

# Or with API Key (preferred in production)
curl -X GET "https://api.twilio.com/2010-04-01/Accounts/ACXXXXXXXX.json" \
  -u "SKXXXXXXXX:your_api_secret"
```

### Python SDK

```python
from twilio.rest import Client

# With Account SID + Auth Token
client = Client("ACXXXXXXXX", "your_auth_token")

# With API Key (preferred)
client = Client("SKXXXXXXXX", "your_api_secret", "ACXXXXXXXX")

# For subaccount operations
client = Client("ACXXXXXXXX", "your_auth_token")
subaccount_client = Client("SUBACCOUNT_SID", "subaccount_auth_token")
```

### Node.js SDK

```javascript
const twilio = require("twilio");

// With Account SID + Auth Token
const client = twilio("ACXXXXXXXX", "your_auth_token");

// With API Key
const client = twilio("SKXXXXXXXX", "your_api_secret", { accountSid: "ACXXXXXXXX" });
```

## Accounts (Subaccounts)

Subaccounts isolate phone numbers, usage, and billing per customer/team.

### Create Subaccount

```
POST https://api.twilio.com/2010-04-01/Accounts.json
```

| Parameter | Type | Description |
|-----------|------|-------------|
| FriendlyName | string | Up to 64 chars. Default: "SubAccount Created at {timestamp}" |

```python
account = client.api.accounts.create(friendly_name="PunyCenter-42-MariasSalon")
print(account.sid)         # ACXXXXXXXX (subaccount SID)
print(account.auth_token)  # Use this for subaccount operations
```

### List Accounts

```
GET https://api.twilio.com/2010-04-01/Accounts.json
```

| Parameter | Description |
|-----------|-------------|
| friendlyName | Filter by exact name |
| status | Filter: active, suspended, closed |
| pageSize | 1-1000 (default 50) |

### Fetch / Update Account

```
GET  https://api.twilio.com/2010-04-01/Accounts/{Sid}.json
POST https://api.twilio.com/2010-04-01/Accounts/{Sid}.json
```

```python
# Suspend a subaccount
client.api.accounts(subaccount_sid).update(status="suspended")

# Reactivate
client.api.accounts(subaccount_sid).update(status="active")
```

### Account Properties

| Property | Type | Notes |
|----------|------|-------|
| sid | string | 34-char ID (AC...) |
| authToken | string | Secret credential |
| ownerAccountSid | string | Parent account |
| friendlyName | string | Up to 64 chars |
| status | enum | active / suspended / closed |
| type | enum | Trial / Full |
| dateCreated | string | RFC 2822 |
| subresourceUris | object | Related resource endpoints |

## Phone Numbers

### Search Available Numbers

```
GET https://api.twilio.com/2010-04-01/Accounts/{AccountSid}/AvailablePhoneNumbers/{CountryCode}/Local.json
GET https://api.twilio.com/2010-04-01/Accounts/{AccountSid}/AvailablePhoneNumbers/{CountryCode}/TollFree.json
GET https://api.twilio.com/2010-04-01/Accounts/{AccountSid}/AvailablePhoneNumbers/{CountryCode}/Mobile.json
```

| Parameter | Description |
|-----------|-------------|
| AreaCode | Filter by area code (e.g., 787) |
| Contains | Pattern match (e.g., 555****) |
| VoiceEnabled | true/false |
| SmsEnabled | true/false |

```python
# Search for voice+SMS numbers in Puerto Rico
# IMPORTANT: Twilio treats PR as its own country code, NOT US + area code 787
numbers = client.available_phone_numbers("PR").local.list(
    voice_enabled=True,
    sms_enabled=True,
    limit=5
)
for num in numbers:
    print(num.phone_number, num.friendly_name)

# US numbers with area code filter
numbers = client.available_phone_numbers("US").local.list(
    area_code=212,
    voice_enabled=True,
    limit=5
)
```

**Puerto Rico gotcha:** Using `available_phone_numbers("US").local.list(area_code=787)` returns nothing. Twilio requires country code `PR` for Puerto Rico numbers. No area code filter needed — all PR numbers are 787 or 939.

### Purchase a Number

```
POST https://api.twilio.com/2010-04-01/Accounts/{AccountSid}/IncomingPhoneNumbers.json
```

```python
number = client.incoming_phone_numbers.create(
    phone_number="+17875551234",
    voice_url="https://voice.puny.bz/twilio/voice",
    sms_url="https://voice.puny.bz/twilio/sms"
)
print(number.sid)  # PNXXXXXXXX
```

### Update Number Webhooks

```python
client.incoming_phone_numbers("PNXXXXXXXX").update(
    voice_url="https://voice.puny.bz/twilio/voice",
    voice_method="POST",
    sms_url="https://voice.puny.bz/twilio/sms",
    sms_method="POST"
)
```

### List Owned Numbers

```python
numbers = client.incoming_phone_numbers.list()
for num in numbers:
    print(num.phone_number, num.sid)
```

## Addresses (Regulatory Compliance)

Many countries require a physical address on file for phone number purchases.

```
POST https://api.twilio.com/2010-04-01/Accounts/{AccountSid}/Addresses.json
```

| Parameter | Required | Description |
|-----------|----------|-------------|
| CustomerName | Yes | Company/individual name |
| Street | Yes | Street address |
| City | Yes | City |
| Region | Yes | State/province |
| PostalCode | Yes | ZIP/postal code |
| IsoCountry | Yes | ISO country code (US, CO, MX) |
| EmergencyEnabled | No | E911 capability |

```python
address = client.addresses.create(
    customer_name="Maria's Salon",
    street="123 Calle Sol",
    city="San Juan",
    region="PR",
    postal_code="00901",
    iso_country="US"
)
```

## Messaging Services

Group phone numbers under a service for smart routing and compliance.

```python
# Create
service = client.messaging.v1.services.create(
    friendly_name="Maria's Salon SMS"
)

# Add phone number
client.messaging.v1.services(service.sid).phone_numbers.create(
    phone_number_sid="PNXXXXXXXX"
)
```

## Sending Messages

```python
# Via phone number
message = client.messages.create(
    from_="+17875551234",
    to="+17875559876",
    body="Your appointment is confirmed for tomorrow at 2pm."
)

# Via Messaging Service
message = client.messages.create(
    messaging_service_sid="MGXXXXXXXX",
    to="+17875559876",
    body="Your appointment is confirmed."
)

# MMS (with media)
message = client.messages.create(
    from_="+17875551234",
    to="+17875559876",
    body="Check out your new look!",
    media_url=["https://example.com/photo.jpg"]
)
```

## Making Calls

```python
call = client.calls.create(
    from_="+17875551234",
    to="+17875559876",
    url="https://voice.puny.bz/twilio/voice",  # TwiML URL
    method="POST"
)
```

## Usage Records

Track usage and costs across your account and subaccounts.

```
GET https://api.twilio.com/2010-04-01/Accounts/{AccountSid}/Usage/Records.json
```

| Parameter | Description |
|-----------|-------------|
| Category | calls, sms, recordings, totalprice, etc. |
| StartDate | YYYY-MM-DD or relative (-30days) |
| EndDate | YYYY-MM-DD |
| IncludeSubaccounts | true/false |

```python
records = client.usage.records.list(
    category="calls",
    start_date="2026-03-01",
    end_date="2026-03-31"
)
for record in records:
    print(f"{record.category}: {record.count} calls, ${record.price}")
```

## A2P 10DLC (US SMS Compliance)

Required for sending SMS to US numbers from 10-digit long codes. Three-step process:

### 1. Register Brand

Register your business identity with The Campaign Registry (TCR).

```python
# Via Twilio Trust Hub (Messaging > Trust Hub > Brands)
# Or via API:
brand = client.messaging.v1.brand_registrations.create(
    customer_profile_bundle_sid="BUXXXXXXXX",  # Pre-created profile
    a2p_profile_bundle_sid="BUXXXXXXXX"
)
```

Brand vetting determines your Trust Score (low/medium/high) which affects:
- Message throughput (SMS per second)
- Daily message limits

### 2. Register Campaign

Describe your messaging use case.

```python
campaign = client.messaging.v1.services("MGXXXXXXXX") \
    .us_app_to_person.create(
        brand_registration_sid="BNXXXXXXXX",
        description="Appointment reminders and booking confirmations",
        message_flow="Customers book appointments via puny.bz and receive confirmations via SMS",
        message_samples=[
            "Your appointment with Maria's Salon is confirmed for March 15 at 2:00 PM.",
            "Reminder: You have an appointment tomorrow at 3:00 PM. Reply STOP to opt out."
        ],
        us_app_to_person_usecase="MIXED",
        has_embedded_links=True,
        has_embedded_phone=False,
        opt_in_message="You will receive appointment reminders via SMS. Reply STOP to opt out.",
        opt_out_keywords=["STOP", "CANCEL", "UNSUBSCRIBE"],
        opt_out_message="You have been unsubscribed. No more messages will be sent.",
        help_keywords=["HELP", "INFO"],
        help_message="Reply STOP to opt out. Contact us at support@puny.bz for help."
    )
```

### 3. Associate Numbers

Link phone numbers to the campaign via the Messaging Service.

### A2P Use Cases

| Use Case | Description |
|----------|-------------|
| MIXED | Multiple purposes (most common) |
| MARKETING | Promotional messages |
| CUSTOMER_CARE | Support messages |
| DELIVERY_NOTIFICATION | Shipping/delivery updates |
| ACCOUNT_NOTIFICATION | Account alerts |
| 2FA | Two-factor authentication |

### Trust Score Impact

| Score | Throughput | Daily Cap |
|-------|-----------|-----------|
| Low | 1 SMS/sec | 2,000/day |
| Medium | 10 SMS/sec | 10,000/day |
| High | Variable | Higher limits |

## Webhooks

Twilio sends HTTP requests to your URLs when events occur.

### Webhook Security

Validate signatures to ensure requests come from Twilio:

```python
from twilio.request_validator import RequestValidator

validator = RequestValidator("your_auth_token")
is_valid = validator.validate(
    url="https://voice.puny.bz/twilio/voice",
    params=request.POST,
    signature=request.META.get("HTTP_X_TWILIO_SIGNATURE", "")
)
```

### Common Webhook Events

| Event | URL Parameter | Method |
|-------|--------------|--------|
| Incoming call | VoiceUrl | POST |
| Call status | StatusCallback | POST |
| Incoming SMS | SmsUrl | POST |
| Message status | StatusCallback | POST |

## Pagination

All list endpoints return paginated results:

```json
{
  "accounts": [...],
  "first_page_uri": "/2010-04-01/Accounts.json?PageSize=50&Page=0",
  "next_page_uri": "/2010-04-01/Accounts.json?PageSize=50&Page=1&PageToken=...",
  "page": 0,
  "page_size": 50
}
```

Follow `next_page_uri` until it's null.

## Error Handling

Twilio returns standard HTTP codes plus error codes:

| HTTP Status | Meaning |
|-------------|---------|
| 200 | Success |
| 201 | Created |
| 400 | Bad request |
| 401 | Authentication failed |
| 404 | Resource not found |
| 429 | Rate limited |
| 500 | Twilio server error |

Error response includes `code`, `message`, and `more_info` URL.

## Puny.bz Context

This project uses Twilio with subaccounts per organization:
- Master credentials: `TWILIO_ACCOUNT_SID`, `TWILIO_AUTH_TOKEN` (settings.py)
- Provisioning service: `profiles/services/twilio_provisioning.py`
- PunyDesk Bun service: `services/punydesk/` (voice + SMS gateway)
- Voice webhook: `POST voice.puny.bz/twilio/voice`
- SMS webhook: `POST voice.puny.bz/twilio/sms`
- WhatsApp detected by `whatsapp:` prefix in From header

For provisioning flow details, see `references/provisioning.md`.
