# 🚢 Vessel Anomaly Detection System

> **Empowering Maritime Intelligence with Real-Time Data & AI-Powered Surveillance**

**Vessel Anomaly Detection** is a comprehensive maritime tracking and analysis platform designed to provide real-time insights into vessel movements, environmental conditions, and **AI-driven Oil Spill Detection**. By integrating multiple data sources (Satellite, Weather, AIS), it offers a holistic view of maritime activities, aiding in navigation, ecological monitoring, and decision-making.

The application features a modern **React** frontend with glowing interactive cards, a **Node.js** backend for data orchestration, and a **Python/Flask Microservice** for deep-learning-based oil spill analysis.

[![License: ISC](https://img.shields.io/badge/License-ISC-blue.svg)](https://opensource.org/licenses/ISC)
[![React](https://img.shields.io/badge/React-19.x-blue)](https://react.dev/)
[![PyTorch](https://img.shields.io/badge/PyTorch-DeepLabV3+-ee4c2c)](https://pytorch.org/)
[![Hugging Face](https://img.shields.io/badge/Hugging_Face-Spaces-yellow)](https://huggingface.co/)
[![Node.js](https://img.shields.io/badge/Node.js-18+-green)](https://nodejs.org/)
[![MongoDB](https://img.shields.io/badge/MongoDB-Atlas-green)](https://www.mongodb.com/)
[![Tailwind CSS](https://img.shields.io/badge/Tailwind-4.0-38B2AC)](https://tailwindcss.com/)

---

## 🌟 Key Features

### 🖥️ Modern User Interface
*   **Interactive Dashboard**: A sleek, dark-themed dashboard with glowing card effects (Aceternity UI).
*   **Visual Analytics**: Real-time visualization of vessel status, weather, and AI analysis.

### 📍 Core Functionality
*   **Real-Time Vessel Tracking**: Track vessels globally by MMSI or Name (via RapidAPI/MarineTraffic).
*   **🛰️ Satellite Visuals**: Fetch high-res satellite imagery from Mapbox for the vessel's current coordinates.
*   **🌤️ Live Weather**: Real-time environmental data (Wind, Temp, Pressure) from OpenWeatherMap.

### 🧠 AI Oil Spill Detection (New!)
*   **Deep Learning Model**: Uses **DeepLabV3+** (ResNet50 encoder) to segment oil spills from satellite imagery.
*   **Real-Time Inference**: Analyzes satellite images on-the-fly to detect potential anomalies.
*   **Visual Overlay**: Generates a grayscale analysis map with **Red Highlights** for detected spills.
*   **Risk Assessment**: Calculates confidence score and percentage of oil coverage.

### ⚙️ Backend & Performance
*   **Smart Caching (MongoDB)**: Caches vessel data for **24 hours** to minimize API costs and rate limits.
*   **Microservice Architecture**: Decouples the heavy ML inference (Python) from the main application logic (Node.js).

---

## 📂 Folder Structure

```
Vessel-Tracking/
├── frontend/             # React + Vite Frontend
│   ├── src/components/   # UI Components (GlowingEffect, Boxes)
│   └── src/App.jsx       # Main Dashboard UI
├── final-version/        # Node.js Backend (API Gateway)
│   ├── server.js         # Express Server & caching logic
│   └── db.js             # MongoDB Schema & Connection
├── api/                  # Vercel Serverless Function Entrypoint
│   └── index.js          # Bridges Vercel to Express
├── Model/                # ML Development Environment
│   ├── api.py            # Local Flask Interface for Model
│   └── deeplabv3p_best.pth # Trained Model Weights
├── hf_space/             # Deployment Bundle (All-in-One)
│   ├── app.py            # Combined Python Backend (ML + Logic)
│   ├── Dockerfile        # Container config for Hugging Face
│   ├── static/           # Built Frontend Assets
│   └── templates/        # Frontend HTML
├── package.json          # Root configuration for Vercel
└── vercel.json           # Vercel Configuration (Alternative Deployment)
```

---

## 🚀 Deployment Options

You can deploy this project in two ways:

### Option 1: Hugging Face Spaces (All-in-One) 🌟 *Recommended*
Host the **Entire Application** (Frontend + Backend + ML) in a single Docker container.

1.  **Create a Space**: Go to Hugging Face and create a new Space with the **Docker** SDK.
2.  **Upload Files**: Upload the contents of the `hf_space/` directory.
    *   *Important*: Ensure `deeplabv3p_best.pth` is included.
3.  **Environment Variables**: Set the following secrets in your Space settings:
    *   `MONGO_URI`
    *   `MAPBOX_ACCESS_TOKEN`
    *   `WEATHER_API_KEY`
    *   `RAPIDAPI_KEY`
4.  **Run**: The Space will build and serve your app on port 7860.

### Option 2: Vercel + Hugging Face (Split)
Host Frontend/Backend on Vercel and ML Service on Hugging Face.

1.  **ML Service**: Deploy `hf_space/` to Hugging Face (Port 7860).
2.  **Frontend/Backend**: Deploy the root repo to **Vercel**.
    *   Set `ML_SERVICE_URL` in Vercel to your Hugging Face Space URL.
    *   Set other API keys in Vercel.

---

## 🛠️ Local Setup

### Prerequisites
*   Node.js (v18+)
*   Python (3.9+)
*   MongoDB Atlas URI
*   API Keys (Mapbox, OpenWeatherMap, RapidAPI)

### 1. Install Dependencies

**Frontend:**
```bash
cd frontend
npm install
```

**Backend:**
```bash
cd final-version
npm install
```

**ML Service (Python):**
```bash
python -m venv venv
source venv/bin/activate
pip install -r hf_space/requirements.txt
```

### 2. Configure Environment
Create a `.env` file in `final-version/` and `Model/` (or root):
```env
MONGO_URI=mongodb+srv://...
WEATHER_API_KEY=...
MAPBOX_ACCESS_TOKEN=...
RAPIDAPI_KEY=...
ML_SERVICE_URL=http://127.0.0.1:5001/predict
```

### 3. Run the System
You need 3 terminals:

1.  **ML Service**: `python Model/api.py` (Runs on port 5001)
2.  **Backend**: `node final-version/server.js` (Runs on port 3000)
3.  **Frontend**: `cd frontend && npm run dev` (Runs on port 5173)

---

## 📡 API Endpoints

### `GET /api/vessel-position`
Returns vessel data + ML analysis.

**Params**: `?name=COMPASS` or `?name=244110352` (MMSI)

**Response**:
```json
{
  "name": "COMPASS",
  "mmsi": "244110352",
  "latitude": 53.259,
  "longitude": 6.497,
  "satelliteImage": "<base64>",
  "oilSpillData": {
    "is_spill": false,
    "confidence": 0.59,
    "analysisImage": "<base64_overlay>"
  }
}
```

---

---

## ❓ Troubleshooting

### 1. `405 Method Not Allowed` on Hugging Face
*   **Cause**: The static file handler is catching the API request.
*   **Fix**: Ensure `app.route('/predict')` is defined **before** `app.route('/<path:path>')` in `app.py`. We have fixed this in V3.

### 2. `MongoDB Connection Error`
*   **Cause**: IP address not whitelisted in MongoDB Atlas.
*   **Fix**: Go to Atlas -> Network Access -> Add IP Address -> Allow Access from Anywhere (0.0.0.0/0) or add the specific Serverless IP.

### 3. Missing API Keys
*   **Symptoms**: Weather showing "N/A" or Satellite image failing.
*   **Fix**: Double-check your `.env` variables or Vercel Environment Variables.

---

## 👤 Author

**Ganesh Arihanth**
*   [GitHub Profile](https://github.com/GaneshArihanth)
*   [Hugging Face Space](https://huggingface.co/spaces/GaneshArihanth/Vessel-Oil-Spill-Detection-API)

---
