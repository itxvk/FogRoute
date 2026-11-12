# 🌫️ FogRoute — Live Fog & Road Safety Platform

**FogRoute** is an intelligent, real-time road safety and visibility advisory platform designed to help motorists and families navigate safely through hazardous dense winter fog.

---

## ✨ Key Features

1. **🗺️ Live Fog Risk Map**
   - Interactive GIS map powered by Leaflet and high-resolution Esri ArcGIS Online tiles.
   - Live visibility tracking across major localities and highway sectors.
   - Real-time heat mapping and color-coded risk alerts (Low, Moderate, High, Severe).

2. **🛣️ Safe Alternate Route Suggestions**
   - Automatically computes visibility conditions between origin and destination.
   - When dense or blinding fog is detected on primary highways, an alternate, safer arterial route, bypass, or elevated corridor is recommended.

3. **📡 Real-Time Data Authenticity**
   - Meteorological data streamed live from **Open-Meteo High-Resolution Forecasting Models** and atmospheric sensors.
   - Live parameters including visibility (meters), relative humidity (%), ambient temperature (°C), and real-time Air Quality Index (AQI).

4. **👥 Community Fog Verification & Reporting**
   - Real-time crowd-sourced fog reports from verified drivers on the road.
   - Dual-action community voting ("Fog still exists" / "Fog cleared") with anti-duplicate vote safeguards.

5. **🆘 Family Safety & Emergency SOS**
   - Family emergency contact configuration stored in the secure backend.
   - One-tap SOS beacon broadcasting live GPS coordinates and local visibility metrics.

---

## 🛠️ Tech Stack & Architecture

- **Frontend**: Pure modern vanilla JavaScript (ES6+), HTML5, CSS3, Leaflet GIS.
- **Backend**: [Supabase](https://supabase.com/) (PostgreSQL 17).
  - `public.fog_reports` — Crowd-sourced incident tracking.
  - `public.report_votes` — Community verification tallies with security-definer RPC functions (`cast_vote`).
  - `public.route_inquiries` — Logs every journey query with environmental telemetry.
  - `public.profiles` — User profile metadata and emergency family contact bindings.
- **Weather Services**: Open-Meteo API & Open-Meteo Air Quality API.

---

## 🚀 Running Locally

1. Clone the repository:
   ```bash
   git clone https://github.com/vishonic/Fog-Route.git
   cd Fog-Route
   ```

2. Start a local HTTP server:
   ```bash
   # Using Python
   python -m http.server 8000
   ```

3. Open in your browser:
   ```
   http://localhost:8000/
   ```

---

## 📄 License
MIT License. Created for public safety and winter highway accident prevention.
