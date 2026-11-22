# Privacy Policy & Data Processing Notice
**System:** Home Assistant Hydronic Heating & Multi-Agent AI Control System  
**Last Updated:** November 22, 2026  

---

## 1. Purpose and Scope of This Notice
This document outlines the telemetry data collected, processed, and stored by the automated residential heating control system in accordance with the principles of the General Data Protection Regulation (**EU GDPR 2016/679**), specifically **Article 5 (Data Minimisation, Purpose Limitation, Storage Limitation)** and **Article 25 (Data Protection by Design and by Default)**.

### Applicability of GDPR (Personal Data Assessment)
This system operates strictly within a private residential environment for the sole purpose of technical building automation and thermal comfort optimization. **No directly identifiable personal data** (such as names, personal identification numbers, facial images, audio recordings, exact GPS coordinates of individuals, or personal device MAC addresses) is collected or processed. 

However, because high-resolution indoor environmental telemetry (e.g., window opening events or minor thermal rises attributed to general occupancy) could theoretically be categorized as indirect household behavioral metadata under broad GDPR interpretations, this notice documents all data flows transparently.

---

## 2. Categories of Data and Sensors Used
The system collects purely physical, environmental, and HVAC-related telemetry. No cameras, microphones, or personal tracking beacons are integrated into the heating logic.

| Category | Specific Sensors & Entities | Data Points Collected | Purpose of Processing |
| :--- | :--- | :--- | :--- |
| **Indoor Air Temperature** | IKEA Vindstyrka / Vallhorn / Ikea Zigbee sensors (Living Room x3, Hallway, Workroom), Dallas 1-Wire , RuuviTag  | Ambient air temperature (`°C`) | Calculating zone averages, PID feedback loops, and thermal stability (`±0.10 °C`). |
| **Window State** | Zigbee Contact Sensor  | Binary state (`open` / `closed`) | Automatically closing the bedroom radiator valve during ventilation and enforcing a 30-minute post-ventilation settling pause. |
| **Radiator & Valve Telemetry** | Shelly BLU TRV actuators (Bedroom, Living Room x3, Hallway, Workroom) | Valve opening (`%`), valve body temperature (`°C`), battery level (`%`), calibration status | Executing PID valve control, detecting mechanical valve anomalies, and estimating thermal lag. |
| **Boiler & Hydronic Loop** | Central loop sensors (AS2 supply/return) | Supply and return water temperatures (`°C`) | Monitoring central boiler room heat delivery (`Boiler Elf Durin`). |
| **PID & Control State** | Internal `input_number` and `input_text` helpers | Setpoints (`°C`), PID gains (`Kp`, `Ki`, `Kd`), I-memory (`%`), system status logs | Maintaining control loop memory and inter-agent status heartbeats. |
| **Outdoor & Forecast Data** | Local patio outdoor sensor , `weather.forecast_home`, solar production forecast (`sensor.power_production_next_hour`) | Outdoor temperature (`°C`), wind speed (`m/s`), solar radiation estimate (`W` / `Wh`) | Weather feedforward compensation and preventing overheating during solar gain. |

*Note on Indirect Occupancy Inference:* System prompts include general physical assumptions (e.g., approximate metabolic heat load of ~120–160 W in the bedroom during nighttime hours) solely to prevent PID integral wind-up. No individual presence tracking or identity attribution occurs.

---

## 3. Where and How Data Is Processed
The architecture follows a **local-first, hybrid edge-cloud design** to minimize external data transmission:

### A. Local Edge Processing (On-Premises Server)
* **Location:** Local Home Assistant server hardware located inside the residence (Finland).
* **Components:**
  * **PID Control Loops & Automations:** Executed 100% locally in real time.
  * **Local LLM (`Ollama` via `ai_task.ollama_ai_task`):** Both the Whole House AI Supervisor (*Gandalf*) and individual Room Guards (*Chamber Elf, Hearth Elf, Forge Elf, Corridor Elf*) execute locally on on-premises hardware.
  * **Local History Scripts (`house_4h_digest.py`, `*_ai_data.py`):** Aggregate 4-hour and 26-hour rolling CSV summaries strictly within local `/config/scripts/` storage.
* **External Exposure:** Zero external network traffic is generated during real-time PID execution, local LLM inference, or random status reply generation (`/config/hold_replies.txt`).

### B. Cloud Processing (External Services)
External API calls are strictly limited to anonymized technical telemetry and public meteorological queries:

1. **Google Gemini (Offline 24h PID Profile Analysis):**
   * **Purpose:** Periodic offline optimization of PID tuning parameters (`Kp`, `Ki`, `Kd`) based on 24-hour thermal response curves.
   * **Data Transmitted:** Anonymized time-series numerical telemetry (room temperatures, setpoints, valve percentages, outdoor temperature, solar wattage) using generic fantasy/zone identifiers (e.g., *The Slumber Chamber*, *The Forge*).
   * **Data Excluded:** No street addresses, GPS coordinates, user names, IP identifiers, or personal account metadata are included in the prompt payload.
2. **Weather & Solar Forecast Providers (`weather.forecast_home`):**
   * **Purpose:** Fetching hourly outdoor temperature, wind, and solar irradiance forecasts.
   * **Data Transmitted:** General regional coordinates configured in Home Assistant's weather integration to retrieve localized meteorological forecasts.

---

## 4. Data Storage, Retention, and Security
* **Short-Term Operational Memory:** Rolling 4-hour and 26-hour CSV digests are overwritten continuously in local storage (`/config/scripts/`).
* **Home Assistant Recorder Database:** Standard sensor state history is stored locally according to the Home Assistant instance's configured recorder purge interval.
* **Access Control:** Access to the Home Assistant dashboard, SSH console, and `/config` filesystem is restricted exclusively to the system owner via authenticated credentials.
* **No Third-Party Sharing:** Telemetry data is never sold, shared with advertisers, or transmitted to third parties outside the technical API calls listed in Section 3B.

---

## 5. Summary of GDPR Compliance Principles
* **Lawfulness & Transparency:** Operated exclusively by the household owner for private domestic climate control (GDPR Article 2(2)(c) household exemption applies; documented voluntarily for transparency).
* **Data Minimisation:** Only physical temperature, valve position, and binary window states required for thermodynamic control are processed.
* **Storage Limitation:** Rolling diagnostic buffers automatically expire after 4 to 26 hours.
* **Integrity & Confidentiality:** Real-time control and primary AI inference remain strictly offline on local hardware.
