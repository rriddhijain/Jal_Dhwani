# Jal Dhwani: The Acoustic Water-Policy Engine 🌊🔊

**"Shazam for Water Pumps"**

[![Competition Track](https://img.shields.io/badge/Track-AI%20for%20Communities%2C%20Access%20%26%20Public%20Impact-blue)](https://aiforBharat.org)
[![Hackathon](https://img.shields.io/badge/AI%20for%20Bharat-2026-orange)](https://aiforBharat.org)
[![Cost](https://img.shields.io/badge/BOM%20Cost-₹700-green)](https://github.com)
[![License](https://img.shields.io/badge/License-MIT-yellow)](LICENSE)

---

## 🎯 The Problem: Water Inequity in Rural India

India is running out of groundwater. Farmers over-irrigate because they can't see underground water levels, and **there's no mechanism to ensure fair water distribution across communities**. 

**Current Solutions Fall Short:**
- Soil moisture sensors cost ₹5,000+ (unaffordable for smallholder farmers)
- Fragile hardware gets destroyed by tractors and corrosion
- **Most critically: They don't address community-level resource fairness**

**The Impact:**
- Top 20% of farmers often consume 40%+ of village water
- Wells run dry, causing motor burnout and crop failure
- No visibility into village-wide water stress patterns
- Panchayats lack tools for equitable resource management

---

## 💡 Our Solution: Community-First Acoustic Monitoring

Jal Dhwani is a **₹700 edge-AI device** that clips onto existing water pumps. It uses acoustic monitoring to detect resource depletion and employs **Generative AI to balance individual farmer needs with community fairness**.

### The Architecture (Slide 5 Logic)

```
Input: Pump starts → Acoustic Sensor
  ↓
Edge: ESP32 runs offline AI → Detects Resource Depletion
  ↓
Cloud Reasoning: AWS Bedrock analyzes Usage + Weather + Community Needs
  ↓
Decision: AI decides 'Stop Pump for Fairness/Safety'
  ↓
Output: Amazon Polly triggers Voice Call to the farmer
```

---

## 🚀 Key Features

### 1. **Resource Depletion Detection** (Edge AI)
- **₹50 piezoelectric sensor** "listens" to pump vibrations
- **ESP32-S3 performs FFT analysis** to detect well depletion (2-10 kHz frequency signatures)
- **Offline AI ensures safety** even without internet connectivity
- **2-second emergency shutdown** prevents motor burnout

### 2. **Community-Aware AI Scheduling** (Cloud)
- **AWS Bedrock (Claude 3.5 Sonnet)** analyzes:
  - **Usage**: Individual pump runtime + village-wide patterns
  - **Weather**: 7-day rainfall forecasts
  - **Community Needs**: Fairness scores, equity index, usage distribution
- **Balances individual irrigation needs with village-wide fairness**
- Prevents top farmers from monopolizing water resources

### 3. **Voice-First Farmer Notifications** (Accessibility)
- **Amazon Polly** generates calls in **Hindi, Tamil, Telugu, Marathi**
- Explains **both safety concerns AND community fairness context**
- Example: *"पंप में संसाधन की कमी है। समुदाय में निष्पक्षता के लिए 1 घंटे बंद रहेगा।"*
  - *(Resource depletion detected. Pump stopped for 1 hour to ensure community fairness.)*

### 4. **Community Fairness Dashboard** (Public Impact)
- **Village-wide heatmap** showing resource stress and usage equity
- **Real-time fairness metrics**: Equity index, usage distribution, sustainability scores
- **Alerts for usage imbalance** (e.g., top 20% using >40% of water)
- **Monthly public impact reports** tracking water saved and fairness improvements

### 5. **Zero-Internet Edge Protection** (Resilience)
- **Offline AI** continues resource depletion detection without cloud
- **Local buffering** stores 24 hours of telemetry when offline
- **Auto-sync** uploads data when connectivity restores

---

## 🏗️ Technical Architecture

### **Edge Layer** (ESP32-S3 + Piezoelectric Sensor)
- **Hardware**: ESP32-S3 (₹450), Piezo Sensor (₹50), 5V Relay (₹200)
- **Processing**: On-device FFT (Fast Fourier Transform) for frequency analysis
- **Offline AI**: Local resource depletion detection without cloud dependency
- **Communication**: MQTT over TLS to AWS IoT Core

### **Cloud Layer** (AWS Serverless)
- **Ingestion**: AWS IoT Core → Lambda → DynamoDB (telemetry + community usage data)
- **AI Brain**: AWS Bedrock (Claude 3.5 Sonnet) for fairness-aware decision making
- **Voice**: Amazon Polly (text-to-speech in 4 Indian languages)
- **Analytics**: Village-wide usage patterns, equity calculations, public impact tracking

### **Frontend Layer** (React Dashboard)
- **Framework**: React.js + Tailwind CSS
- **Features**: Interactive village map, fairness heatmaps, real-time WebSocket updates
- **Target Users**: FPO/Panchayat administrators

---

## 📊 Public Impact Metrics

Our system tracks measurable community outcomes:

| Metric | Description |
|--------|-------------|
| **Equity Index** | Gini coefficient of water usage distribution (0 = perfect equality) |
| **Usage Fairness Score** | Percentile ranking of each farmer's consumption |
| **Resource Sustainability** | Days until predicted aquifer depletion |
| **Water Saved** | Liters conserved through AI-optimized scheduling |
| **Fairness Interventions** | Number of times AI delayed pumps for equity |

**Example Dashboard Alert:**
> ⚠️ **Water-Stressed Zone Detected**: 5 pumps in North Village showing high depletion. Top 3 farmers using 55% of water. Fairness intervention recommended.

---

## 🛠️ Installation & Deployment

### **Hardware Setup** (10 minutes)
1. Clip piezoelectric sensor onto pump casing (no drilling required)
2. Connect 5V relay inline with pump power supply
3. Power on ESP32-S3 device
4. Connect to "JalDhwani-{device_id}" Wi-Fi AP
5. Configure via web interface: Wi-Fi credentials, farmer phone, location, language

### **Cloud Deployment** (AWS SAM)
```bash
# Deploy serverless infrastructure
cd cloud
sam build
sam deploy --guided

# Configure IoT Core certificates
aws iot create-keys-and-certificate --set-as-active
```

### **Frontend Deployment**
```bash
cd frontend
npm install
npm run build
# Deploy to S3 + CloudFront or Vercel
```

---

## 📁 Kiro Spec-Driven Development (Judges' Guide)

This project was built using the **Kiro Spec-First workflow** for rigorous, testable development.

### **Documentation Location**: `.kiro/specs/acoustic-pump-monitor/`

| File | Description | Lines |
|------|-------------|-------|
| **requirements.md** | 16 testable requirements in EARS notation | 350+ |
| **design.md** | Complete architecture with 44 correctness properties | 950+ |
| **tasks.md** | 28-task implementation roadmap with property-based tests | 400+ |

### **Key Highlights:**
- ✅ **16 Requirements** covering edge AI, cloud reasoning, fairness, and public impact
- ✅ **44 Correctness Properties** for property-based testing (using Hypothesis, RapidCheck, fast-check)
- ✅ **28 Implementation Tasks** across 4 parallel tracks (Edge, Cloud, Frontend, Integration)
- ✅ **Community Fairness** integrated into every layer (Requirement 16)

**Example Property Test:**
```python
@given(
    usage_distribution=st.lists(st.floats(min_value=0, max_value=100), min_size=10),
    threshold=st.floats(min_value=0.3, max_value=0.5)
)
def test_fairness_intervention_trigger(usage_distribution, threshold):
    """
    Property: For any village where top 20% farmers use >40% of water,
    the system should trigger a fairness intervention alert.
    """
    top_20_percent_usage = sum(sorted(usage_distribution, reverse=True)[:2])
    total_usage = sum(usage_distribution)
    
    if top_20_percent_usage / total_usage > threshold:
        assert should_trigger_fairness_alert(usage_distribution) == True
```

---

## 🎯 Differentiation: Why Jal Dhwani?

| Feature | Jal Dhwani | Soil Moisture Sensors | Manual Monitoring |
|---------|------------|----------------------|-------------------|
| **Cost** | ₹700 | ₹5,000+ | Free (but labor-intensive) |
| **Installation** | 10 minutes, non-invasive | Requires burial, fragile | N/A |
| **Community Fairness** | ✅ AI-driven equity optimization | ❌ Individual-only | ❌ No visibility |
| **Offline Safety** | ✅ Edge AI works without internet | ❌ Cloud-dependent | ✅ Manual control |
| **Voice Accessibility** | ✅ Local language calls | ❌ App-only | ❌ No notifications |
| **Public Impact Tracking** | ✅ Village-wide metrics | ❌ Individual metrics | ❌ No data |

---

## 🌍 Social Impact & Scalability

### **Target Deployment:**
- **Primary**: 100,000+ smallholder farmers in groundwater-stressed regions (Rajasthan, Maharashtra, Karnataka)
- **Secondary**: FPOs and Panchayats managing 500+ villages

### **Expected Outcomes:**
- **30% reduction** in groundwater over-extraction through AI scheduling
- **50% improvement** in usage equity (measured by Gini coefficient)
- **₹10,000+ savings** per farmer annually (reduced motor burnout, optimized irrigation)
- **Community empowerment** through transparent fairness metrics

### **Scalability:**
- **Low-cost hardware** (₹700 BOM) enables mass adoption
- **Retrofit design** works with existing pumps (no replacement needed)
- **Serverless cloud** scales automatically with village growth
- **Open-source** for community contributions and localization

---

## 🤝 Team & Acknowledgments

**Built for**: AI for Bharat Hackathon 2026  
**Track**: AI for Communities, Access & Public Impact

**Technology Partners:**
- AWS (IoT Core, Lambda, Bedrock, Polly, DynamoDB)
- Espressif (ESP32-S3 microcontroller)
- OpenWeatherMap (weather forecasting API)

**Special Thanks:**
- Rural farmers in Maharashtra who provided feedback on voice notifications
- FPO administrators who validated fairness metrics
- Kiro AI for spec-driven development methodology

---

## 📜 License

MIT License - See [LICENSE](LICENSE) for details.

---

## 📞 Contact & Demo

**Live Demo**: [Coming Soon]  
**Documentation**: `.kiro/specs/acoustic-pump-monitor/`  
**Issues**: [GitHub Issues](https://github.com/your-repo/issues)

**For Judges**: Please review our comprehensive spec documentation in `.kiro/specs/acoustic-pump-monitor/` for detailed architecture, requirements, and testing strategy.

---

**Built with ❤️ for equitable water access in rural India**
