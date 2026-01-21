# Safe Route Recommender

MVP for a routing engine that prioritizes safety based on crime, lighting, and traffic data.

## Project Structure
- `backend/`: FastAPI application
- `etl/`: Data ingestion scripts
- `engine/`: Safety scoring logic
- `mobile/`: React Native app stub
- `docker-compose.yml`: Infrastructure (PostGIS, OSRM)

## Setup
3.  Run the API: `uvicorn backend.main:app --reload`.

## 🧠 Recommendation Logic
The core "Safe Route" algorithm works in three steps:

1.  **Route Generation**: We query the **OSRM (Open Source Routing Machine)** public API to get multiple driving/walking paths between the Start and End points.
2.  **Spatial Sampling**: We sample points along each route (Start, Middle, End, and intermediate segments) to analyze the neighborhoods passed through.
3.  **Safety Scoring**:
    Each point is scored against our **Crime Database** (loaded in-memory):
    - **Base Score**: Starts at 100.
    - **Crime Penalty**: We check for crimes within a **500m radius**.
        - Weighted by Severity: *Murder (10x)*, *Robbery (8x)*, *Theft (3x)*.
        - Formula: `Penalty = Σ (Crime_Count * Severity_Weight)`
    - **Lighting Bonus**: (Future Scope) +2 points for well-lit streets.
    - **Final Score**: `MAX(0, 100 - Penalty)`.
    
    The route with the highest average score is flagged as **✅ Safest**.

## 📊 Data Sources
- **Crime Data**: Real **NCRB 2022 District-wise Crime Data** for New Delhi.
    - We map district-level aggregate statistics to coordinate clusters to simulate "Hotspots" for this MVP.
- **Maps**: OpenStreetMap (OSM) via Leaflet.js.
- **Routing**: OSRM Demo API.

## 🏗 Architecture
**Current State (Serverless MVP)**:
- **Backend**: FastAPI (Python) running on Vercel/Lambda.
- **Database**: In-Memory Pandas DataFrame (Reads standard CSV).
- **Frontend**: Lightweight HTML/JS (Leaflet).

**Legacy / Alternative**:
- The project includes a `docker-compose.yml` for running a full **PostgreSQL + PostGIS** database and a local **Dockerized OSRM** instance for offline use or high-performance production environments.
