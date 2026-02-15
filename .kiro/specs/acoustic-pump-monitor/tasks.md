# Implementation Plan: Acoustic Pump Monitor

## Overview

This implementation plan breaks down the Acoustic Pump Monitor system into three parallel development tracks: Edge (ESP32-S3 firmware in C++), Cloud (AWS Lambda functions in Python), and Frontend (React dashboard). The plan follows an incremental approach where each component is built with testing integrated throughout, culminating in end-to-end integration.

## Tasks

### Track 1: Edge Device Firmware (ESP32-S3)

- [ ] 1. Set up ESP32-S3 firmware project structure
  - Initialize PlatformIO project with ESP-IDF framework
  - Configure platformio.ini with ESP32-S3 board settings
  - Add ESP-DSP library for FFT processing
  - Add MQTT client library (PubSubClient or AWS IoT SDK)
  - Create header files for core interfaces (PiezoSensor, FFTProcessor, CavitationDetector, RelayController, MQTTClient)
  - _Requirements: 1.1, 1.2, 2.1, 3.1, 6.2, 14.1_

- [ ] 2. Implement piezoelectric sensor interface
  - [ ] 2.1 Create PiezoSensor class with ADC sampling
    - Implement initialize() to configure ADC pin and sampling rate
    - Implement readSample() to read raw ADC values
    - Implement startContinuousSampling() with timer-based interrupts
    - Implement circular buffer for storing samples (2 seconds at 1 kHz = 2048 samples)
    - _Requirements: 1.1, 1.2, 1.4_
  
  - [ ]* 2.2 Write property test for sampling rate guarantee
    - **Property 1: Sampling Rate Guarantee**
    - **Validates: Requirements 1.2**
    - Generate random sampling durations, verify rate ≥ 1 kHz
  
  - [ ]* 2.3 Write property test for buffer capacity
    - **Property 2: Buffer Capacity Invariant**
    - **Validates: Requirements 1.4**
    - Verify buffer holds at least 2 seconds of samples
  
  - [ ]* 2.4 Write unit test for sensor saturation handling
    - Test that saturated ADC values (0 or 4095) are logged and monitoring continues
    - _Requirements: 1.3_

- [ ] 3. Implement FFT processor and cavitation detector
  - [ ] 3.1 Create FFTProcessor class using ESP-DSP
    - Implement initialize() to allocate FFT buffers
    - Implement computeFFT() using dsps_fft2r_fc32 (ESP-DSP function)
    - Apply Hann window before FFT to reduce spectral leakage
    - Implement findPeaks() to identify frequency peaks
    - Implement getEnergyInBand() to calculate energy in frequency range
    - _Requirements: 2.1, 2.2_
  
  - [ ] 3.2 Create CavitationDetector class with rule-based detection
    - Implement detectState() using energy ratio threshold (cavitation band / baseline > 2.5)
    - Implement updateBaseline() to learn normal pump operation during first 60 seconds
    - Require 3 consecutive detections to confirm cavitation (avoid false positives)
    - _Requirements: 2.2_
  
  - [ ]* 3.3 Write property test for cavitation classification
    - **Property 3: Cavitation Classification Correctness**
    - **Validates: Requirements 2.2**
    - Generate FFT spectra with elevated 2-10 kHz energy, verify "cavitating" classification
  
  - [ ]* 3.4 Write unit tests for FFT edge cases
    - Test with all-zero input (silent pump)
    - Test with saturated input (clipped signal)
    - Test with known sine wave (verify frequency peak detection)
    - _Requirements: 2.1_

- [ ] 4. Implement relay controller for pump control
  - [ ] 4.1 Create RelayController class
    - Implement initialize() to configure GPIO pin (active high/low)
    - Implement setState() to open/close relay
    - Implement emergencyStop() for immediate shutdown
    - Add relay state feedback verification (if hardware supports)
    - _Requirements: 6.2, 6.3, 6.4_
  
  - [ ]* 4.2 Write property test for command-to-relay mapping
    - **Property 18: Command-to-Relay Mapping**
    - **Validates: Requirements 6.3, 6.4**
    - For any command, verify "stop" opens relay and "start" closes relay
  
  - [ ]* 4.3 Write property test for emergency shutdown timing
    - **Property 4: Emergency Shutdown Timing**
    - **Validates: Requirements 2.3**
    - Verify relay opens within 2 seconds of cavitation detection

- [ ] 5. Implement MQTT client for cloud communication
  - [ ] 5.1 Create MQTTClient class with AWS IoT Core integration
    - Implement connect() with TLS certificate authentication
    - Implement publish() for telemetry and status messages
    - Implement subscribe() for control commands with callback
    - Implement connection retry logic with exponential backoff
    - _Requirements: 3.1, 3.2, 6.1_
  
  - [ ] 5.2 Implement telemetry message serialization
    - Create JSON serialization for TelemetryMessage struct
    - Include device_id, timestamp, pump_state, fft_peaks, cavitation_confidence, relay_state
    - _Requirements: 3.5_
  
  - [ ]* 5.3 Write property test for telemetry schema compliance
    - **Property 9: Telemetry Schema Compliance**
    - **Validates: Requirements 3.5**
    - Generate random telemetry data, verify all required fields present
  
  - [ ]* 5.4 Write property test for MQTT retry on failure
    - **Property 8: MQTT Retry on Failure**
    - **Validates: Requirements 3.4**
    - Simulate connection failures, verify buffering and 30-second retry

- [ ] 6. Implement offline operation and local buffering
  - [ ] 6.1 Create local telemetry buffer with circular queue
    - Implement 24-hour buffer (288 messages at 5-minute intervals)
    - Implement FIFO overflow behavior (overwrite oldest)
    - Store buffer in SPIFFS or LittleFS filesystem
    - _Requirements: 9.3, 9.4, 9.5_
  
  - [ ]* 6.2 Write property test for offline safety guarantee
    - **Property 5: Offline Safety Guarantee**
    - **Validates: Requirements 2.5, 9.1, 9.2**
    - Simulate offline state, verify cavitation detection and shutdown continue
  
  - [ ]* 6.3 Write property test for circular buffer overflow
    - **Property 30: Circular Buffer Overflow**
    - **Validates: Requirements 9.5**
    - Fill buffer, write new message, verify oldest overwritten
  
  - [ ]* 6.4 Write property test for connectivity restoration sync
    - **Property 28: Connectivity Restoration Sync**
    - **Validates: Requirements 9.3**
    - Simulate offline→online transition, verify buffered data uploaded

- [ ] 7. Implement device provisioning and configuration
  - [ ] 7.1 Create provisioning mode with Wi-Fi AP
    - Detect first boot (check for saved config in NVS)
    - Start Wi-Fi access point with SSID "JalDhwani-{device_id}"
    - Serve captive portal web interface for configuration
    - _Requirements: 14.1, 14.2, 14.3_
  
  - [ ] 7.2 Create web interface for device configuration
    - HTML form for Wi-Fi SSID/password, farmer phone, location (lat/lng), language
    - Save configuration to NVS (non-volatile storage)
    - Restart device and connect to configured Wi-Fi
    - _Requirements: 14.3, 14.4_
  
  - [ ]* 7.3 Write property test for configuration persistence
    - **Property 40: Configuration Persistence Round-Trip**
    - **Validates: Requirements 14.5**
    - Save random config, reboot, verify loaded config matches

- [ ] 8. Implement main firmware loop and state machine
  - [ ] 8.1 Create main loop integrating all components
    - Initialize all peripherals (sensor, relay, MQTT)
    - Run state machine: PROVISIONING → CONNECTING → MONITORING → OFFLINE
    - In MONITORING state: sample sensor → FFT → detect cavitation → control relay → publish telemetry
    - Publish periodic health telemetry every 15 minutes
    - Handle incoming MQTT control commands
    - _Requirements: 1.1, 1.5, 2.3, 3.3, 6.2, 15.1_
  
  - [ ]* 8.2 Write property test for event-driven telemetry
    - **Property 6: Event-Driven Telemetry Publishing**
    - **Validates: Requirements 3.1, 3.2**
    - Generate random state changes, verify telemetry published within 5 seconds
  
  - [ ]* 8.3 Write property test for periodic health telemetry
    - **Property 7: Periodic Health Telemetry**
    - **Validates: Requirements 3.3, 15.1**
    - Simulate 15-minute windows, verify at least one health message published

- [ ] 9. Checkpoint - Edge firmware integration test
  - Flash firmware to ESP32-S3 hardware
  - Connect piezoelectric sensor and relay module
  - Verify provisioning mode and configuration
  - Test with running pump (or vibration source)
  - Verify cavitation detection and relay control
  - Verify MQTT telemetry publishing
  - Ensure all tests pass, ask the user if questions arise.

### Track 2: Cloud Infrastructure (AWS Lambda + DynamoDB)

- [ ] 10. Set up AWS infrastructure with SAM template
  - Create AWS SAM template.yaml defining all resources
  - Define DynamoDB table: jal-dhwani-telemetry (partition key: device_id, sort key: timestamp, TTL: 2 years)
  - Define IoT Core thing type and policy for device authentication
  - Define Lambda functions: ingest, analyze, control, notify, query
  - Define API Gateway for frontend REST API
  - Configure IAM roles and permissions
  - _Requirements: 4.1, 4.2, 12.1_

- [ ] 11. Implement ingest Lambda function
  - [ ] 11.1 Create ingest Lambda with telemetry validation
    - Parse incoming IoT Core event (MQTT message)
    - Validate telemetry schema (required fields and types)
    - Store valid telemetry in DynamoDB with TTL
    - Trigger EventBridge event for downstream analysis
    - Log errors for invalid telemetry
    - _Requirements: 4.1, 4.2, 4.3, 4.5_
  
  - [ ]* 11.2 Write property test for schema validation
    - **Property 10: Schema Validation Enforcement**
    - **Validates: Requirements 4.1**
    - Generate random telemetry, verify validation performed
  
  - [ ]* 11.3 Write property test for storage round-trip
    - **Property 11: Telemetry Storage Round-Trip**
    - **Validates: Requirements 4.2**
    - Store random telemetry, query by keys, verify equivalent data returned
  
  - [ ]* 11.4 Write property test for invalid message rejection
    - **Property 12: Invalid Message Rejection**
    - **Validates: Requirements 4.3**
    - Generate malformed messages, verify rejection and error logging

- [ ] 12. Implement analyze Lambda function with Bedrock AI
  - [ ] 12.1 Create analyze Lambda with AI integration
    - Query recent telemetry for device (last 24 hours)
    - Fetch weather forecast from OpenWeatherMap API
    - Call Amazon Bedrock (Claude 3.5 Sonnet) with prompt template
    - Parse AI response (JSON with action, duration, reasoning)
    - Invoke control Lambda with recommendation
    - _Requirements: 5.1, 5.2, 5.3, 13.1, 13.2_
  
  - [ ] 12.2 Implement weather API integration with caching
    - Query OpenWeatherMap API with device coordinates
    - Cache weather data in DynamoDB with 6-hour TTL
    - Implement fallback logic for API failures
    - _Requirements: 13.1, 13.2, 13.3, 13.4_
  
  - [ ]* 12.3 Write property test for AI input completeness
    - **Property 14: AI Input Completeness**
    - **Validates: Requirements 5.2**
    - Verify both pump telemetry and weather data queried before AI call
  
  - [ ]* 12.4 Write property test for AI output structure
    - **Property 15: AI Output Structure**
    - **Validates: Requirements 5.3, 5.4**
    - Verify AI response contains action, duration_minutes, reasoning
  
  - [ ]* 12.5 Write property test for weather API fallback
    - **Property 36: Weather API Fallback**
    - **Validates: Requirements 13.3**
    - Simulate API failure, verify recommendations generated with pump data only
  
  - [ ]* 12.6 Write property test for weather cache reuse
    - **Property 37: Weather Cache Reuse**
    - **Validates: Requirements 13.4**
    - Cache weather data, verify subsequent requests use cache within 6 hours

- [ ] 13. Implement control Lambda function
  - [ ] 13.1 Create control Lambda for pump commands
    - Receive recommendation from analyze Lambda
    - Publish MQTT control command to device via IoT Core
    - Invoke notify Lambda for voice notification
    - Log control action to DynamoDB
    - _Requirements: 6.1, 7.1_
  
  - [ ]* 13.2 Write property test for control command propagation
    - **Property 16: Control Command Propagation**
    - **Validates: Requirements 6.1**
    - Generate AI commands, verify MQTT messages published to device topics

- [ ] 14. Implement notify Lambda function with Polly
  - [ ] 14.1 Create notify Lambda with voice generation
    - Receive device_id and reasoning text
    - Query device configuration for farmer phone and language
    - Call Amazon Polly to synthesize speech (support hi-IN, ta-IN, te-IN, mr-IN)
    - Store audio file in S3
    - Initiate phone call using Twilio or Amazon Connect
    - _Requirements: 7.1, 7.2, 7.3, 7.4_
  
  - [ ]* 14.2 Write property test for voice notification generation
    - **Property 20: Voice Notification Generation**
    - **Validates: Requirements 7.1**
    - Simulate pump stop events, verify voice messages generated
  
  - [ ]* 14.3 Write property test for multi-language support
    - **Property 22: Multi-Language Support**
    - **Validates: Requirements 7.3**
    - For each language {hi, ta, te, mr}, verify Polly generates speech
  
  - [ ]* 14.4 Write unit test for voice message content
    - Verify generated message contains AI reasoning text
    - _Requirements: 7.2_

- [ ] 15. Implement query Lambda function for dashboard API
  - [ ] 15.1 Create query Lambda with multiple query types
    - Implement device_status query (latest telemetry for device)
    - Implement village_heatmap query (all devices in village with cavitation stats)
    - Implement telemetry_history query (time range for device)
    - Implement health_metrics query (device uptime and availability)
    - _Requirements: 8.1, 8.2, 12.2, 15.5_
  
  - [ ]* 15.2 Write property test for query parameter support
    - **Property 31: Query Parameter Support**
    - **Validates: Requirements 12.2**
    - Generate random query filters, verify matching records returned

- [ ] 16. Implement data archival and export functions
  - [ ] 16.1 Create archival Lambda triggered by DynamoDB TTL
    - Query telemetry older than 2 years
    - Export to S3 in Parquet format for long-term storage
    - Delete from DynamoDB after successful archival
    - _Requirements: 12.4_
  
  - [ ] 16.2 Create export Lambda for CSV generation
    - Query telemetry based on filters
    - Convert to CSV format
    - Return pre-signed S3 URL for download
    - _Requirements: 12.5_
  
  - [ ]* 16.3 Write property test for CSV export round-trip
    - **Property 33: CSV Export Round-Trip**
    - **Validates: Requirements 12.5**
    - Export random telemetry to CSV, parse back, verify data preserved

- [ ] 17. Checkpoint - Cloud infrastructure deployment
  - Deploy SAM template to AWS account
  - Configure IoT Core certificates for test device
  - Test ingest Lambda with simulated MQTT messages
  - Test analyze Lambda with Bedrock API
  - Test notify Lambda with Polly TTS
  - Verify DynamoDB storage and queries
  - Ensure all tests pass, ask the user if questions arise.

### Track 3: Frontend Dashboard (React + Tailwind)

- [ ] 18. Set up React project structure
  - Initialize React project with Vite
  - Install dependencies: react-leaflet (maps), tailwindcss, axios, recharts (graphs)
  - Configure Tailwind CSS
  - Create directory structure: components/, services/, utils/
  - Set up environment variables for API endpoint
  - _Requirements: 8.1_

- [ ] 19. Implement API service layer
  - [ ] 19.1 Create DashboardAPI class
    - Implement getVillagePumps(villageId) to fetch all pumps
    - Implement getPumpTelemetry(deviceId, startTime, endTime) for history
    - Implement getPumpStatus(deviceId) for real-time status
    - Implement WebSocket connection for real-time updates
    - _Requirements: 8.1, 8.4_
  
  - [ ]* 19.2 Write unit tests for API client
    - Mock API responses, verify correct data parsing
    - Test error handling for network failures
    - _Requirements: 8.1_

- [ ] 20. Implement PumpMap component with heatmap
  - [ ] 20.1 Create PumpMap component using react-leaflet
    - Display map centered on village coordinates
    - Render markers for each pump with color-coded status
    - Implement color logic: green (F < 2/day), yellow (2 ≤ F < 5/day), red (F ≥ 5/day)
    - Highlight water-stressed zones (≥3 red pumps in area)
    - _Requirements: 8.1, 8.2, 8.5_
  
  - [ ]* 20.2 Write property test for pump color coding
    - **Property 24: Pump Color Coding Logic**
    - **Validates: Requirements 8.2**
    - Generate random cavitation frequencies, verify correct colors assigned
  
  - [ ]* 20.3 Write property test for water-stressed zone detection
    - **Property 27: Water-Stressed Zone Detection**
    - **Validates: Requirements 8.5**
    - Generate pump clusters, verify zones highlighted when ≥3 pumps have F ≥ 5/day

- [ ] 21. Implement PumpCard component for detailed view
  - [ ] 21.1 Create PumpCard component
    - Display device_id, pump_state, runtime_hours, cavitation_count
    - Display last_seen timestamp
    - Display AI recommendations
    - Add "View Details" button to open full telemetry history
    - _Requirements: 8.3_
  
  - [ ]* 21.2 Write property test for pump detail completeness
    - **Property 25: Pump Detail Completeness**
    - **Validates: Requirements 8.3**
    - Generate random pump data, verify all required fields displayed

- [ ] 22. Implement real-time updates with WebSocket
  - [ ] 22.1 Create WebSocket connection manager
    - Connect to API Gateway WebSocket endpoint
    - Subscribe to pump updates for village
    - Update React state when new telemetry arrives
    - Implement reconnection logic with exponential backoff
    - _Requirements: 8.4_
  
  - [ ]* 22.2 Write property test for real-time dashboard updates
    - **Property 26: Real-Time Dashboard Updates**
    - **Validates: Requirements 8.4**
    - Simulate WebSocket messages, verify UI updates within 1 second

- [ ] 23. Implement device health monitoring UI
  - [ ] 23.1 Create HealthIndicator component
    - Display battery voltage, Wi-Fi signal strength, uptime, firmware version
    - Show warning icon for low battery (<3.3V) or weak signal (<-80 dBm)
    - Mark devices as "offline" if no telemetry for 60 minutes
    - _Requirements: 15.2, 15.3, 15.4_
  
  - [ ]* 23.2 Write property test for low health warning display
    - **Property 42: Low Health Warning Display**
    - **Validates: Requirements 15.3**
    - Generate health metrics below thresholds, verify warnings displayed
  
  - [ ]* 23.3 Write property test for offline device detection
    - **Property 43: Offline Device Detection**
    - **Validates: Requirements 15.4**
    - Generate devices with no recent telemetry, verify "offline" status

- [ ] 24. Implement telemetry history and analytics
  - [ ] 24.1 Create TelemetryChart component
    - Display time-series graph of cavitation events
    - Display runtime hours over time
    - Display pump state transitions
    - Allow date range selection
    - _Requirements: 12.2_
  
  - [ ] 24.2 Create AvailabilityReport component
    - Calculate and display device uptime percentage
    - Show availability trends over time
    - _Requirements: 15.5_
  
  - [ ]* 24.3 Write property test for uptime calculation
    - **Property 44: Uptime Calculation**
    - **Validates: Requirements 15.5**
    - Generate random telemetry history, verify uptime = (online_time / total_time) × 100

- [ ] 25. Checkpoint - Frontend integration test
  - Run development server locally
  - Connect to deployed AWS backend
  - Test map rendering with real pump data
  - Test real-time updates via WebSocket
  - Test pump detail views and telemetry history
  - Verify responsive design on mobile devices
  - Ensure all tests pass, ask the user if questions arise.

### Track 4: End-to-End Integration

- [ ] 26. Deploy complete system to test environment
  - [ ] 26.1 Deploy edge firmware to test devices
    - Flash firmware to multiple ESP32-S3 devices
    - Configure each device with unique ID and location
    - Install on test pumps or vibration sources
    - _Requirements: 14.1, 14.4_
  
  - [ ] 26.2 Deploy cloud infrastructure to AWS
    - Deploy SAM template to production AWS account
    - Configure IoT Core certificates for all test devices
    - Set up CloudWatch alarms for Lambda errors
    - _Requirements: 4.1, 5.1, 6.1, 7.1_
  
  - [ ] 26.3 Deploy frontend to hosting service
    - Build production React bundle
    - Deploy to S3 + CloudFront or Vercel
    - Configure API endpoint environment variables
    - _Requirements: 8.1_

- [ ] 27. Perform end-to-end system testing
  - [ ] 27.1 Test normal pump operation flow
    - Start pump, verify normal state detected
    - Verify telemetry published to cloud
    - Verify dashboard shows pump as "normal" (green)
    - _Requirements: 1.1, 2.2, 3.1, 8.2_
  
  - [ ] 27.2 Test cavitation detection and emergency shutdown
    - Simulate cavitation (run pump dry or use test signal)
    - Verify cavitation detected within 2 seconds
    - Verify relay opens to stop pump
    - Verify telemetry shows "cavitating" state
    - Verify dashboard shows pump as "cavitating" (red)
    - _Requirements: 2.2, 2.3, 6.3, 8.2_
  
  - [ ] 27.3 Test AI-driven irrigation scheduling
    - Trigger analyze Lambda with repeated cavitation events
    - Verify Bedrock AI generates recommendation
    - Verify control command sent to device
    - Verify pump controlled according to AI decision
    - _Requirements: 5.2, 5.3, 6.1, 6.2_
  
  - [ ] 27.4 Test voice notification delivery
    - Trigger automatic pump stop
    - Verify Polly generates voice message in configured language
    - Verify phone call initiated to farmer
    - _Requirements: 7.1, 7.2, 7.3, 7.4_
  
  - [ ] 27.5 Test offline operation
    - Disconnect device from Wi-Fi
    - Simulate cavitation
    - Verify local detection and shutdown still work
    - Reconnect Wi-Fi
    - Verify buffered telemetry uploaded
    - _Requirements: 9.1, 9.2, 9.3_
  
  - [ ] 27.6 Test dashboard real-time updates
    - Trigger pump state changes
    - Verify dashboard updates within 1 second
    - Test with multiple simultaneous devices
    - _Requirements: 8.4_

- [ ] 28. Final checkpoint - System validation
  - Verify all 44 correctness properties pass
  - Verify all functional requirements met
  - Verify system operates within cost constraints (₹700 BOM)
  - Verify installation time <10 minutes
  - Document any known issues or limitations
  - Ensure all tests pass, ask the user if questions arise.

## Notes

- Tasks marked with `*` are optional property-based tests and can be skipped for faster MVP
- Each task references specific requirements for traceability
- Tracks 1, 2, and 3 can be developed in parallel by different team members
- Checkpoints ensure incremental validation at component and system levels
- Property tests validate universal correctness properties across all inputs
- Unit tests validate specific examples, edge cases, and integration points
- End-to-end testing validates the complete closed-loop control system

