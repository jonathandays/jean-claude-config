---
name: twilio-cli
description: Use when working with the Twilio CLI — managing profiles, subaccounts, phone numbers, messaging services, webhooks, configuration, or any Twilio resource from the command line. Trigger on any mention of twilio CLI, twilio command, "twilio login", "twilio profiles", "twilio api:core", phone number purchase via CLI, configuring Twilio webhooks from terminal, or "how do I do X with Twilio CLI". Use this skill even for simple questions like "how do I list my Twilio numbers" or "how do I switch Twilio profiles" — always check here first before suggesting twilio commands.
---

# Twilio CLI

The Twilio CLI manages authentication, profiles, subaccounts, phone numbers, messaging services, and all Twilio resources from the terminal. Commands follow the pattern: `twilio <resource>:<action> [flags]`.

Use `twilio <command> --help` for detailed docs on any command.

## Installation

```bash
# macOS (recommended)
brew tap twilio/brew && brew install twilio

# Linux (apt)
wget -qO- https://twilio-cli-prod.s3.amazonaws.com/twilio_pub.asc | sudo apt-key add -
sudo touch /etc/apt/sources.list.d/twilio.list
echo 'deb https://twilio-cli-prod.s3.amazonaws.com/apt/ /' | sudo tee /etc/apt/sources.list.d/twilio.list
sudo apt update && sudo apt install -y twilio

# Windows (scoop)
scoop bucket add twilio-scoop https://github.com/twilio/scoop-twilio-cli
scoop install twilio

# npm (not recommended — no auto-updates)
npm install -g twilio-cli

# Verify
twilio version
```

Requires Node.js 20+. Don't mix installation methods (causes PATH conflicts).

## Profiles & Authentication

Profiles store credentials locally. Auth tokens are used once to create an API Key — they're never stored.

```bash
twilio login                          # Create profile (prompts for SID + Auth Token)
twilio profiles:list                  # List all profiles, shows active one
twilio profiles:use PROFILE_ID        # Switch active profile
twilio profiles:remove PROFILE_ID     # Delete a profile
twilio phone-numbers:list -p dev      # Use specific profile for one command
```

### Regional Profiles

```bash
twilio login --region ie1 --edge dublin    # Ireland
twilio login --region au1 --edge sydney    # Australia
twilio login --region jp1 --edge tokyo     # Japan
```

Both `--region` and `--edge` are required. Use region-specific Auth Tokens.

### Environment Variables (Alternative)

```bash
export TWILIO_ACCOUNT_SID=ACXXXXXXXX
export TWILIO_API_KEY=SKXXXXXXXX
export TWILIO_API_SECRET=XXXXXXXX
export TWILIO_REGION=us1          # optional
export TWILIO_EDGE=ashburn        # optional
```

### Credential Precedence

1. `-p PROFILE` flag (highest)
2. Environment variables
3. Active profile (lowest)

## Configuration

Global settings that apply across all profiles. Stored in `~/.twilio-cli/config.json`.

```bash
twilio config:list                              # Show current settings
twilio config:set --edge=sydney                 # Set edge location
twilio config:set --edge=                       # Remove a setting (prompts confirmation)
twilio config:set --require-profile-input       # Force -p flag on every command
twilio config:set --no-require-profile-input    # Revert above
```

Priority: Environment variables > config.json values.

## Subaccounts

Subaccounts separate usage by customer, team, or product. Managed through IAM API.

### Method 1: Set as Active Profile

```bash
twilio profiles:use SUBACCOUNT_PROFILE
twilio phone-numbers:list              # Uses subaccount credentials
```

### Method 2: Per-Command Flag

```bash
twilio phone-numbers:list -p SUBACCOUNT_PROFILE
```

### Core API Commands (V2010)

Core commands (`twilio api:core:*`) need a **Main** API Key, not Standard. Standard keys (created via `twilio login`) can't manage subaccounts.

```bash
# Set master credentials as env vars for core commands
export TWILIO_ACCOUNT_SID=ACXXXXXXXX
export TWILIO_API_KEY=SKXXXXXXXX
export TWILIO_API_SECRET=XXXXXXXX

# Then pass --account-sid for subaccount operations
twilio api:core:available-phone-numbers:local:list \
    --area-code="787" \
    --country-code US \
    --account-sid=SUBACCOUNT_SID

twilio api:core:messages:create \
    --from="+17875551234" \
    --to="+17875559876" \
    --body="Hello" \
    --account-sid=SUBACCOUNT_SID
```

## Phone Numbers

```bash
# Search available numbers in Puerto Rico
# IMPORTANT: Twilio treats PR as its own country code, NOT US + area code 787
twilio api:core:available-phone-numbers:local:list \
    --country-code PR \
    --voice-enabled \
    --sms-enabled

# Search US numbers with area code filter
twilio api:core:available-phone-numbers:local:list \
    --country-code US \
    --area-code 212 \
    --voice-enabled \
    --sms-enabled

# Search toll-free
twilio api:core:available-phone-numbers:toll-free:list \
    --country-code US

# Purchase a number
twilio api:core:incoming-phone-numbers:create \
    --phone-number="+17875551234"

# List owned numbers
twilio phone-numbers:list
twilio api:core:incoming-phone-numbers:list

# Update number configuration
twilio api:core:incoming-phone-numbers:update \
    --sid PNXXXXXXXX \
    --voice-url "https://voice.puny.bz/twilio/voice" \
    --sms-url "https://voice.puny.bz/twilio/sms"
```

## Messaging Services

```bash
# Create messaging service
twilio api:messaging:v1:services:create \
    --friendly-name "My Business SMS"

# Add phone number to service
twilio api:messaging:v1:services:phone-numbers:create \
    --service-sid MGXXXXXXXX \
    --phone-number-sid PNXXXXXXXX

# List services
twilio api:messaging:v1:services:list
```

## Calls & Messages

```bash
# Send SMS
twilio api:core:messages:create \
    --from="+17875551234" \
    --to="+17875559876" \
    --body="Hello from Twilio CLI"

# Make a call
twilio api:core:calls:create \
    --from="+17875551234" \
    --to="+17875559876" \
    --url="http://demo.twilio.com/docs/voice.xml"

# List recent calls/messages
twilio api:core:calls:list --limit 10
twilio api:core:messages:list --limit 10
```

## Accounts (Subaccounts)

```bash
# Create subaccount
twilio api:core:accounts:create \
    --friendly-name "Customer ABC"

# List all accounts
twilio api:core:accounts:list

# Get specific account
twilio api:core:accounts:fetch --sid ACXXXXXXXX

# Suspend/close subaccount
twilio api:core:accounts:update \
    --sid ACXXXXXXXX \
    --status suspended
```

## Output Formatting

```bash
twilio phone-numbers:list -o json     # JSON output
twilio phone-numbers:list -o csv      # CSV output
twilio phone-numbers:list -o tsv      # Tab-separated
twilio phone-numbers:list -o columns  # Default columnar
```

## Webhooks (Local Development)

The CLI can tunnel webhooks to localhost for development:

```bash
twilio phone-numbers:update PNXXXXXXXX \
    --sms-url http://localhost:8000/twilio/sms
```

For production, use proper HTTPS URLs.

## Logging & Debugging

```bash
twilio api:core:messages:create \
    --from="+17875551234" \
    --to="+17875559876" \
    --body="test" \
    -l debug                           # Enable debug logging
```

## Puny.bz Context

In this project, Twilio is used with **subaccounts per organization**:
- Master credentials in env vars (`TWILIO_ACCOUNT_SID`, `TWILIO_AUTH_TOKEN`)
- Subaccounts created via `profiles/services/twilio_provisioning.py`
- Phone numbers purchased per region (PR=787, US, CO, MX)
- Messaging Services created per org for SMS routing
- Voice webhooks point to `voice.puny.bz/twilio/voice`
- SMS webhooks point to `voice.puny.bz/twilio/sms`
- Production phones: `+17875581942` (local), `+18887944620` (toll-free)

For the provisioning service details, see `references/provisioning.md`.
