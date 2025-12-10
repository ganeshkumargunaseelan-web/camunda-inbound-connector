# Architecture Overview

## System Design

The connector sits between messaging platforms and Camunda 8. Webhooks receive incoming messages, the routing engine decides what to do with them, and Zeebe handles the process execution.

```
Messaging Channels          Connector                    Camunda 8
─────────────────          ─────────                    ─────────
WhatsApp ───┐
            │              ┌──────────────┐
Telegram ───┼─── POST ───> │ Webhook      │
            │              │ Receivers    │
SMS ────────┘              └──────┬───────┘
                                  │
                           ┌──────▼───────┐
                           │ Message      │
                           │ Parser       │
                           └──────┬───────┘
                                  │
                           ┌──────▼───────┐
                           │ Arabic       │
                           │ Processor    │
                           └──────┬───────┘
                                  │
                           ┌──────▼───────┐             ┌──────────────┐
                           │ Routing      │────────────>│ Start Process │
                           │ Engine       │             │ or Correlate  │
                           └──────────────┘             └──────────────┘
```

## Module Structure

The codebase follows hexagonal architecture. Business logic stays isolated from external dependencies.

```
inbound-messaging-connector/
│
├── inbound-connector-domain/
│   └── Core models: InboundMessage, Channel, Sender, Attachment
│
├── inbound-connector-application/
│   └── Business logic: routing rules, Arabic processing, use cases
│
├── inbound-connector-infrastructure/
│   └── External systems: webhook parsers, Zeebe client, storage adapters
│
├── inbound-connector-api/
│   └── REST controllers for each webhook endpoint
│
└── inbound-connector-starter/
    └── Spring Boot application entry point
```

## Message Flow

1. **Webhook receives POST request** from WhatsApp/Telegram/SMS provider
2. **Parser converts** provider-specific JSON to standard InboundMessage format
3. **Arabic processor** normalizes text (removes diacritics, converts numerals)
4. **Routing engine** evaluates rules by priority, first match wins
5. **Zeebe client** either starts new process or correlates to existing instance

## Routing Logic

Rules are evaluated top-down by priority. Each rule can match on:

- Keywords (Arabic or English)
- Regex patterns
- Channel type
- Locale (detected from phone prefix)

When a rule matches, the connector either starts a new process instance or sends a correlation message to an existing one.

## Key Components

| Component | Purpose |
|-----------|---------|
| InboundMessage | Domain entity holding message data |
| ProcessInboundMessageUseCase | Main processing entry point |
| MessageRoutingService | Rule evaluation logic |
| ArabicTextProcessor | Tashkeel removal, numeral conversion |
| ProcessStarter | Creates Zeebe process instances |
| MessageCorrelator | Correlates messages to running processes |

## Arabic Text Handling

The connector normalizes Arabic input before routing:

- Strips diacritical marks (harakat)
- Removes tatweel characters
- Standardizes hamza variants
- Converts Eastern Arabic numerals to Western

This ensures consistent keyword matching regardless of how the text was typed.

## GCC Locale Detection

Phone prefixes map to locales automatically:

```
+966 → ar-SA (Saudi Arabia)
+971 → ar-AE (UAE)
+965 → ar-KW (Kuwait)
+973 → ar-BH (Bahrain)
+974 → ar-QA (Qatar)
+968 → ar-OM (Oman)
```

Locale can be used in routing rules to direct messages to region-specific processes.

## Storage Options

Media attachments can be stored in:

- Local filesystem
- MinIO (S3-compatible)
- AWS S3

Configure the storage backend in application.yml.

## Tech Stack

| Component | Version |
|-----------|---------|
| Java | 21 LTS |
| Spring Boot | 3.3.5 |
| Camunda | 8.4.0 |
| PostgreSQL | 14+ |

---

G. Ganesh Kumar
Solution Architect
ganeshkumargunaseelan@gmail.com
+971-55-816-0396
