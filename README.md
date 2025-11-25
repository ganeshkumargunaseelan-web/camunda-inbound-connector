# Inbound Messaging Connector for Camunda 8

**Built for the GCC Region** | **Full Arabic Support** | **Regional SMS Providers**

A middleware connector designed specifically for Middle East businesses to connect WhatsApp, Telegram, and regional SMS providers to Camunda 8 workflows.

---

## Why This Connector?

Most messaging integrations don't handle Arabic properly. This one does.

### Arabic Language Processing
- **Diacritics Removal (Tashkeel)** - Strips حَرَكَات for consistent keyword matching
- **Tatweel Normalization** - Handles ـــ stretching characters
- **Hamza Standardization** - Normalizes أ إ آ ء variants
- **Eastern Arabic Numerals** - Converts ٠١٢٣٤٥٦٧٨٩ to 0123456789
- **RTL Text Support** - Proper right-to-left handling throughout

### GCC Locale Detection
Automatically detects customer country from phone number:
| Prefix | Country | Locale |
|--------|---------|--------|
| +966 | Saudi Arabia | ar-SA |
| +971 | UAE | ar-AE |
| +965 | Kuwait | ar-KW |
| +973 | Bahrain | ar-BH |
| +974 | Qatar | ar-QA |
| +968 | Oman | ar-OM |

### Regional SMS Providers
Native support for GCC SMS gateways:
- **Unifonic** - Saudi Arabia, UAE
- **STC Business** - Saudi Arabia
- **Etisalat (e&)** - UAE
- **Infobip** - Multi-region

---

## Supported Channels

| Channel  | Provider                         |
| -------- | -------------------------------- |
| WhatsApp | Meta Business API                |
| Telegram | Bot API                          |
| SMS      | Unifonic, Infobip, STC, Etisalat |

---

## Key Features

- **Rule-Based Routing** - Route messages by keywords (Arabic/English), regex, channel, or locale
- **Process Correlation** - Link follow-up messages to running Camunda instances
- **Media Handling** - Download and store attachments (Local, MinIO, S3)
- **Bilingual Keywords** - Match "help" and "مساعدة" in the same rule

### Routing Example
```yaml
rules:
  - name: arabic-support
    priority: 100
    processDefinitionKey: arabic-support-process
    locale: ar
    keywords:
      - مساعدة    # help
      - دعم       # support
      - شكوى      # complaint
    processVariables:
      language: arabic
      region: gcc
```

---

## Download

**[Download the latest JAR from Releases](../../releases/latest)**

---

## Quick Start

### Requirements
- Java 21 LTS
- PostgreSQL 14+
- Camunda 8 (self-hosted or Cloud)

### Setup

1. Download the JAR from [Releases](../../releases/latest)

2. Create deployment folder:
```bash
mkdir inbound-connector && cd inbound-connector
```

3. Copy and edit config:
```bash
cp config/application-example.yml application.yml
```

4. Configure your settings in `application.yml`:
```yaml
spring:
  datasource:
    url: jdbc:postgresql://localhost:5432/inbound_connector
    username: your_db_user
    password: your_db_password

zeebe:
  client:
    broker:
      gateway-address: localhost:26500

inbound-connector:
  channels:
    whatsapp:
      enabled: true
      access-token: your-meta-access-token
```

5. Run:
```bash
java -jar inbound-messaging-connector-1.0.0.jar
```

---

## Webhook Endpoints

| Channel | Endpoint | Method |
|---------|----------|--------|
| WhatsApp | `/webhook/whatsapp` | GET (verify), POST |
| Telegram | `/webhook/telegram` | POST |
| Unifonic | `/webhook/sms/unifonic` | POST |
| Infobip | `/webhook/sms/infobip` | POST |
| STC | `/webhook/sms/stc` | POST |
| Etisalat | `/webhook/sms/etisalat` | POST |

---

## Documentation

| Document | Description |
|----------|-------------|
| [Architecture Overview](docs/architecture-overview.md) | System design and data flow |
| [Configuration Guide](docs/configuration-guide.md) | All settings explained |
| [API Reference](docs/api-reference.md) | Webhook formats and responses |

---

## Version

| Component | Version |
|-----------|---------|
| Connector | 1.0.0 |
| Java | 21 LTS |
| Spring Boot | 3.3.5 |
| Camunda | 8.4.0 |

---

**Developed by:**

G. Ganesh Kumar, Solution Architect
📞 UAE: +971-55 816 0396
📱 WhatsApp: +91-95000 03051
📧 Email: ganeshkumargunaseelan@gmail.com
