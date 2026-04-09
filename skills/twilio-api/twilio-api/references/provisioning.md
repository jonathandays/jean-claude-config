# Twilio Provisioning in Puny.bz

See the twilio-cli skill's `references/provisioning.md` for full details.

## Quick Reference

- **Service**: `profiles/services/twilio_provisioning.py` — `TwilioProvisioningService`
- **Model**: `profiles/models.py` — `PunyCenterConfig` (stores subaccount SID, token, phone, messaging service)
- **Regions**: PR (area code 787), US, CO, MX
- **Flow**: create_subaccount → buy_phone_number → create_messaging_service → configure_voice_webhook
- **A2P 10DLC**: Not implemented yet — no brand/campaign registration code exists
