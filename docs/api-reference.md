# API Reference

All the webhook endpoints and their expected formats.

## Webhook Endpoints

### WhatsApp

WhatsApp uses two HTTP methods on the same endpoint.

**Verification (GET)** - Meta calls this when you first set up the webhook:

```http
GET /webhook/whatsapp?hub.mode=subscribe&hub.verify_token=YOUR_TOKEN&hub.challenge=CHALLENGE
```

Returns the challenge value if your verify token matches. Otherwise returns 403.

**Message Reception (POST)** - Called for every message:

```http
POST /webhook/whatsapp
Content-Type: application/json

{
  "object": "whatsapp_business_account",
  "entry": [...]
}
```

Response:
```json
{
  "status": "received",
  "messageId": "550e8400-e29b-41d4-a716-446655440000",
  "processed": true
}
```

---

### Telegram

```http
POST /webhook/telegram
Content-Type: application/json
X-Telegram-Bot-Api-Secret-Token: optional-secret

{
  "update_id": 123456789,
  "message": {...}
}
```

Response:
```json
{
  "status": "received",
  "messageId": "550e8400-e29b-41d4-a716-446655440000",
  "processed": true
}
```

The `X-Telegram-Bot-Api-Secret-Token` header is optional but recommended. Configure it in your `application.yml` under `telegram.webhook-secret`.

---

### SMS Webhooks

#### Unifonic

```http
POST /webhook/sms/unifonic
Content-Type: application/json

{
  "MessageID": "...",
  "SenderID": "+971...",
  "Body": "message text"
}
```

#### Infobip

```http
POST /webhook/sms/infobip
Content-Type: application/json

{
  "results": [...]
}
```

#### STC

```http
POST /webhook/sms/stc
Content-Type: application/json

{
  "message_id": "...",
  "from": "+966...",
  "text": "message text"
}
```

#### Etisalat

```http
POST /webhook/sms/etisalat
Content-Type: application/json

{
  "msgId": "...",
  "sender": "+971...",
  "message": "message text"
}
```

#### Generic SMS

```http
POST /webhook/sms
Content-Type: application/json
```

Routes to configured provider automatically.

---

## Health & Monitoring Endpoints

### Health Check

```http
GET /actuator/health
```

**Response:**
```json
{
  "status": "UP",
  "components": {
    "db": {"status": "UP"},
    "diskSpace": {"status": "UP"}
  }
}
```

### Application Info

```http
GET /actuator/info
```

### Prometheus Metrics

```http
GET /actuator/prometheus
```

### Protected Endpoints (Require Basic Auth)

```http
GET /actuator/metrics
GET /actuator/env
GET /actuator/loggers
GET /actuator/beans
GET /actuator/threaddump
```

---

## Channel-Specific Health Endpoints

### WhatsApp Health

```http
GET /webhook/whatsapp/health
```

**Response:**
```json
{
  "status": "healthy",
  "service": "WhatsApp Webhook Receiver",
  "channel": "WHATSAPP"
}
```

### Telegram Health

```http
GET /webhook/telegram/health
```

### SMS Health

```http
GET /webhook/sms/health
```

**Response:**
```json
{
  "status": "healthy",
  "service": "SMS Webhook Receiver",
  "provider": "unifonic",
  "supportedProviders": ["unifonic", "infobip", "mobily", "stc", "etisalat"]
}
```

---

## Webhook Setup Info Endpoints

### SMS Setup Info

```http
GET /webhook/sms/info
```

**Response:**
```json
{
  "endpoints": {
    "unifonic": "/webhook/sms/unifonic",
    "infobip": "/webhook/sms/infobip",
    "stc": "/webhook/sms/stc",
    "etisalat": "/webhook/sms/etisalat"
  },
  "currentProvider": "unifonic",
  "setup": {
    "unifonic": {
      "step1": "Login to Unifonic dashboard",
      "step2": "Go to Settings - Webhooks",
      ...
    }
  }
}
```

### Telegram Setup Info

```http
GET /webhook/telegram/info
```

---

## Process Variables Injected

When a message is routed to a Camunda process, these variables are automatically available:

| Variable | Type | Description |
|----------|------|-------------|
| `messageId` | String | Internal message UUID |
| `externalMessageId` | String | Original message ID from channel |
| `channel` | String | WHATSAPP, TELEGRAM, SMS_* |
| `messageText` | String | Message text content |
| `senderId` | String | Sender identifier |
| `senderPhone` | String | Sender phone number |
| `senderName` | String | Sender display name |
| `isArabic` | Boolean | True if Arabic text detected |
| `locale` | String | Detected locale (ar-SA, ar-AE, etc.) |
| `sentAt` | String | Message sent timestamp |
| `receivedAt` | String | Message received timestamp |
| `hasAttachments` | Boolean | True if message has attachments |
| `attachmentCount` | Integer | Number of attachments |
| `attachmentUrl` | String | First attachment URL (if any) |

### Custom Variables from Routing Rules

Variables defined in `processVariables` are also injected:

```yaml
rules:
  - name: example
    processVariables:
      priority: high
      department: support
```

These become available as `priority` and `department` in the process.

---

## Error Responses

### 400 Bad Request

```json
{
  "error": "Invalid payload format",
  "provider": "WhatsApp"
}
```

### 500 Internal Server Error

```json
{
  "error": "Processing failed",
  "message": "Database connection error"
}
```

---

## Testing Webhooks

### Using cURL

```bash
# WhatsApp
curl -X POST http://localhost:8080/webhook/whatsapp \
  -H "Content-Type: application/json" \
  -d @examples/whatsapp-webhook-sample.json

# Telegram
curl -X POST http://localhost:8080/webhook/telegram \
  -H "Content-Type: application/json" \
  -d @examples/telegram-webhook-sample.json

# SMS
curl -X POST http://localhost:8080/webhook/sms/unifonic \
  -H "Content-Type: application/json" \
  -d @examples/sms-unifonic-sample.json
```

### Quick Health Check

```bash
curl http://localhost:8080/actuator/health
```

---

**Developed by:**

G. Ganesh Kumar, Solution Architect
📞 UAE: +971-55 816 0396
📱 WhatsApp: +91-95000 03051
📧 Email: ganeshkumargunaseelan@gmail.com
