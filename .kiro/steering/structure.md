# Project Structure

## Current Organization

Jal Dhwani is organized into three main components: firmware (edge), cloud (AWS), and frontend (web dashboard).

## Directory Layout

```
/
├── .kiro/                  # Kiro configuration and steering rules
│   └── steering/           # AI assistant guidance documents
├── firmware/               # ESP32 edge device code
│   ├── src/                # C++ source files
│   │   ├── main.cpp        # Main firmware logic
│   │   ├── acoustic.cpp    # FFT and signal processing
│   │   ├── mqtt.cpp        # AWS IoT Core communication
│   │   └── relay.cpp       # Pump control logic
│   ├── include/            # Header files
│   ├── lib/                # Third-party libraries
│   ├── test/               # Unit tests for firmware
│   └── platformio.ini      # PlatformIO configuration
├── cloud/                  # AWS Lambda functions and infrastructure
│   ├── functions/          # Lambda function code
│   │   ├── ingest/         # IoT data ingestion handler
│   │   ├── analyze/        # Bedrock AI analysis
│   │   ├── control/        # Pump control commands
│   │   └── notify/         # Polly voice notifications
│   ├── layers/             # Lambda layers (shared dependencies)
│   ├── template.yaml       # AWS SAM template
│   └── tests/              # Cloud function tests
├── frontend/               # React web dashboard
│   ├── public/             # Static assets
│   ├── src/
│   │   ├── components/     # React components
│   │   │   ├── Map/        # Village heatmap component
│   │   │   ├── Dashboard/  # Admin dashboard
│   │   │   └── PumpCard/   # Individual pump status
│   │   ├── services/       # API clients
│   │   ├── utils/          # Helper functions
│   │   ├── App.jsx         # Main app component
│   │   └── index.jsx       # Entry point
│   ├── package.json
│   └── tailwind.config.js
├── docs/                   # Documentation
│   ├── architecture.md     # System architecture diagrams
│   ├── api.md              # API documentation
│   └── deployment.md       # Deployment guide
└── README.md               # Project overview
```

## Code Organization Patterns

### Firmware (Edge)
- All sensor logic in `firmware/src/acoustic.cpp`
- MQTT communication isolated in `firmware/src/mqtt.cpp`
- Hardware control (relay) in `firmware/src/relay.cpp`
- Configuration constants in header files

### Cloud (AWS Lambda)
- One function per responsibility (ingest, analyze, control, notify)
- Shared utilities in Lambda layers
- Infrastructure as Code using AWS SAM
- Environment-specific configs in `template.yaml`

### Frontend (React)
- Component-based architecture
- Reusable UI components in `components/`
- API calls centralized in `services/`
- Tailwind for styling (utility-first CSS)

## Naming Conventions

### Files
- Firmware: snake_case for C++ files (`acoustic_sensor.cpp`)
- Frontend: PascalCase for components (`PumpCard.jsx`)
- Cloud: kebab-case for Lambda functions (`analyze-pump-data`)

### Variables
- Firmware: camelCase for variables, UPPER_CASE for constants
- Frontend: camelCase for variables and functions
- Cloud: camelCase for Python/Node.js code

## Architecture Patterns

### Edge Intelligence
- Process FFT on-device to reduce cloud bandwidth
- Offline-first: Critical safety features work without internet
- MQTT for lightweight, reliable communication

### Serverless Cloud
- Event-driven architecture using Lambda
- Stateless functions for scalability
- DynamoDB for NoSQL telemetry storage

### Agentic AI Design
- AWS Bedrock (Claude 3.5) as decision engine
- Multi-factor reasoning: pump health + weather + crop stage
- Autonomous action: System controls pump without human intervention

### Voice-First UX
- Amazon Polly for accessibility in rural areas
- Local language support (Hindi, Tamil, etc.)
- Simple audio explanations for farmer notifications

## Development Principles

1. Edge-first safety: Critical features must work offline
2. Low-cost hardware: Target ₹700 BOM per unit
3. Retrofit design: Must work with existing pumps
4. Voice accessibility: Prioritize audio over visual UI for farmers
5. Community scale: Design for village-level deployment
