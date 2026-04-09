# Meta Graph API Quick Reference

## Base URL

All Graph API requests go to: `https://graph.facebook.com/v19.0/`

## Authentication

Every request needs an `access_token` parameter (query string or header).

```bash
# Query string
curl "https://graph.facebook.com/v19.0/me?access_token=TOKEN"

# Header
curl -H "Authorization: Bearer TOKEN" "https://graph.facebook.com/v19.0/me"
```

## Common Endpoints

### User

```
GET /me                          # Current user profile
GET /me/accounts                 # Pages the user manages
GET /me/permissions              # Granted permissions
```

### Facebook Pages

```
GET  /{page-id}                  # Page info
GET  /{page-id}/feed             # Page posts
POST /{page-id}/feed             # Create post (message=)
GET  /{page-id}/photos           # Page photos
POST /{page-id}/photos           # Upload photo (url=, caption=)
GET  /{page-id}/videos           # Page videos
POST /{page-id}/messages         # Send Messenger message
GET  /{page-id}/conversations    # Messenger conversations
```

### Posts & Comments

```
GET  /{post-id}                  # Post details
GET  /{post-id}/comments         # Post comments
POST /{post-id}/comments         # Reply to post (message=)
GET  /{comment-id}/comments      # Comment replies
POST /{comment-id}/comments      # Reply to comment
DELETE /{post-id}                # Delete post
DELETE /{comment-id}             # Delete comment
```

### Instagram

```
GET  /{ig-user-id}               # IG account info
GET  /{ig-user-id}/media         # IG posts
POST /{ig-user-id}/media         # Create media container
POST /{ig-user-id}/media_publish # Publish container
GET  /{ig-media-id}/comments     # Media comments
POST /{ig-media-id}/comments     # Reply to comment
GET  /{ig-user-id}/stories       # Stories
GET  /{ig-user-id}/insights      # Account insights
```

### Instagram Content Publishing

**Image:**
```
POST /{ig-user-id}/media
  image_url=URL
  caption=TEXT
  access_token=TOKEN
→ { "id": "CONTAINER_ID" }

POST /{ig-user-id}/media_publish
  creation_id=CONTAINER_ID
  access_token=TOKEN
```

**Video/Reel:**
```
POST /{ig-user-id}/media
  media_type=REELS
  video_url=URL
  caption=TEXT
→ { "id": "CONTAINER_ID" }

# Check status (video processing is async)
GET /{CONTAINER_ID}?fields=status_code
# Wait until status_code = FINISHED

POST /{ig-user-id}/media_publish
  creation_id=CONTAINER_ID
```

**Carousel:**
```
# Create each child
POST /{ig-user-id}/media
  image_url=URL1
  is_carousel_item=true
→ child_1_id

POST /{ig-user-id}/media
  image_url=URL2
  is_carousel_item=true
→ child_2_id

# Create carousel
POST /{ig-user-id}/media
  media_type=CAROUSEL
  children=child_1_id,child_2_id
  caption=TEXT

# Publish
POST /{ig-user-id}/media_publish
  creation_id=CAROUSEL_CONTAINER_ID
```

## Webhooks

### Verification (GET to your callback URL)

```
hub.mode=subscribe
hub.verify_token=YOUR_TOKEN
hub.challenge=CHALLENGE_STRING
```

Return the challenge string with 200 OK.

### Event Payload (POST to your callback URL)

```json
{
  "object": "page",
  "entry": [
    {
      "id": "PAGE_ID",
      "time": 1234567890,
      "messaging": [
        {
          "sender": {"id": "USER_PSID"},
          "recipient": {"id": "PAGE_ID"},
          "timestamp": 1234567890,
          "message": {
            "mid": "MESSAGE_ID",
            "text": "Hello!"
          }
        }
      ],
      "changes": [
        {
          "field": "feed",
          "value": {
            "item": "comment",
            "comment_id": "COMMENT_ID",
            "post_id": "POST_ID",
            "message": "Great post!",
            "from": {"id": "USER_ID", "name": "User Name"}
          }
        }
      ]
    }
  ]
}
```

## Rate Limits

| Type | Limit |
|------|-------|
| Application | 200 calls/user/hour |
| Page | 4800 calls/24 hours (at 100% app usage) |
| Instagram Content Publishing | 25 posts/24 hours |
| Instagram API | 200 calls/user/hour |

## Error Codes

| Code | Meaning |
|------|---------|
| 1 | Unknown error |
| 2 | Service unavailable |
| 4 | Too many calls (rate limited) |
| 10 | Permission denied |
| 100 | Invalid parameter |
| 190 | Invalid/expired access token |
| 200 | Permission not granted |
| 506 | Duplicate post |

## Permissions Reference

| Permission | Access Level | Purpose |
|------------|-------------|---------|
| pages_show_list | Standard | List user's pages |
| pages_read_engagement | Standard | Read page content |
| pages_manage_posts | Standard | Create/edit posts |
| pages_manage_engagement | Standard | Manage comments |
| pages_manage_metadata | Advanced | Page settings |
| pages_messaging | Advanced | Messenger messages |
| instagram_basic | Standard | Read IG profile/media |
| instagram_content_publish | Standard | Publish IG content |
| instagram_manage_comments | Standard | Manage IG comments |
| instagram_manage_messages | Advanced | IG DMs |
| business_management | Advanced | Business Manager access |

**Standard** = can use in development, needs app review for production
**Advanced** = needs app review + business verification
