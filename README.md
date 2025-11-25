# Inbound Messaging Connector for Camunda 8

A middleware connector designed for GCC-region businesses to integrate WhatsApp, Telegram, and regional SMS providers with Camunda 8 workflows. Includes full Arabic language normalization and localization support.

---

## Why This Connector

Arabic messages differ from standard Latin-based inputs. This connector provides a reliable preprocessing layer to ensure correct interpretation and routing.

### Arabic Language Processing

- Diacritics removal (tashkeel)
- Tatweel normalization
- Hamza normalization (أ إ آ ء)
- Eastern Arabic number conversion (٠١٢٣٤٥٦٧٨٩ → 0123456789)
- Right-to-left text handling

### GCC Locale Detection

Automatic region identification from MSISDN prefixes.

| Prefix | Country | Locale |
|--------|---------|--------|
| +966 | Saudi Arabia | ar-SA |
| +971 | UAE | ar-AE |
| +965 | Kuwait | ar-KW |
| +973 | Bahrain | ar-BH |
| +974 | Qatar | ar-QA |
| +968 | Oman | ar-OM |

---

## Supported Messaging Channels

| Channel | Provider |
|---------|----------|
| WhatsApp | Meta Business API |
| Telegram | Bot API |
| SMS | Unifonic, Infobip, STC, Etisalat |

---

## Key Features

- Rule-based message routing
- Arabic and English keyword matching
- Process instance correlation
- Media download and storage (Local, MinIO, S3)
- Unified message normalization across all channels

### Routing Example

```yaml
rules:
  - name: arabic-support
    priority: 100
    processDefinitionKey: arabic-support-process
    locale: ar
    keywords:
      - مساعدة
      - دعم
      - شكوى
    processVariables:
      language: arabic
      region: gcc
```

---

## Download

The latest JAR is available in the [Releases](../../releases/latest) section.

---

## Quick Start

### Requirements

- Java 21 LTS
- PostgreSQL 14+
- Camunda 8 (self-hosted or cloud)

### Setup

1. Create deployment folder:

```bash
mkdir inbound-connector
cd inbound-connector
```

2. Copy the configuration template:

```bash
cp config/application-example.yml application.yml
```

3. Edit `application.yml`:

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

4. Run the connector:

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

- [Architecture Overview](docs/architecture-overview.md)
- [Configuration Guide](docs/configuration-guide.md)
- [API Reference](docs/api-reference.md)

---

## Version Information

| Component | Version |
|-----------|---------|
| Connector | 1.0.0 |
| Java | 21 LTS |
| Spring Boot | 3.3.5 |
| Camunda | 8.4.0 |

---

## Developed By

**G. Ganesh Kumar**
Solution Architect
UAE
ganeshkumargunaseelan@gmail.com
+971-55-816-0396
