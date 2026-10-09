# Humidity IoT System (ESP32 + DHT11 + Flask)

An IoT system that reads temperature and humidity from a DHT11 sensor on an ESP32 and sends the values to a Flask REST API every 2 seconds, where other apps can read the latest reading. It was built as an assignment for SIC 5. An alternative MQTT receiver is also included.

## How It Works

```
DHT11 ──► ESP32 (post-data.ino): every 2 s read sensor, get NTP time (UTC+7), HTTP POST
              ▼
   Flask REST API (app.py): keeps the latest reading in memory
              ▼
   GET /api/sensor_data · /api/temperature · /api/humidity
```

During development the ESP32 posted to the local Flask server through an ngrok tunnel.

## Features

- **Firmware:** reads the DHT11 on GPIO 4, skips invalid readings, adds an NTP timestamp, sends a URL-encoded POST, and logs to the serial monitor.
- **REST API (Flask-RESTful):** receive a reading and read the latest temperature, humidity, or full record. CORS enabled.
- **MQTT option:** a Paho client subscribing to `/sensor/data/temperature` and `/sensor/data/humidity` on `test.mosquitto.org`, updating the same in-memory record.
- **Modular structure:** app factory, URL registration, and controllers in separate modules.

## API Endpoints

| Method | Endpoint | Description |
|---|---|---|
| `POST` | `/api/sensor_data` | Receive a reading (form fields `temperature`, `humidity`, `timestamp`) |
| `GET` | `/api/sensor_data` | Latest reading |
| `GET` | `/api/temperature` | Latest temperature |
| `GET` | `/api/humidity` | Latest humidity |

```bash
curl -X POST http://localhost:5000/api/sensor_data -d "temperature=29.5&humidity=70.0&timestamp=14:05:12"
```

## Hardware

ESP32 board, DHT11 sensor (data pin on GPIO 4), breadboard, jumper wires, USB power.

![IoT circuit](media/Rangkaian-IoT.jpg)

## Tech Stack

Arduino (C++) with `WiFi`, `HTTPClient`, `DHT`, `NTPClient`; Python, Flask 3, Flask-RESTful, Flask-CORS; Paho MQTT; ngrok.

## Project Structure

```
humadity_IoT-main/
├── post-data.ino            # ESP32 firmware
├── app.py                   # Entry point (REST API; MQTT mode commented out)
├── app/                     # App factory, urls.py, path_url/humadity.py, controller/{humadity,mqtt}/main.py
├── media/                   # Circuit photo and POST output screenshot
└── .gitignore               # Ignores virtual environments and caches
```

## Getting Started

```bash
git clone https://github.com/harrymardika/humadity_IoT-main.git
cd humadity_IoT-main
python -m venv .venv && source .venv/bin/activate    # Windows: .venv\Scripts\activate
pip install flask flask-restful flask-cors paho-mqtt
python app.py
```

The API runs at `http://localhost:5000`. To reach it from the ESP32 over the internet, run `ngrok http 5000`.

**Flash the ESP32:** open `post-data.ino` in the Arduino IDE (ESP32 board support), install the Adafruit **DHT sensor library** and **NTPClient**, set your Wi-Fi `ssid`/`password` and `serverName` (your API URL ending in `/api/sensor_data`), upload, and open the serial monitor at 9600 baud.

**MQTT mode:** uncomment the MQTT block in `app.py`, then publish values to the two topics.

Serial output of the POST requests:

![POST output](media/output.jpg)

## Limitations

Only the latest reading is kept in memory (no history or database), CORS allows all origins, and the API has no authentication.

## Author

**Harry Mardika** · [GitHub](https://github.com/harrymardika)
