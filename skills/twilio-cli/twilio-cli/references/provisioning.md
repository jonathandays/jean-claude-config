# Twilio Provisioning in Puny.bz

## File: `profiles/services/twilio_provisioning.py`

The `TwilioProvisioningService` handles automated Twilio setup per organization.

## Region Configuration

```python
REGION_CONFIG = {
    'PR': {'country': 'PR', 'area_code': None, 'name': 'Puerto Rico'},  # PR is its own country in Twilio!
    'US': {'country': 'US', 'area_code': None, 'name': 'United States'},
    'CO': {'country': 'CO', 'area_code': None, 'name': 'Colombia'},
    'MX': {'country': 'MX', 'area_code': None, 'name': 'Mexico'},
}
```

**Gotcha:** Twilio treats Puerto Rico as country code `PR`, not `US` with area code 787/939. Using `US` + `area_code=787` returns zero results.

## Provisioning Flow

1. **`provision_organization(org_id, org_name, region)`** — orchestrates the full flow
2. **`create_subaccount(org_id, org_name)`** — creates Twilio subaccount named `PunyCenter-{org_id}-{org_name[:30]}`
3. **`buy_phone_number(subaccount_sid, subaccount_token, region)`** — searches voice+SMS enabled local numbers, falls back to toll-free
4. **`create_messaging_service(subaccount_sid, subaccount_token, org_name)`** — creates SMS routing service
5. **`configure_voice_webhook(subaccount_sid, subaccount_token, phone_sid)`** — sets voice webhook URL

## Data Model: `PunyCenterConfig`

File: `profiles/models.py` (lines 1789-1938)

| Field | Type | Purpose |
|-------|------|---------|
| `org` | OneToOneField | Link to Organization |
| `twilio_subaccount_sid` | CharField | Subaccount SID (ACxxx) |
| `twilio_subaccount_token` | CharField | Subaccount auth token |
| `phone_number` | CharField | E.164 format, unique |
| `phone_number_sid` | CharField | Twilio phone SID (PNxxx) |
| `messaging_service_sid` | CharField | Messaging Service SID (MGxxx) |
| `phone_region` | CharField | Region code (PR/US/CO/MX) |
| `status` | CharField | pending/provisioning/active/failed |

## Data Model: `TwilioProfile`

File: `profiles/models.py` (lines 937-1046)

Legacy org-level Twilio config with `send_sms()`, `send_mms()`, `call()`, `lookup()` methods.

## A2P 10DLC Status

**Not implemented.** No brand registration, campaign registration, or Trust Product code exists. Standard local numbers are used without A2P compliance registration.
