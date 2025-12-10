# Assets

This folder contains visual and template assets for the Inbound Messaging Connector.

## Files

| File | Description |
|------|-------------|
| `connector-logo.svg` | SVG logo for the connector |
| `connector-overview.png` | Architecture overview diagram (add manually) |
| `connector-template.json` | Camunda element template for BPMN modeler |

## Connector Template

The `connector-template.json` file can be imported into Camunda Modeler to use the Inbound Messaging Connector in your BPMN diagrams.

### Installation in Camunda Modeler

1. Open Camunda Modeler
2. Go to **File** > **Open Element Templates**
3. Navigate to this `assets` folder
4. Select `connector-template.json`
5. The template will now appear in the properties panel when you select a Start Event

### Template Features

- **Channel Configuration**: Select from WhatsApp, Telegram, or SMS providers
- **Message Routing**: Configure keywords and regex patterns
- **Arabic Processing**: Toggle normalization features
- **Correlation**: Set up message correlation for running instances
- **Output Mapping**: Map incoming message data to process variables

## Adding the Overview Image

To add the connector overview diagram:

1. Save your overview image as `connector-overview.png` in this folder
2. Recommended dimensions: 1200x800 pixels
3. The image is referenced in the main README.md
