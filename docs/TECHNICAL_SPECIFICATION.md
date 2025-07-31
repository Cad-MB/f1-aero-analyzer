# TECHNICAL SPECIFICATION

**Project:** F1 Aero Performance Analyzer - F1 2026 Data Analysis Platform  
**Type:** Personal Development Project / Technical Portfolio  
**Objective:** Master data analysis technologies in the context of new F1 2026 regulations  
**Estimated Duration:** 12-18 months  
**Start Date:** August 2025

---

## 1. PROJECT CONTEXT

### 1.1 Learning Objectives
Develop a comprehensive platform demonstrating mastery of technologies used in F1 performance analysis, based on the new 2026 regulations introducing:
- Active aerodynamics (X-mode/Z-mode)
- 50/50 hybrid propulsion (400kW thermal + 350kW electric)
- Enhanced energy recovery (8.5 MJ/lap)
- Complex telemetry (400+ channels)

### 1.2 Purpose
Create a demonstrable technical portfolio for motorsport/high-tech sector recruiters, proving the ability to manage complex real-time data systems.

---

## 2. FUNCTIONAL SPECIFICATIONS

### 2.1 Data Sources

**Public APIs to Integrate:**
- **FastF1**: Official F1 telemetry data
- **Ergast API**: Historical race/results data
- **OpenF1 API**: Real-time telemetry during races
- **OpenWeatherMap**: Circuit weather conditions

**Data to Simulate/Generate:**
- X/Z-mode aerodynamic transitions
- Hybrid ICE/ERS energy flow
- Non-public parameters (pressures, internal temperatures)

### 2.2 Functional Modules

#### MODULE 1: Real-Time Ingestion and Processing

**Features:**
```python
# Data pipeline
- WebSocket connection to live APIs
- F1 data parsing (specific format)
- Enrichment with simulated 2026 data
- Optimized time-series storage
```

**Technical Specifications:**
- Frequency: 10-20 Hz (public API limit)
- Target latency: <100ms
- Volume: ~500k points/session

**Implementation Example:**
```python
class TelemetryProcessor:
    def __init__(self):
        self.kafka_producer = KafkaProducer(
            bootstrap_servers=['localhost:9092'],
            value_serializer=lambda v: json.dumps(v).encode('utf-8')
        )
    
    async def process_live_data(self, raw_telemetry):
        # Enrich with simulated 2026 data
        enhanced_data = self.simulate_2026_parameters(raw_telemetry)
        
        # Publish to Kafka for parallel processing
        self.kafka_producer.send('telemetry-topic', enhanced_data)
        
        # Time-series storage
        await self.influx_client.write(enhanced_data)
```

#### MODULE 2: Active Aerodynamics Simulation

**Objective:** Simulate behavior of new 2026 aero systems

**Simplified Physics Model:**
```python
class AeroSimulator:
    """
    Simulates X (low drag) and Z (high downforce) modes
    based on 2026 specs
    """
    
    def calculate_aero_forces(self, speed, mode, ride_height):
        if mode == 'X':
            # Straight line mode - 55% drag reduction
            drag_coeff = 0.45 * self.base_drag
            downforce_coeff = 0.3 * self.base_downforce
        else:  # Z-mode
            drag_coeff = self.base_drag
            downforce_coeff = self.base_downforce
        
        # Simplified aero forces
        dynamic_pressure = 0.5 * AIR_DENSITY * speed**2
        drag = drag_coeff * dynamic_pressure * FRONTAL_AREA
        downforce = downforce_coeff * dynamic_pressure * WING_AREA
        
        return {
            'drag_n': drag,
            'downforce_n': downforce,
            'efficiency': downforce / drag,
            'mode': mode
        }
```

**3D Visualization:**
- Three.js aero flow rendering
- Animated X/Z mode transitions
- Real-time force vectors

#### MODULE 3: Hybrid Energy Management

**2026 System Simulation:**
```python
class HybridEnergyManager:
    def __init__(self):
        self.max_ice_power = 400  # kW
        self.max_mgu_power = 350  # kW
        self.battery_capacity = 4  # MJ
        self.max_recovery = 8.5  # MJ/lap
        
    def optimize_deployment(self, track_position, battery_soc):
        """
        Optimal ICE/ERS deployment algorithm
        """
        if self.is_braking_zone(track_position):
            # Maximum recovery
            return {
                'ice_power': 0,
                'mgu_mode': 'recovery',
                'power_kw': -min(350, self.available_recovery())
            }
        elif self.is_acceleration_zone(track_position):
            # Optimal deployment
            ice = self.max_ice_power * 0.85
            ers = min(
                self.max_mgu_power * 0.9,
                self.battery_to_power(battery_soc)
            )
            return {
                'ice_power': ice,
                'mgu_mode': 'deploy',
                'power_kw': ice + ers
            }
```

#### MODULE 4: Artificial Intelligence & Predictions

**ML for Performance Analysis:**

```python
class PerformancePredictor:
    def __init__(self):
        self.models = {
            'laptime': self.load_model('lstm_laptime_predictor.h5'),
            'tire_deg': self.load_model('xgboost_tire_model.pkl'),
            'overtake': self.load_model('rf_overtake_probability.pkl')
        }
    
    def predict_laptime(self, current_state):
        features = self.extract_features(current_state)
        # LSTM for time series
        return self.models['laptime'].predict(features)
    
    def calculate_overtake_probability(self, attacker, defender):
        """
        Overtaking probability with new 2026 system
        Manual Override Mode included
        """
        features = np.array([
            attacker.speed - defender.speed,
            attacker.battery_soc - defender.battery_soc,
            attacker.manual_override_available,
            self.track_characteristics[attacker.position],
            attacker.tire_age - defender.tire_age
        ])
        
        return self.models['overtake'].predict_proba(features)[0][1]
```

**Model Training:**
- Dataset: 2 seasons of F1 data (2023-2024)
- Feature engineering for 2026 adaptation
- Cross-validation on historical races

#### MODULE 5: Interactive Visualization

**Main Dashboard:**

```javascript
// React + Three.js Architecture
const F1Dashboard = () => {
  return (
    <Grid container spacing={2}>
      {/* 3D Circuit View */}
      <Grid item xs={12} md={8}>
        <Canvas>
          <TrackVisualization 
            cars={realtimeData.cars}
            showAeroMode={true}
            showEnergyFlow={true}
          />
        </Canvas>
      </Grid>
      
      {/* Real-time Metrics */}
      <Grid item xs={12} md={4}>
        <AeroModeIndicator mode={currentCar.aeroMode} />
        <EnergyFlowChart 
          ice={currentCar.icePower}
          ers={currentCar.ersPower}
          recovery={currentCar.ersRecovery}
        />
        <BatteryStatus soc={currentCar.batterySoc} />
      </Grid>
      
      {/* AI Predictions */}
      <Grid item xs={12}>
        <PredictionsPanel 
          laptime={predictions.laptime}
          strategy={predictions.optimalStrategy}
          overtaking={predictions.overtakeOpportunities}
        />
      </Grid>
    </Grid>
  );
};
```

**Visual Features:**
- Race replay with enriched data
- Multi-driver comparison mode
- What-if scenarios (strategy changes)
- Video export for presentation

---

## 3. TECHNICAL ARCHITECTURE

### 3.1 Technology Stack

**Backend:**
```yaml
Core:
  - Python 3.11+
  - FastAPI (REST/WebSocket API)
  - Celery (async tasks)

Data Pipeline:
  - Apache Kafka (streaming)
  - InfluxDB (time-series)
  - PostgreSQL (metadata)
  - Redis (cache/pubsub)

ML/Analytics:
  - TensorFlow/Keras (deep learning)
  - Scikit-learn (classical ML)
  - XGBoost (predictions)
  - Pandas/NumPy (processing)
```

**Frontend:**
```json
{
  "dependencies": {
    "react": "^18.2.0",
    "typescript": "^5.0.0",
    "three": "^0.160.0",
    "@react-three/fiber": "^8.0.0",
    "@react-three/drei": "^9.0.0",
    "d3": "^7.0.0",
    "recharts": "^2.8.0",
    "@reduxjs/toolkit": "^2.0.0",
    "socket.io-client": "^4.5.0"
  }
}
```

**Infrastructure:**
```yaml
Development:
  - Docker Compose (all services)
  - LocalStack (local AWS)

Production (Demo):
  - Kubernetes (K3s for simplicity)
  - GitHub Actions (CI/CD)
  - Prometheus + Grafana (monitoring)
```

### 3.2 System Architecture

```
┌─────────────────────────────────────────────────┐
│             Frontend (React/Three.js)            │
│  - Interactive dashboard                         │
│  - 3D circuit visualization                      │
│  - Real-time charts                              │
└────────────────────┬────────────────────────────┘
                     │ WebSocket/REST
┌────────────────────▼────────────────────────────┐
│            API Gateway (FastAPI)                 │
│  - Authentication                                │
│  - Rate limiting                                 │
│  - WebSocket handler                             │
└────────────────────┬────────────────────────────┘
                     │
┌────────────────────▼────────────────────────────┐
│          Services Layer (Python)                 │
│  ┌─────────────┐ ┌─────────────┐ ┌───────────┐│
│  │  Telemetry  │ │    Aero     │ │    ML     ││
│  │  Processor  │ │  Simulator  │ │ Predictor ││
│  └─────────────┘ └─────────────┘ └───────────┘│
└────────────────────┬────────────────────────────┘
                     │
┌────────────────────▼────────────────────────────┐
│            Message Queue (Kafka)                 │
└────────────────────┬────────────────────────────┘
                     │
┌────────────────────▼────────────────────────────┐
│             Data Layer                           │
│  ┌──────────┐ ┌───────────┐ ┌────────────────┐│
│  │InfluxDB  │ │PostgreSQL │ │Redis Cache     ││
│  │(TS Data) │ │(Metadata) │ │(Live State)    ││
│  └──────────┘ └───────────┘ └────────────────┘│
└─────────────────────────────────────────────────┘
```

---

## 4. DEVELOPMENT PLAN

### Phase 1: Foundations (Months 1-2)
- [x] Dev environment setup (Docker Compose)
- [ ] Basic data pipeline (FastF1 → Kafka → InfluxDB)
- [ ] Basic REST API
- [ ] First React dashboard

### Phase 2: Core Features (Months 3-5)
- [ ] 2026 aero simulator
- [ ] Hybrid energy model
- [ ] Live data integration (OpenF1)
- [ ] Three.js circuit visualization

### Phase 3: Artificial Intelligence (Months 6-8)
- [ ] Dataset preparation (2 years F1 data)
- [ ] Predictive model training
- [ ] Real-time predictions API
- [ ] Strategy optimizer

### Phase 4: Advanced Visualization (Months 9-11)
- [ ] Complete multi-view dashboard
- [ ] Replay mode with enrichment
- [ ] Inter-driver comparisons
- [ ] Export/share analyses

### Phase 5: Finalization (Month 12)
- [ ] Load testing
- [ ] Complete documentation
- [ ] Demo videos
- [ ] Cloud deployment (demo)

---

## 5. DELIVERABLES

### 5.1 Source Code
- Public GitHub repository
- Detailed README
- API documentation (OpenAPI/Swagger)
- Unit/integration tests

### 5.2 Demonstrations
1. **Live Demo**: Real-time analysis during a GP
2. **Tech Video**: Architecture and technical choices (10 min)
3. **Case Studies**: Specific race analyses
4. **Blog Posts**: Technical articles on challenges solved

### 5.3 Success Metrics
- Prediction accuracy: >85%
- Processing latency: <100ms
- Demo uptime: >99%
- Test coverage: >80%

---

## 6. SKILLS DEVELOPED

### Technical
- **Data Engineering**: High-frequency streaming, time-series
- **Machine Learning**: Prediction, optimization, RL basics
- **3D Visualization**: Three.js, WebGL, complex animations
- **Cloud Native**: Kubernetes, microservices, observability
- **Domain Knowledge**: Aerodynamics, hybrid systems, telemetry

### Soft Skills
- Long-term complex project management
- Technical documentation
- Technical solution presentation
- Autonomous learning in new domain

---

## 7. REQUIRED RESOURCES

### Hardware
- Development PC (32GB RAM minimum)
- GPU for ML (optional, can use Colab)

### Cloud Services (Budget)
- AWS/GCP free tier sufficient for dev
- GitHub (free)
- Potential VPS for live demo (~€20/month)

### Time
- 15-20h/week
- Total: ~700-800h over 12 months

---

## APPENDICES

**A1.** List of public F1 APIs and data formats  
**A2.** 2026 technical regulations specifications  
**A3.** Simplified aerodynamic mathematical models  
**A4.** Public datasets for ML training  

---

This project will demonstrate complete mastery of the modern data stack applied to a complex and exciting domain, perfect for a high-performance tech-oriented portfolio.