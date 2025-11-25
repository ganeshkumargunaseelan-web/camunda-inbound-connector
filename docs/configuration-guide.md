# Configuration Guide

This guide covers all configuration options for the Inbound Messaging Connector.

## Where to Put Your Configuration

Spring Boot loads configuration in this order (later ones override earlier):

1. Defaults baked into the JAR
2. `application.yml` in the same folder as the JAR
3. Environment variables (e.g., `SPRING_DATASOURCE_PASSWORD=secret`)
4. Command line args (e.g., `--server.port=9090`)

For most deployments, just create an `application.yml` file next to your JAR.

## Database Configuration

### PostgreSQL (Required)

```yaml
spring:
  datasource:
    url: jdbc:postgresql://localhost:5432/inbound_connector
    username: postgres
    password: your_password
    driver-class-name: org.postgresql.Driver

    # Connection Pool (HikariCP)
    hikari:
      maximum-pool-size: 10
      minimum-idle: 5
      connection-timeout: 30000
      idle-timeout: 600000
      max-lifetime: 1800000

  jpa:
    hibernate:
      ddl-auto: update    # auto-create tables
    show-sql: false       # set true for debugging
```

### Environment Variables

```bash
export SPRING_DATASOURCE_URL=jdbc:postgresql://host:5432/db
export SPRING_DATASOURCE_USERNAME=user
export SPRING_DATASOURCE_PASSWORD=secret
```

## Camunda 8 Configuration

### Self-Hosted Zeebe

```yaml
zeebe:
  client:
    broker:
      gateway-address: localhost:26500
    security:
      plaintext: true     # false for TLS
```

### Camunda Cloud (SaaS)

```yaml
zeebe:
  client:
    cloud:
      region: bru-2
      cluster-id: your-cluster-id
      client-id: your-client-id
      client-secret: your-client-secret
```

## Channel Configuration

### WhatsApp Business API

```yaml
inbound-connector:
  channels:
    whatsapp:
      enabled: true
      verify-token: your-webhook-verify-token
      access-token: your-meta-access-token
      phone-number-id: "123456789"
      business-account-id: "987654321"
      api-version: v18.0
```

**Meta Business Suite Setup:**
1. Go to Meta Business Suite → WhatsApp → API Setup
2. Copy Access Token and Phone Number ID
3. Configure Webhook URL: `https://your-domain.com/webhook/whatsapp`
4. Set Verify Token (must match config)
5. Subscribe to: messages, message_echoes

### Telegram Bot

```yaml
inbound-connector:
  channels:
    telegram:
      enabled: true
      bot-token: "123456:ABC-DEF1234ghIkl-zyx57W2v1u123ew11"
      webhook-secret: optional-secret
```

**Telegram Setup:**
1. Message @BotFather to create bot
2. Copy the bot token
3. Set webhook:
   ```
   curl https://api.telegram.org/bot<TOKEN>/setWebhook?url=https://your-domain.com/webhook/telegram
   ```

### SMS Providers

#### Unifonic

```yaml
inbound-connector:
  channels:
    sms:
      enabled: true
      provider: unifonic
      unifonic:
        app-sid: your-app-sid
        sender-id: your-sender-id
```

#### Infobip

```yaml
inbound-connector:
  channels:
    sms:
      enabled: true
      provider: infobip
      infobip:
        api-key: your-api-key
        base-url: https://api.infobip.com
```

#### STC / Etisalat

```yaml
inbound-connector:
  channels:
    sms:
      enabled: true
      provider: stc  # or etisalat
      stc:
        api-key: your-api-key
        sender-id: your-sender-id
```

## Storage Configuration

### Local Storage (Development)

```yaml
storage:
  type: local
  local:
    path: ./uploads
```

### MinIO (Self-Hosted S3)

```yaml
storage:
  type: minio
  minio:
    endpoint: http://localhost:9000
    access-key: minioadmin
    secret-key: minioadmin
    bucket: inbound-connector
```

### AWS S3 (Production)

```yaml
storage:
  type: s3
  s3:
    region: me-south-1        # Bahrain for GCC
    bucket: your-bucket-name
    access-key: AKIA...
    secret-key: ...
```

## Routing Rules Configuration

```yaml
rules:
  - name: rule-name
    description: Rule description
    priority: 100             # Higher = evaluated first
    enabled: true
    processDefinitionKey: your-bpmn-process-id

    # Matching criteria (all optional, empty = match all)
    keywords:
      - help
      - مساعدة
    regexPatterns:
      - "ORDER-\\d+"
    channels:
      - WHATSAPP
      - TELEGRAM
    messageTypes:
      - TEXT
      - IMAGE
    locale: ar                # ar, en, ar-SA, ar-AE, etc.
    senderPhonePrefix: "+971"

    # Actions
    startNewProcess: true     # false = correlate to existing
    correlationKey: "${sender.phone}"
    processVariables:
      custom_var: value
```

## Security Configuration

```yaml
inbound-connector:
  security:
    csrf-enabled: false       # Keep false for webhooks
    actuator:
      username: admin
      password: secure-password-here
```

## Logging Configuration

```yaml
logging:
  level:
    root: INFO
    io.camunda.connector.inbound: DEBUG
    org.springframework.web: INFO
    org.hibernate.SQL: DEBUG  # SQL logging

  file:
    name: ./logs/inbound-connector.log
    max-size: 100MB
    max-history: 30

  pattern:
    file: "%d{yyyy-MM-dd HH:mm:ss} [%thread] %-5level %logger{36} - %msg%n"
```

## Monitoring Configuration

```yaml
management:
  endpoints:
    web:
      exposure:
        include: health,info,metrics,prometheus
  endpoint:
    health:
      show-details: when-authorized
```

## Complete Example

See `config/application-example.yml` for a complete configuration template with all options.

---

**Developed by:**

G. Ganesh Kumar, Solution Architect
📞 UAE: +971-55 816 0396
📱 WhatsApp: +91-95000 03051
📧 Email: ganeshkumargunaseelan@gmail.com
