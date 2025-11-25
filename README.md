Inbound Messaging Connector for Camunda 8

This connector is something I built to make it easier for businesses—especially in the GCC region—to connect their messaging channels directly with Camunda 8 workflows. It handles Arabic text properly, supports regional SMS providers, and gives a unified way to route incoming messages into BPMN processes.

What It Does

Receives messages from WhatsApp Business API, Telegram bots, and popular SMS gateways

Routes every message to the right Camunda 8 workflow based on rules you define

Handles Arabic language cases like RTL text, diacritics, and Eastern numerals

Downloads and stores media attachments (local folder, MinIO, or AWS S3)

Correlates follow-up messages to existing process instances automatically

## Supported Channels

| Channel  | Provider                         |
| -------- | -------------------------------- |
| WhatsApp | Meta Business API                |
| Telegram | Bot API                          |
| SMS      | Unifonic, Infobip, STC, Etisalat |






## Download

**[Download the latest JAR from Releases](../../releases/latest)**

## Getting Started

### You'll Need

- Java 21 (LTS release)
- PostgreSQL 14 or newer
- Running Camunda 8 instance (self-hosted or Cloud)

### Setup

1. Download the JAR from [Releases](../../releases/latest)

2. Create a folder and copy files:
```bash
mkdir inbound-connector && cd inbound-connector
# Copy the downloaded JAR here
cp config/application-example.yml application.yml
```

3. Open `application.yml` and fill in your details:

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

4. Start the connector:
```bash
java -jar inbound-messaging-connector-1.0.0.jar
```

## Webhook URLs

Configure these endpoints in your channel provider dashboards:

| Channel | URL | Notes |
|---------|-----|-------|
| WhatsApp | `/webhook/whatsapp` | GET for verification, POST for messages |
| Telegram | `/webhook/telegram` | POST only |
| Unifonic SMS | `/webhook/sms/unifonic` | POST only |
| Infobip SMS | `/webhook/sms/infobip` | POST only |
| STC SMS | `/webhook/sms/stc` | POST only |
| Etisalat SMS | `/webhook/sms/etisalat` | POST only |

## Monitoring

The connector exposes standard Spring Boot actuator endpoints:

- `GET /actuator/health` - Quick health check
- `GET /actuator/info` - Application info
- `GET /actuator/prometheus` - Metrics for Prometheus

## More Information

Check the `docs/` folder:
- `architecture-overview.md` - How the system is structured
- `configuration-guide.md` - All configuration options explained
- `api-reference.md` - Webhook formats and responses

## Version Info

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
