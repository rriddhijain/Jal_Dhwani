# Requirements Document: Acoustic Pump Monitor

**Competition Track**: AI for Communities, Access & Public Impact

## Introduction

Jal Dhwani is an edge-AI acoustic monitoring system that prevents groundwater over-extraction and ensures community fairness in rural India. The system uses non-invasive vibration analysis to detect resource depletion, employs generative AI for community-aware irrigation scheduling, and provides autonomous pump control with voice-first farmer notifications explaining fairness decisions. The system operates across three tiers: edge devices (ESP32-S3 with piezoelectric sensors running offline AI), cloud infrastructure (AWS IoT Core, Lambda, Bedrock analyzing Usage + Weather + Community Needs), and frontend dashboards (React-based admin interface with fairness metrics).

## Glossary

- **Acoustic_Monitor**: The edge device consisting of ESP32-S3 microcontroller, piezoelectric sensor, and relay module with offline AI capabilities
- **Resource_Depletion**: Well water level dropping below sustainable threshold, detected via acoustic signatures (frequency changes in 2-10 kHz range)
- **FFT_Processor**: Fast Fourier Transform algorithm running on ESP32-S3 for frequency analysis and offline resource depletion detection
- **Telemetry_Service**: AWS Lambda functions that process pump data and community usage patterns
- **AI_Engine**: Amazon Bedrock (Claude 3.5 Sonnet) that synthesizes Usage + Weather + Community Needs for fairness-aware decisions
- **Voice_Notifier**: Amazon Polly service for text-to-speech in local languages explaining fairness and safety decisions
- **Admin_Dashboard**: React-based web interface for FPO/Panchayat administrators showing community fairness metrics
- **Relay_Controller**: 5V relay module that controls pump power for safety and fairness
- **MQTT_Broker**: AWS IoT Core message broker for device-cloud communication
- **Community_Fairness**: Equitable water distribution across village farmers based on usage patterns and needs
- **FPO**: Farmer Producer Organization
- **Panchayat**: Village-level administrative body
- **Public_Impact**: Measurable improvement in community water equity and resource sustainability

## Requirements

### Requirement 1: Acoustic Vibration Capture

**User Story:** As a farmer, I want the system to monitor my pump's vibration continuously, so that it can detect when my well is running dry.

#### Acceptance Criteria

1. WHEN the pump is powered on, THE Acoustic_Monitor SHALL begin capturing vibration data from the piezoelectric sensor
2. WHILE the pump is running, THE Acoustic_Monitor SHALL sample vibration data at a minimum frequency of 1 kHz
3. WHEN vibration amplitude exceeds sensor range, THE Acoustic_Monitor SHALL log a saturation event and continue monitoring
4. THE Acoustic_Monitor SHALL buffer at least 2 seconds of vibration data for FFT analysis
5. WHEN the pump is powered off, THE Acoustic_Monitor SHALL stop data capture and enter low-power mode

### Requirement 2: Real-Time Resource Depletion Detection

**User Story:** As a farmer, I want the system to detect resource depletion immediately, so that my pump motor doesn't get damaged and community water resources are protected.

#### Acceptance Criteria

1. WHEN vibration data is captured, THE FFT_Processor SHALL perform frequency analysis on the ESP32-S3 using offline AI
2. WHEN the FFT analysis detects frequency signatures characteristic of resource depletion (typically 2-10 kHz range), THE Acoustic_Monitor SHALL classify the pump state as "resource_depleted"
3. WHEN resource depletion is detected, THE Acoustic_Monitor SHALL trigger an immediate pump shutdown within 2 seconds for safety
4. THE FFT_Processor SHALL distinguish between normal pump operation and resource depletion with at least 90% accuracy
5. WHILE operating offline (no internet), THE Acoustic_Monitor SHALL continue resource depletion detection and emergency shutdown

### Requirement 3: Edge-to-Cloud Telemetry

**User Story:** As a system administrator, I want pump data transmitted to the cloud, so that AI can analyze community usage patterns and optimize for fairness.

#### Acceptance Criteria

1. WHEN resource depletion is detected, THE Acoustic_Monitor SHALL publish telemetry data to the MQTT_Broker within 5 seconds
2. WHEN the pump state changes (on/off/resource_depleted), THE Acoustic_Monitor SHALL publish a state change event via MQTT including usage duration
3. WHILE connected to Wi-Fi, THE Acoustic_Monitor SHALL publish periodic health telemetry and usage metrics every 5 minutes
4. WHEN MQTT connection fails, THE Acoustic_Monitor SHALL buffer telemetry locally and retry transmission every 30 seconds
5. THE Acoustic_Monitor SHALL include device ID, timestamp, pump state, FFT frequency peaks, usage duration, and sensor readings in telemetry messages

### Requirement 4: Cloud Data Ingestion

**User Story:** As a system architect, I want telemetry data reliably ingested into the cloud, so that it can be stored and analyzed.

#### Acceptance Criteria

1. WHEN telemetry arrives at the MQTT_Broker, THE Telemetry_Service SHALL validate the message schema
2. WHEN telemetry is valid, THE Telemetry_Service SHALL store it in DynamoDB with device ID and timestamp as keys
3. IF telemetry is malformed, THEN THE Telemetry_Service SHALL log the error and discard the message
4. THE Telemetry_Service SHALL process incoming telemetry within 1 second of receipt
5. WHEN telemetry is stored, THE Telemetry_Service SHALL trigger downstream analysis functions

### Requirement 5: Community-Aware AI Irrigation Scheduling

**User Story:** As a farmer, I want AI to decide when I should irrigate considering both my needs and community fairness, so that water is distributed equitably.

#### Acceptance Criteria

1. WHEN pump telemetry indicates repeated resource depletion events, THE AI_Engine SHALL analyze village-wide usage patterns and resource health
2. WHEN generating irrigation recommendations, THE AI_Engine SHALL synthesize Usage + Weather + Community Needs (pump health data, weather forecasts, and village-wide usage fairness metrics)
3. WHEN the AI_Engine determines irrigation should be delayed for fairness or safety, THE AI_Engine SHALL generate a pump control command with community-aware reasoning
4. THE AI_Engine SHALL provide recommendations in structured format including action (start/stop/delay), duration, fairness impact, and natural language explanation
5. WHEN weather forecasts predict rain within 24 hours OR community usage is imbalanced, THE AI_Engine SHALL factor this into irrigation scheduling

### Requirement 6: Autonomous Pump Control

**User Story:** As a farmer, I want the system to automatically control my pump, so that I don't have to manually monitor it.

#### Acceptance Criteria

1. WHEN the AI_Engine generates a pump control command, THE Telemetry_Service SHALL publish the command to the device via MQTT
2. WHEN the Acoustic_Monitor receives a pump control command, THE Relay_Controller SHALL execute the command within 2 seconds
3. WHEN executing a "stop" command, THE Relay_Controller SHALL open the relay to cut pump power
4. WHEN executing a "start" command, THE Relay_Controller SHALL close the relay to restore pump power
5. WHEN a control command is executed, THE Acoustic_Monitor SHALL publish a confirmation message to the MQTT_Broker

### Requirement 7: Voice-First Farmer Notifications with Fairness Context

**User Story:** As a farmer, I want to receive voice calls in my local language explaining why the pump was stopped and how it relates to community fairness, so that I understand both safety and equity concerns.

#### Acceptance Criteria

1. WHEN the pump is automatically stopped for fairness or safety, THE Voice_Notifier SHALL generate a voice message in the farmer's configured language
2. WHEN generating voice messages, THE Voice_Notifier SHALL use the AI_Engine's natural language explanation including fairness reasoning and community impact
3. THE Voice_Notifier SHALL support Hindi, Tamil, Telugu, and Marathi languages
4. WHEN a voice message is generated, THE Voice_Notifier SHALL initiate a phone call to the farmer's registered number
5. WHEN the farmer answers, THE Voice_Notifier SHALL play the synthesized voice message explaining the pump action, safety concerns, and community fairness considerations

### Requirement 8: Community Fairness Dashboard with Public Impact Metrics

**User Story:** As a Panchayat administrator, I want to see village-wide water stress patterns and fairness metrics, so that I can coordinate equitable community water management and measure public impact.

#### Acceptance Criteria

1. WHEN an administrator accesses the Admin_Dashboard, THE Admin_Dashboard SHALL display a map of all pumps in the village with fairness metrics
2. WHEN displaying pump locations, THE Admin_Dashboard SHALL color-code pumps based on resource depletion frequency and usage fairness (green/yellow/red heatmap)
3. WHEN an administrator selects a pump, THE Admin_Dashboard SHALL show detailed telemetry including resource depletion events, runtime hours, usage fairness score, and AI recommendations
4. THE Admin_Dashboard SHALL update pump status and community fairness metrics in real-time as telemetry arrives
5. WHEN multiple pumps show high resource depletion rates OR usage imbalance, THE Admin_Dashboard SHALL highlight the area as a water-stressed zone requiring fairness intervention

### Requirement 9: Offline Edge Safety with Local AI

**User Story:** As a farmer in an area with unreliable internet, I want critical safety features to work offline using local AI, so that my pump is protected even without connectivity.

#### Acceptance Criteria

1. WHILE the Acoustic_Monitor has no internet connection, THE Acoustic_Monitor SHALL continue resource depletion detection using local FFT analysis and offline AI
2. WHILE operating offline, THE Acoustic_Monitor SHALL execute emergency pump shutdown when resource depletion is detected for safety
3. WHEN internet connectivity is restored, THE Acoustic_Monitor SHALL upload buffered telemetry and usage data to the cloud for community fairness analysis
4. THE Acoustic_Monitor SHALL store at least 24 hours of telemetry and usage metrics locally when offline
5. WHEN local storage is full, THE Acoustic_Monitor SHALL overwrite the oldest telemetry data

### Requirement 10: Low-Cost Hardware Design

**User Story:** As a product manager, I want the device to cost under ₹700 per unit, so that it's affordable for smallholder farmers.

#### Acceptance Criteria

1. THE Acoustic_Monitor SHALL use an ESP32-S3 microcontroller (₹450 cost)
2. THE Acoustic_Monitor SHALL use a piezoelectric disc sensor (₹50 cost)
3. THE Acoustic_Monitor SHALL use a 5V relay module with wiring (₹200 cost)
4. THE Acoustic_Monitor SHALL have a total bill of materials not exceeding ₹700
5. THE Acoustic_Monitor SHALL be manufacturable using standard PCB assembly processes

### Requirement 11: Non-Invasive Retrofit Installation

**User Story:** As a farmer, I want to install the device on my existing pump without modifications, so that installation is quick and reversible.

#### Acceptance Criteria

1. THE Acoustic_Monitor SHALL attach to the pump casing using a clip or adhesive mount
2. THE Acoustic_Monitor SHALL not require drilling, welding, or permanent modifications to the pump
3. THE Relay_Controller SHALL connect inline with the pump's power supply using standard electrical connectors
4. THE Acoustic_Monitor SHALL be installable by a farmer or technician in under 10 minutes
5. THE Acoustic_Monitor SHALL work with common Indian pump models (0.5 HP to 5 HP submersible and centrifugal pumps)

### Requirement 12: Data Persistence and Retrieval

**User Story:** As a data analyst, I want historical pump data stored reliably, so that I can analyze long-term trends and improve the AI model.

#### Acceptance Criteria

1. WHEN telemetry is stored in DynamoDB, THE Telemetry_Service SHALL retain data for at least 2 years
2. WHEN querying historical data, THE Telemetry_Service SHALL support queries by device ID, time range, and pump state
3. THE Telemetry_Service SHALL store FFT frequency peaks, pump state, timestamp, and AI recommendations for each event
4. WHEN data exceeds 2 years, THE Telemetry_Service SHALL archive old data to S3 for long-term storage
5. THE Telemetry_Service SHALL support exporting data in CSV format for external analysis

### Requirement 13: Weather API Integration

**User Story:** As the AI engine, I need access to weather forecasts, so that I can factor rainfall predictions into irrigation decisions.

#### Acceptance Criteria

1. WHEN generating irrigation recommendations, THE AI_Engine SHALL query weather forecast APIs for the pump's geographic location
2. THE AI_Engine SHALL retrieve at least 7-day weather forecasts including rainfall probability and temperature
3. WHEN weather API calls fail, THE AI_Engine SHALL fall back to irrigation recommendations based solely on pump health data
4. THE AI_Engine SHALL cache weather data for 6 hours to minimize API costs
5. THE AI_Engine SHALL support integration with open-source weather APIs (OpenWeatherMap or similar)

### Requirement 14: Device Configuration and Provisioning

**User Story:** As a field technician, I want to configure device settings during installation, so that each pump is properly registered in the system.

#### Acceptance Criteria

1. WHEN the Acoustic_Monitor is first powered on, THE Acoustic_Monitor SHALL enter provisioning mode
2. WHILE in provisioning mode, THE Acoustic_Monitor SHALL broadcast a Wi-Fi access point for configuration
3. WHEN a technician connects to the provisioning AP, THE Acoustic_Monitor SHALL serve a web interface for entering Wi-Fi credentials, farmer phone number, and location
4. WHEN configuration is saved, THE Acoustic_Monitor SHALL connect to the configured Wi-Fi network and register with the MQTT_Broker
5. THE Acoustic_Monitor SHALL store configuration in non-volatile memory to persist across power cycles

### Requirement 15: System Health Monitoring

**User Story:** As a system administrator, I want to monitor device health, so that I can identify and fix failing devices proactively.

#### Acceptance Criteria

1. WHILE connected to the cloud, THE Acoustic_Monitor SHALL publish device health metrics every 15 minutes
2. THE Acoustic_Monitor SHALL report battery voltage (if battery-backed), Wi-Fi signal strength, uptime, and firmware version
3. WHEN device health metrics indicate low battery or weak signal, THE Admin_Dashboard SHALL display a warning
4. WHEN a device hasn't reported for 1 hour, THE Admin_Dashboard SHALL mark it as "offline"
5. THE Telemetry_Service SHALL track device uptime and generate availability reports

### Requirement 16: Community Fairness Metrics and Public Impact Tracking

**User Story:** As a Panchayat administrator, I want to track community-level fairness metrics and public impact, so that I can ensure equitable water distribution and measure social outcomes.

#### Acceptance Criteria

1. WHEN analyzing village-wide data, THE AI_Engine SHALL calculate fairness scores based on usage distribution across all farmers
2. THE Admin_Dashboard SHALL display community fairness metrics including usage variance, equity index, and resource sustainability indicators
3. WHEN usage imbalance exceeds threshold (top 20% farmers using >40% of water), THE Admin_Dashboard SHALL alert administrators
4. THE Telemetry_Service SHALL track public impact metrics including water saved, equitable access improvements, and community sustainability scores
5. THE Admin_Dashboard SHALL generate monthly public impact reports showing fairness trends and resource conservation outcomes

