# 🏠 Hierarchical Multi-Agent AI Heating Control (Home Assistant)

An experimental, physics-aware **Building Management System (BMS)** built on top of Home Assistant. Instead of relying on static schedules or basic thermostats, this system combines real-time **PID valve control**, **empirically measured thermal lag models**, and a **hierarchical multi-agent AI supervisor** that dynamically tunes PID parameters and coordinates cross-zone heating.

---

## 🧠 System Architecture (Block Diagram)

```mermaid
flowchart TD
    subgraph SENSORS ["1. Telemetry & Context Layer"]
        S1["🌡️ Primary & Secondary Room Sensors<br/>(Living Room, Hallway, Bedroom, Office)"]
        S2["🔧 Radiator Valve Body Temps<br/>(Hydronic Loop Feedback)"]
        S3["🌤️ Outdoor Temp, Solar Forecast & Indoor Power<br/>(Heat Gain & Weather Compensation)"]
        S4["📊 4h / 24h Rolling CSV Telemetry<br/>(Swing, Delta, PID Avg, Integral)"]
    end

    subgraph SUPERVISOR ["2. Orchestration Layer (AI House Supervisor)"]
        SUP["🎩 AI House Supervisor<br/>(Evaluates 4h summary every cycle)"]
        PHYS["📐 Empirical Physics & Dead-Time Model<br/>• Steady-state lag: LR/Hall 5–10m | Bed 10–15m | Office 20–25m<br/>• Cold-start multiplier: 1.75–2.0x (cold pipe / ΔT ≥ 0.4°C)"]
        BOOST["🔥 Cross-Zone Boost Logic<br/>(Uses central Hallway/LR to assist adjacent rooms)"]
    end

    subgraph GUARDS ["3. Zone Agent Layer (Event-Driven Awakening)"]
        G1["🛋️ Living Room Guard<br/>(3-sensor weighted zone)"]
        G2["🚪 Hallway Guard<br/>(Central distribution hub)"]
        G3["🛏️ Bedroom Guard<br/>(Night-quiet & slow-ramp profile)"]
        G4["🖥️ Office Guard (Local Ollama)<br/>(Long-loop & PC heat-gain aware)"]
    end

    subgraph CONTROL ["4. Real-Time Actuation Layer"]
        PID["⚙️ Zone PID Controllers<br/>(Kp, Ki, Kd, I-Memory, Dynamic Deadband)"]
        TRV["🚰 Hydronic Radiator Valves (TRVs)<br/>(0–100% Continuous Positioning)"]
    end

    S1 & S2 & S3 --> S4
    S4 --> SUP
    PHYS -.-> SUP
    PHYS -.-> G1 & G2 & G3 & G4

    SUP -- "Wakes ONLY zones with<br/>deviation / swing > threshold" --> G1 & G2 & G3 & G4
    SUP -- "Activates temporary<br/>neighbor boost" --> BOOST
    BOOST --> PID

    G1 & G2 & G3 & G4 -- "Tunes Kp / Ki / I-mem / Deadband<br/>or holds steady" --> PID
    S1 & S2 -- "Real-time feedback" --> PID
    PID --> TRV
    TRV -. "Hydronic heat transfer<br/>(with zone dead-time)" .-> S1 & S2
```

---

## ✨ Key Architectural Highlights

* **Event-Driven Multi-Agent Orchestration:** The **AI House Supervisor** acts as an orchestrator. Rather than running expensive LLM inference on every room continuously, it evaluates a 4-hour rolling telemetry digest and wakes **only** the specific Zone Guards experiencing target drift, high temperature swing, or low PID efficiency.
* **Empirical Hydronic Dead-Time & Cold-Start Model:** Every AI agent shares a measured physical model of the building's hydronic loop. Agents distinguish between normal thermal lag (e.g., *20–25 min* in the Office vs. *5–10 min* in the Living Room) and true controller underperformance—applying a **1.75–2.0× cold-start multiplier** when radiator bodies are cold or after large setpoint steps ($\ge 0.4\ ^\circ\text{C}$).
* **Self-Tuning PID Loops:** Instead of directly overriding valve positions, Zone Guards act as control-loop engineers—making bounded micro-adjustments to proportional gain ($K_p$), integral memory ($I$-mem), and dynamic deadband thresholds based on historical CSV trends.
* **Floorplan-Aware Cross-Zone Boosting:** Because the central hallway connects the living room, bedroom, and office, the Supervisor can temporarily boost adjacent zones to assist a room facing high heat loss without causing overshoot.
* **Hybrid Cloud + Local LLM Execution:** Combines cloud reasoning for house-wide orchestration with local **Ollama** inference for dedicated zone control.

---

## 📈 Measured Control Performance

* **Steady-State Precision:** Maintains living areas within **$\pm 0.02\text{–}0.20\ ^\circ\text{C}$** of target setpoint without valve hunting or thermal overshoot.
* **Zero False Interventions:** Dead-time awareness prevents agents from over-tuning $K_p$/$K_i$ while heat is already propagating through the hydronic loop.
