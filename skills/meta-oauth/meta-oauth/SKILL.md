---
name: meta-oauth
description: Use when working with Meta (Facebook + Instagram) integration — OAuth flows, Graph API, Facebook Pages, Instagram Business accounts, posting, comments, messaging (Messenger/Instagram DMs), webhooks, app review, business verification, or any Meta/Facebook/Instagram API work. Trigger on any mention of Facebook API, Instagram API, Graph API, Meta OAuth, Facebook Login, page access token, instagram_content_publish, pages_messaging, Meta Business Manager, Facebook webhook, or "how do I connect Facebook/Instagram". Use this skill even for simple questions about Meta permissions or token types.
---

# Meta (Facebook + Instagram) Integration

Meta has **no official CLI**. Everything is done via the Graph API (REST over HTTPS), web-based developer tools, and OAuth 2.0. This skill covers the full integration flow for connecting Facebook Pages and Instagram Business accounts.

## Web-Based Tools

| Tool | URL | Purpose |
|------|-----|---------|
| Graph API Explorer | developers.facebook.com/tools/explorer/ | Generate tokens, test endpoints |
| Access Token Debugger | developers.facebook.com/tools/debug/accesstoken/ | Inspect token validity, scopes, expiry |
| Webhooks Tool | developers.facebook.com/tools/webhooks/ | Test webhook subscriptions |
| App Dashboard | developers.facebook.com/apps/ | Manage apps, permissions, review |

## 1. Create a Meta App

Go to **developers.facebook.com** > Create App > Choose **Business** type.

You get:
- **App ID** — public identifier
- **App Secret** — keep secret, never expose to frontend

Configure:
- Valid OAuth Redirect URIs (e.g., `https://api.puny.bz/v1/meta/callback/`)
- App Domains (e.g., `puny.bz`, `api.puny.bz`)

## 2. OAuth 2.0 Flow (Facebook Login)

### Step 1: Redirect User to Facebook

```
https://www.facebook.com/v19.0/dialog/oauth?
  client_id={APP_ID}
  &redirect_uri={REDIRECT_URI}
  &scope={PERMISSIONS}
  &state={CSRF_TOKEN}
  &response_type=code
```

### Step 2: Exchange Code for Token

```bash
curl -X GET "https://graph.facebook.com/v19.0/oauth/access_token?\
  client_id={APP_ID}&\
  client_secret={APP_SECRET}&\
  redirect_uri={REDIRECT_URI}&\
  code={CODE}"
```

Returns: `{ "access_token": "...", "token_type": "bearer", "expires_in": 5183944 }`

### Step 3: Get Long-Lived Token

Short-lived tokens expire in ~1-2 hours. Exchange for long-lived (~60 days):

```bash
curl -X GET "https://graph.facebook.com/v19.0/oauth/access_token?\
  grant_type=fb_exchange_token&\
  client_id={APP_ID}&\
  client_secret={APP_SECRET}&\
  fb_exchange_token={SHORT_LIVED_TOKEN}"
```

### Step 4: Get Page Access Token

This is the most important token — most API calls use Page tokens, not user tokens.

```bash
curl -X GET "https://graph.facebook.com/v19.0/me/accounts?\
  access_token={LONG_LIVED_USER_TOKEN}"
```

Returns list of Pages the user manages, each with its own `access_token`. Page tokens derived from long-lived user tokens **never expire**.

### Step 5: Get Instagram Business Account

```bash
curl -X GET "https://graph.facebook.com/v19.0/{PAGE_ID}?\
  fields=instagram_business_account&\
  access_token={PAGE_ACCESS_TOKEN}"
```

Returns `{ "instagram_business_account": { "id": "17841400..." } }`

## 3. Token Types

| Token | Lifetime | Use |
|-------|----------|-----|
| Short-lived user | ~1-2 hours | Initial OAuth exchange |
| Long-lived user | ~60 days | Exchange for page tokens |
| Page access token | Never expires* | All Page/Instagram API calls |
| Instagram user token | Derived from page | Instagram content API |

*When derived from a long-lived user token.

**Store in DB**: Page access token + Instagram business account ID. Refresh the long-lived user token before 60-day expiry.

## 4. Permissions by Feature

### A. Posting (Facebook + Instagram)

**Facebook Pages:**
- `pages_manage_posts` — create/edit/delete posts
- `pages_read_engagement` — read likes, comments, shares
- `pages_show_list` — list pages the user manages

**Instagram:**
- `instagram_basic` — read profile, media
- `instagram_content_publish` — create posts, reels, stories

**Requirements:**
- Instagram must be a **Business** or **Creator** account (not personal)
- Must be connected to a **Facebook Page**

### B. Comments

- `pages_read_engagement` — read comments
- `pages_manage_engagement` — reply to/delete comments
- `instagram_manage_comments` — read/reply/hide Instagram comments

### C. Messaging (Advanced Access)

**Facebook Messenger:**
- `pages_messaging` — send/receive messages
- `pages_manage_metadata` — manage page settings

**Instagram DMs:**
- `instagram_manage_messages` — send/receive Instagram DMs

**Requirements (strict):**
- App Review approval
- Business Verification via Meta Business Manager
- Clear use-case justification with screencasts
- Messaging window: can only respond within 24h of user's last message (except with message tags)

## 5. Graph API Common Endpoints

### Facebook Page Posts

```bash
# Create post
curl -X POST "https://graph.facebook.com/v19.0/{PAGE_ID}/feed" \
  -d "message=Hello from Puny.bz!" \
  -d "access_token={PAGE_ACCESS_TOKEN}"

# Create post with image
curl -X POST "https://graph.facebook.com/v19.0/{PAGE_ID}/photos" \
  -d "url=https://example.com/photo.jpg" \
  -d "caption=Check this out!" \
  -d "access_token={PAGE_ACCESS_TOKEN}"

# Get page posts
curl -X GET "https://graph.facebook.com/v19.0/{PAGE_ID}/feed?\
  access_token={PAGE_ACCESS_TOKEN}"

# Get post comments
curl -X GET "https://graph.facebook.com/v19.0/{POST_ID}/comments?\
  access_token={PAGE_ACCESS_TOKEN}"

# Reply to comment
curl -X POST "https://graph.facebook.com/v19.0/{COMMENT_ID}/comments" \
  -d "message=Thanks for your feedback!" \
  -d "access_token={PAGE_ACCESS_TOKEN}"
```

### Instagram Content Publishing

Two-step process: create container, then publish.

```bash
# Step 1: Create media container (image)
curl -X POST "https://graph.facebook.com/v19.0/{IG_USER_ID}/media" \
  -d "image_url=https://example.com/photo.jpg" \
  -d "caption=Posted via Puny.bz #business" \
  -d "access_token={PAGE_ACCESS_TOKEN}"
# Returns: { "id": "container_id" }

# Step 2: Publish
curl -X POST "https://graph.facebook.com/v19.0/{IG_USER_ID}/media_publish" \
  -d "creation_id={CONTAINER_ID}" \
  -d "access_token={PAGE_ACCESS_TOKEN}"

# Carousel (multiple images)
# Step 1a: Create each image container with is_carousel_item=true
# Step 1b: Create carousel container referencing children
curl -X POST "https://graph.facebook.com/v19.0/{IG_USER_ID}/media" \
  -d "media_type=CAROUSEL" \
  -d "children={CONTAINER_1},{CONTAINER_2}" \
  -d "caption=Multiple photos!" \
  -d "access_token={PAGE_ACCESS_TOKEN}"
# Step 2: Publish as usual
```

### Messaging (Messenger)

```bash
# Send message (within 24h window)
curl -X POST "https://graph.facebook.com/v19.0/{PAGE_ID}/messages" \
  -H "Content-Type: application/json" \
  -d '{
    "recipient": {"id": "{USER_PSID}"},
    "message": {"text": "Hello! How can I help you?"},
    "access_token": "{PAGE_ACCESS_TOKEN}"
  }'
```

## 6. Webhooks

Subscribe to real-time events instead of polling.

### Setup

1. In App Dashboard > Webhooks > Subscribe to Page events
2. Provide callback URL: `https://api.puny.bz/v1/meta/webhook/`
3. Provide verify token (your custom string)

### Verification (GET)

Meta sends a GET request to verify your endpoint:

```python
# Django view
def webhook_verify(request):
    mode = request.GET.get('hub.mode')
    token = request.GET.get('hub.verify_token')
    challenge = request.GET.get('hub.challenge')
    
    if mode == 'subscribe' and token == settings.META_VERIFY_TOKEN:
        return HttpResponse(challenge, status=200)
    return HttpResponse('Forbidden', status=403)
```

### Event Handling (POST)

```python
def webhook_event(request):
    data = json.loads(request.body)
    
    for entry in data.get('entry', []):
        # Messaging events
        for msg_event in entry.get('messaging', []):
            sender_id = msg_event['sender']['id']
            message = msg_event.get('message', {})
            text = message.get('text', '')
            # Handle message...
        
        # Comment events
        for change in entry.get('changes', []):
            if change['field'] == 'feed':
                value = change['value']
                # Handle comment/post...
    
    return HttpResponse('OK', status=200)
```

### Subscribe to Fields

| Object | Fields | Events |
|--------|--------|--------|
| Page | `feed` | New posts, comments, reactions |
| Page | `messages` | New Messenger messages |
| Page | `messaging_postbacks` | Button clicks in Messenger |
| Instagram | `comments` | New comments on posts |
| Instagram | `messages` | New Instagram DMs |

## 7. Development vs Production

### Development Mode
- Only app admins, developers, and testers can use
- Good for building and testing
- No app review needed

### Live Mode (Production)
You must:
1. Submit for **App Review** — per permission
2. Provide **screencasts** showing your app using each permission
3. Write clear **justification** for each permission
4. Complete **Business Verification** (for messaging)

### App Review Tips
- Record a video showing your actual UI using each permission
- Explain the user benefit clearly
- Show opt-in/opt-out flows for messaging
- Expect 1-5 business days for review
- Messaging permissions get extra scrutiny

## 8. Business Verification

Required for: messaging APIs, advanced permissions.

Done via **Meta Business Manager** (business.facebook.com):
1. Verify business identity (legal name, address, phone)
2. Upload documents (business license, utility bill, etc.)
3. Meta may call your business phone to verify
4. Takes 1-5 business days

## 9. Common Gotchas

- Instagram API does **NOT** support personal accounts — Business or Creator only
- Messaging requires strict approval — no shortcuts
- Tokens expire — long-lived user tokens last ~60 days, must refresh
- Rate limits: 200 calls/user/hour for most endpoints
- Webhooks are **required** for real-time messaging (no polling)
- Page tokens from long-lived user tokens don't expire, but if user revokes app access, they're invalidated
- Instagram must be linked to a Facebook Page
- Video publishing is async — check status before publishing

## 10. Recommended MVP Scope

**Phase 1 (Start here):**
1. Connect Facebook Page (OAuth)
2. Connect Instagram Business Account
3. Read/publish posts
4. Read/reply to comments

**Phase 2 (After App Review):**
- Facebook Messenger integration
- Instagram DM integration
- Webhook-driven real-time responses

## Puny.bz Context

### Existing Pattern to Follow

The `GoogleIntegration` model in `puny_api/models.py` provides the reference pattern:

```python
class GoogleIntegration(models.Model):
    INTEGRATION_TYPES = [('calendar', 'Google Calendar'), ('gbp', 'Google Business Profile')]
    org = ForeignKey(Organization)
    integration_type = CharField(choices=INTEGRATION_TYPES)
    refresh_token = TextField()
    access_token = TextField()
    token_expiry = DateTimeField()
    connected_email = EmailField()
    is_active = BooleanField()
```

A similar `MetaIntegration` model would store:
- `page_access_token` — never-expiring page token
- `page_id` — Facebook Page ID
- `page_name` — for display
- `instagram_business_account_id` — linked IG account
- `user_access_token` — long-lived user token (for refresh)
- `token_expiry` — when user token expires
- `permissions` — JSONField of granted scopes

### Django Implementation

Use `social-auth-app-django` (already in requirements.txt) or custom OAuth views following the Google pattern in `puny_api/views/google_integration_api.py`.

### Key Environment Variables Needed

```
META_APP_ID=your_app_id
META_APP_SECRET=your_app_secret
META_VERIFY_TOKEN=your_webhook_verify_token
```

For detailed Graph API reference, see `references/graph-api.md`.
