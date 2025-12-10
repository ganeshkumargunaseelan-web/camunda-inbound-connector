<p align="center">
  <img src="assets/connector-logo.svg" alt="Inbound Messaging Connector Logo" width="120" height="120">
</p>

# Inbound Messaging Connector for Camunda 8

> Route WhatsApp, Telegram, and SMS messages into Camunda 8 workflows — with full Arabic language support for the GCC region.

[![Java](https://img.shields.io/badge/Java-21_LTS-orange)](https://openjdk.org/)
[![Spring Boot](https://img.shields.io/badge/Spring_Boot-3.3.5-green)](https://spring.io/projects/spring-boot)
[![Camunda](https://img.shields.io/badge/Camunda-8.4.0-blue)](https://camunda.com/)
[![License](https://img.shields.io/badge/License-MIT-yellow)](LICENSE)

---

## Overview

<p align="center">
  <img src="assets/Camunda Connector.png" alt="Connector Overview" width="800">
</p>

A middleware connector that receives inbound messages from multiple channels and routes them to Camunda 8 process instances. Built specifically for GCC businesses where Arabic text handling and regional SMS providers are essential.

```
┌──────────────┐     ┌──────────────┐     ┌──────────────┐
│   WhatsApp   │     │   Telegram   │     │     SMS      │
│  Business    │     │   Bot API    │     │  (Regional)  │
└──────┬───────┘     └──────┬───────┘     └──────┬───────┘
       │                    │                    │
       └────────────────────┼────────────────────┘
                            ▼
                 ┌─────────────────────┐
                 │  Inbound Connector  │
                 │  ─────────────────  │
                 │  • Parse Message    │
                 │  • Normalize Arabic │
                 │  • Detect Locale    │
                 │  • Route to Process │
                 └──────────┬──────────┘
                            ▼
                 ┌─────────────────────┐
                 │    Camunda 8        │
                 │    ─────────        │
                 │  Start Process  OR  │
                 │  Correlate Message  │
                 └─────────────────────┘
```

---

## Why This Connector?

| Challenge | Solution |
|-----------|----------|
| Arabic diacritics break keyword matching | Tashkeel removal before routing |
| Multiple hamza forms cause mismatches | Hamza normalization (أ إ آ → ا) |
| Eastern numerals not recognized | Convert ٠١٢٣ → 0123 automatically |
| No GCC SMS provider support | Unifonic, STC, Etisalat built-in |
| Manual locale detection | Auto-detect from phone prefix |

---

## Supported Channels

| Channel | Provider | Status |
|---------|----------|--------|
| WhatsApp | Meta Business API | ✅ Ready |
| Telegram | Bot API | ✅ Ready |
| SMS | Unifonic | ✅ Ready |
| SMS | Infobip | ✅ Ready |
| SMS | STC Business | ✅ Ready |
| SMS | Etisalat (e&) | ✅ Ready |

---

## Arabic Language Processing

The connector normalizes Arabic text before routing:

| Feature | Input | Output |
|---------|-------|--------|
| Diacritics Removal | مَرْحَبًا | مرحبا |
| Tatweel Removal | مــــرحبا | مرحبا |
| Hamza Normalization | أهلاً / إهلا | اهلا |
| Number Conversion | ٠١٢٣٤٥ | 012345 |

### GCC Locale Detection

Automatically identifies customer region from phone number:

```
+966 → Saudi Arabia (ar-SA)
+971 → UAE (ar-AE)
+965 → Kuwait (ar-KW)
+973 → Bahrain (ar-BH)
+974 → Qatar (ar-QA)
+968 → Oman (ar-OM)
```

---

## Quick Start

### Prerequisites

- Java 21 LTS
- PostgreSQL 14+
- Camunda 8 (Self-Managed or SaaS)

### Installation

**1. Download the JAR**

Get the latest release from [Releases](../../releases/latest)

**2. Configure**

Create `application.yml`:

```yaml
spring:
  datasource:
    url: jdbc:postgresql://localhost:5432/inbound_connector
    username: db_user
    password: db_password

zeebe:
  client:
    broker:
      gateway-address: localhost:26500

inbound-connector:
  channels:
    whatsapp:
      enabled: true
      verify-token: your-verify-token
      access-token: your-meta-access-token
```

**3. Run**

```bash
java -jar inbound-messaging-connector-1.0.0.jar
```

---

## Webhook Endpoints

Configure these URLs in your messaging provider dashboards:

| Provider | Endpoint | Method |
|----------|----------|--------|
| WhatsApp | `https://your-domain/webhook/whatsapp` | GET, POST |
| Telegram | `https://your-domain/webhook/telegram` | POST |
| Unifonic | `https://your-domain/webhook/sms/unifonic` | POST |
| Infobip | `https://your-domain/webhook/sms/infobip` | POST |
| STC | `https://your-domain/webhook/sms/stc` | POST |
| Etisalat | `https://your-domain/webhook/sms/etisalat` | POST |

---

## Routing Rules

Route messages to different processes based on keywords, patterns, or locale:

```yaml
routing:
  rules:
    - name: arabic-support
      priority: 100
      keywords:
        - مساعدة
        - دعم
        - شكوى
      processDefinitionKey: arabic-support-process
      processVariables:
        language: arabic
        team: gcc-support

    - name: order-status
      priority: 90
      pattern: "order|طلب|#[0-9]+"
      processDefinitionKey: order-tracking
      correlate: true
      correlationKey: senderPhone
```

---

## Message Correlation

Link follow-up messages to running process instances:

```
Customer: "I want to check my order"     → Starts: order-tracking process
Customer: "Order #12345"                 → Correlates to running instance
Customer: "Cancel it"                    → Correlates to same instance
```

Uses `senderPhone` as the default correlation key.

---

## Media Storage

Attachments (images, documents) can be stored in:

| Backend | Configuration |
|---------|---------------|
| Local | `storage.type: local` |
| MinIO | `storage.type: minio` |
| AWS S3 | `storage.type: s3` |

---

## Connector Template

Import the [connector template](assets/connector-template.json) into Camunda Modeler to use this connector in your BPMN diagrams.

```
assets/connector-template.json
```

The template provides a visual configuration interface for:
- Channel selection (WhatsApp, Telegram, SMS providers)
- Arabic text normalization options
- Routing keywords and patterns
- Message correlation settings
- Output variable mapping

---

## Documentation

| Document | Description |
|----------|-------------|
| [Architecture](docs/architecture-overview.md) | System design and data flow |
| [Configuration](docs/configuration-guide.md) | All settings explained |
| [API Reference](docs/api-reference.md) | Webhook formats and responses |

---

## Tech Stack

| Component | Version |
|-----------|---------|
| Java | 21 LTS |
| Spring Boot | 3.3.5 |
| Camunda | 8.4.0 |
| PostgreSQL | 14+ |

---

## Use Cases

- **Customer Support** — Route Arabic/English queries to appropriate teams
- **Order Tracking** — Let customers check status via WhatsApp
- **Appointment Booking** — Conversational scheduling flows
- **Claims Intake (FNOL)** — Capture incident details via messaging
- **Government Services** — Citizen service requests in Arabic

---

## Author

**G. Ganesh Kumar**
Solution Architect | UAE

ganeshkumargunaseelan@gmail.com
+971-55-816-0396
