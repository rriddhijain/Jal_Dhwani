# Technology Stack

**Competition Track**: AI for Communities, Access & Public Impact

## Architecture Overview

Three-tier system optimized for community fairness and public impact: Edge → Cloud → Frontend

## Edge Layer (Hardware)

- Microcontroller: ESP32-S3 (Wi-Fi + DSP capabilities)
- Sensor: Piezoelectric Disc (₹50 vibration sensor)
- Actuator: 5V Relay Module (pump power control)
- Language: C++ firmware
- Processing: On-device FFT (Fast Fourier Transform) for resource depletion detection
- Offline AI: Local detection ensures safety without internet
- Protocol: MQTT for cloud communication

## Cloud Layer (AWS)

- Ingestion: AWS IoT Core (MQTT broker)
- Compute: AWS Lambda (serverless functions)
- Database: Amazon DynamoDB (acoustic telemetry logs + community usage data)
- AI Brain: Amazon Bedrock (Claude 3.5 Sonnet) - analyzes Usage + Weather + Community Needs
- Voice: Amazon Polly (text-to-speech in local languages explaining fairness decisions)
- Integration: Open-source weather APIs
- Community Analytics: Village-wide usage patterns for fairness optimization

## Frontend Layer

- Framework: React.js
- Styling: Tailwind CSS
- Target: FPO/Panchayat admin dashboard
- Features: Village-wide water stress heatmaps, community fairness metrics, equitable usage tracking

## Key Libraries & Dependencies

### Edge (ESP32)
- ESP-IDF or Arduino framework
- FFT library for signal processing
- MQTT client library

### Cloud (AWS)
- AWS SDK for Lambda functions
- Boto3 (Python) for AWS service integration
- Weather API clients (to be specified)

### Frontend
- React.js
- Tailwind CSS
- Map visualization library (Leaflet or similar)
- Audio player components

## Common Commands

```bash
# Edge firmware (ESP32)
# Build firmware
pio run

# Upload to device
pio run --target upload

# Monitor serial output
pio device monitor

# Cloud (AWS)
# Deploy Lambda functions
sam deploy

# Test Lambda locally
sam local invoke

# Frontend
# Install dependencies
npm install

# Run development server
npm run dev

# Build for production
npm run build

# Run tests
npm test
```

## Development Setup

### Required Tools
- PlatformIO or Arduino IDE (for ESP32 development)
- AWS CLI configured with credentials
- Node.js and npm (for frontend)
- AWS SAM CLI (for Lambda deployment)

### Environment Variables
- AWS_REGION
- AWS_IOT_ENDPOINT
- BEDROCK_MODEL_ID
- WEATHER_API_KEY
- DYNAMODB_TABLE_NAME

### Hardware Setup
- ESP32-S3 development board
- Piezoelectric sensor
- 5V relay module
- Test pump or vibration source for development

## Cost Structure

Bill of Materials (per unit):
- ESP32-S3: ₹450
- Piezo Sensor: ₹50
- Relay & Wires: ₹200
- Total: ~₹700 per unit
