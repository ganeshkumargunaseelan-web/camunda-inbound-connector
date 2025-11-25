# Architecture Overview

This document explains how the Inbound Messaging Connector is structured and how data flows through the system.

## High-Level Architecture

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                         MESSAGING CHANNELS                                   │
├─────────────────┬─────────────────┬─────────────────┬───────────────────────┤
│    WhatsApp     │    Telegram     │       SMS       │      Future           │
│   Business API  │    Bot API      │  (Multi-Provider)│    Channels          │
└────────┬────────┴────────┬────────┴────────┬────────┴───────────────────────┘
         │                 │                 │
         ▼                 ▼                 ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│                      INBOUND MESSAGING CONNECTOR                             │
├─────────────────────────────────────────────────────────────────────────────┤
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐        │
│  │  Webhook    │  │  Message    │  │   Arabic    │  │   Media     │        │
│  │  Receivers  │──│  Parsers    │──│  Processor  │──│  Downloader │        │
│  └─────────────┘  └─────────────┘  └─────────────┘  └─────────────┘        │
│         │                                                    │              │
│         ▼                                                    ▼              │
│  ┌─────────────────────────────────────────────────────────────────┐       │
│  │                    MESSAGE ROUTING ENGINE                        │       │
│  │  • Keyword Matching (Arabic/English)                            │       │
│  │  • Regex Pattern Matching                                        │       │
│  │  • Channel-based Routing                                         │       │
│  │  • Locale-based Routing                                          │       │
│  │  • Priority-based Rule Evaluation                                │       │
│  └─────────────────────────────────────────────────────────────────┘       │
│         │                                                                   │
│         ▼                                                                   │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐                         │
│  │  Process    │  │  Message    │  │  Session    │                         │
│  │  Starter    │  │  Correlator │  │  Manager    │                         │
│  └─────────────┘  └─────────────┘  └─────────────┘                         │
└────────┬────────────────┬────────────────┬──────────────────────────────────┘
         │                │                │
         ▼                ▼                ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│                          CAMUNDA 8 ZEEBE                                     │
│  ┌─────────────────┐  ┌─────────────────┐  ┌─────────────────┐              │
│  │ Start Process   │  │ Correlate Msg   │  │  BPMN Process   │              │
│  │ Instance        │  │ to Instance     │  │  Execution      │              │
│  └─────────────────┘  └─────────────────┘  └─────────────────┘              │
└─────────────────────────────────────────────────────────────────────────────┘
```

## Project Structure

I've organized the code using hexagonal architecture (ports & adapters). This keeps the business logic clean and makes it easy to swap out components.

```
inbound-messaging-connector/
├── inbound-connector-domain/          # Core models - no dependencies
│   ├── model/                         # InboundMessage, Channel, Sender, etc.
│   └── port/                          # Repository interfaces
│
├── inbound-connector-application/     # Business logic lives here
│   ├── usecase/                       # ProcessInboundMessageUseCase
│   ├── service/                       # MessageRoutingService, ArabicTextProcessor
│   └── model/                         # RoutingRule configuration
│
├── inbound-connector-infrastructure/  # All external integrations
│   ├── channel/                       # WhatsApp, Telegram, SMS parsers
│   ├── camunda/                       # Zeebe client wrappers
│   ├── storage/                       # Local, MinIO, S3 adapters
│   ├── media/                         # Media download handlers
│   └── persistence/                   # JPA repositories and entities
│
├── inbound-connector-api/             # REST endpoints
│   └── webhook/                       # Controllers for each channel
│
└── inbound-connector-starter/         # Spring Boot app entry point
    ├── config/                        # Bean configurations
    └── resources/                     # YAML config files
```

## How Messages Flow Through the System

### Step 1: Webhook Receives the Message

```
WhatsApp POST /webhook/whatsapp
    → WhatsAppWebhookController
    → WhatsAppMessageParser
    → InboundMessage (domain object)
```

Each channel has its own parser that transforms the provider-specific JSON into our standard `InboundMessage` format.

### Step 2: Processing and Routing

```
InboundMessage
    → ProcessInboundMessageUseCase
        ├── ArabicTextProcessor (detects language, removes diacritics)
        ├── MessageRepository (saves to database)
        └── MessageRoutingService (finds matching rule)
```

The routing engine evaluates rules in priority order. First match wins.

### Step 3: Camunda Integration

Depending on the routing rule configuration:

```
Rule says startNewProcess: true
    → ProcessStarter.startProcess(processKey, variables)

Rule says startNewProcess: false
    → MessageCorrelator.correlate(messageName, correlationKey, variables)
```

## Key Classes

### Domain Layer

| Class | What it does |
|-------|--------------|
| `InboundMessage` | Main entity - holds everything about a message |
| `Channel` | Enum for WHATSAPP, TELEGRAM, SMS_UNIFONIC, etc. |
| `MessageType` | TEXT, IMAGE, DOCUMENT, LOCATION, CONTACT |
| `Sender` | Customer details (phone, name, profile) |
| `Attachment` | Media files with storage location |

### Application Layer

| Class | What it does |
|-------|--------------|
| `ProcessInboundMessageUseCase` | Main entry point for processing |
| `MessageRoutingService` | Evaluates routing rules |
| `ArabicTextProcessor` | Arabic text normalization |
| `RoutingRule` | Configuration for a single routing rule |

### Infrastructure Layer

| Class | What it does |
|-------|--------------|
| `WhatsAppMessageParser` | Parses Meta's webhook format |
| `TelegramMessageParser` | Parses Telegram's update objects |
| `UnifonicSmsParser`, `StcSmsParser`, etc. | SMS provider parsers |
| `ProcessStarter` | Wraps Zeebe's create process instance |
| `MessageCorrelator` | Wraps Zeebe's message correlation |

## Tech Stack

| What | Version |
|------|---------|
| Java | 21 LTS |
| Spring Boot | 3.3.5 |
| Camunda | 8.4.0 |
| Database | PostgreSQL 14+ |
| Storage | Local / MinIO / S3 |
| Build | Maven 3.9+ |

## GCC-Specific Features

I built this with Middle East businesses in mind:

- **Arabic text handling**: Properly strips diacritics (tashkeel), normalizes tatweel and hamza variants
- **Phone-based locale detection**: +966 → Saudi Arabia, +971 → UAE, etc.
- **Right-to-left support**: Text processing preserves RTL directionality
- **Eastern Arabic numerals**: Converts ٠١٢٣٤٥٦٧٨٩ to 0123456789 for routing
- **Regional SMS providers**: Unifonic, Infobip, STC, Etisalat out of the box

---

**Developed by:**

G. Ganesh Kumar, Solution Architect
📞 UAE: +971-55 816 0396
📱 WhatsApp: +91-95000 03051
📧 Email: ganeshkumargunaseelan@gmail.com
