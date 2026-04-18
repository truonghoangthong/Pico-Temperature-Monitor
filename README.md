# Pico Temperature Monitor

An end-to-end IoT pipeline that reads temperature data from a **Raspberry Pi Pico W** (DHT11 sensor), sends it to a **Node.js REST API**, stores it in **InfluxDB**, and visualizes it through a simple **web dashboard**.

---

## Architecture

```
[Raspberry Pi Pico W]
    DHT11 Sensor
        │
        │  HTTP GET /api/v1/embed?value=<temp>
        ▼
[Node.js / Express API]        ← Hosted on Azure VM (20.123.52.71)
    server.mjs
        │
        │  Write Point
        ▼
[InfluxDB]
    Bucket: temperature data
        │
        │  Flux Query
        ▼
[Web Dashboard]
    index.html  →  GET /api/v1/get-data  →  renders table
```

---

## Project Structure

```
pico-temp-monitor/
├── src/
│   ├── server.mjs       # Express API server
│   └── envs.mjs         # Environment variable loader & validation
├── index.html           # Web dashboard (fetches & displays temperature table)
├── temperature.py       # MicroPython script running on Raspberry Pi Pico W
├── package.json
└── README.md
```

---

## Prerequisites

- **Node.js** v18+
- **InfluxDB** v2.x (cloud or self-hosted)
- **Raspberry Pi Pico W** with MicroPython firmware
- DHT11 temperature sensor wired to **GPIO Pin 0**

---

## Getting Started

### 1. Clone the repository

```bash
git clone https://github.com/<your-username>/pico-temp-monitor.git
cd pico-temp-monitor
```

### 2. Install dependencies

```bash
npm install
```

### 3. Configure environment variables

Create a `.env` file in the root directory:

```env
HOST=0.0.0.0
PORT=3000

DB_INFLUX_HOST=http://localhost:8086
DB_INFLUX_ORG=your_org
DB_INFLUX_BUCKET=your_bucket
DB_INFLUX_TOKEN=your_token
```

| Variable          | Description                          | Required |
|-------------------|--------------------------------------|----------|
| `HOST`            | Server bind address                  | ✅       |
| `PORT`            | Server port (1024–65535)             | ✅       |
| `DB_INFLUX_HOST`  | InfluxDB URL                         | ✅       |
| `DB_INFLUX_ORG`   | InfluxDB organization name           | ✅       |
| `DB_INFLUX_BUCKET`| InfluxDB bucket name                 | ✅       |
| `DB_INFLUX_TOKEN` | InfluxDB API token                   | ✅       |

### 4. Start the server

```bash
npm start
```

Server will be running at `http://<HOST>:<PORT>`.

---

## API Endpoints

| Method | Endpoint              | Description                              |
|--------|-----------------------|------------------------------------------|
| `GET`  | `/`                   | Health check — returns `OK`              |
| `GET`  | `/api/v1/`            | InfluxDB connection check — returns 200  |
| `GET`  | `/api/v1/embed?value=<number>` | Write a temperature value to InfluxDB |
| `GET`  | `/api/v1/get-data`    | Fetch all temperature records (last 30 days) |

### Example

```bash
# Write a temperature value
curl "http://localhost:3000/api/v1/embed?value=24.5"

# Fetch stored data
curl "http://localhost:3000/api/v1/get-data"
```

---

## Pico W Setup (MicroPython)

### Hardware

- Raspberry Pi Pico W
- DHT11 sensor → Data pin connected to **GPIO 0**

### Steps

1. Flash MicroPython firmware onto the Pico W.
2. Open `temperature.py` and update your Wi-Fi credentials:

```python
SSID = "YourWiFiName"
PASSWORD = "YourPassword"
```

3. Update `BASE_URL` to point to your deployed API:

```python
BASE_URL = "http://<your-server-ip>/api/v1"
```

4. Upload `temperature.py` to the Pico W using **Thonny** or **mpremote**.
5. The Pico W will connect to Wi-Fi, read the temperature every **10 seconds**, and POST it to the API.

---

## Web Dashboard

Open `index.html` in a browser (or serve it statically). It fetches data from the API and renders a table with timestamps and temperature readings.

> **Note:** Update the `fetch` URL inside `index.html` if your API server address changes.

```js
fetch('http://<your-server-ip>/api/v1/get-data')
```

---

## Tech Stack

| Layer       | Technology                        |
|-------------|-----------------------------------|
| IoT Device  | Raspberry Pi Pico W + MicroPython |
| Sensor      | DHT11 (temperature & humidity)    |
| Backend API | Node.js + Express                 |
| Database    | InfluxDB v2 (time-series)         |
| Frontend    | Vanilla HTML/CSS/JS               |
| Hosting     | Azure VM                          |

---

## License

ISC
