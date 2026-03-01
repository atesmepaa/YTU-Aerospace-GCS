# YTU-Aerospace-GCS
A modern, high-performance Ground Control Station (GCS) application designed for Unmanned Aerial Vehicles (UAVs). Developed with Python and CustomTkinter, this tool provides real-time telemetry, mission planning, and sensor monitoring.

## 🚀 Key Features

* **Real-Time Telemetry:** Monitor altitude, speed, battery status, GPS coordinates, and yaw orientation instantly.
* **Mission Planning:** Intuitive map interface to set waypoints and upload autonomous missions (Task 1 & Task 2).
* **Autonomous Scanning:** Includes an intelligent algorithm to generate zigzag paths for area coverage between two waypoints.
* **Live Video Feed:** Low-latency MJPEG stream support via RPi integration.
* **Modular Communication:** Reliable JSON-based communication protocol via SiK Telemetry Radio.

## 🏗️ Architecture

The project follows a modular design pattern to ensure scalability and maintainability.

* **UI:** Built using `customtkinter` for a modern, dark-themed experience.
* **Hardware Interface:** `pyserial` for communication with Pixhawk/Flight controllers.
* **Logic:** Custom math modules for GPS projections and flight path optimization.

## 📥 Installation

1. **Clone the repository:**
   ```bash
   git clone [https://github.com/yourusername/ytu-macka-gcs.git](https://github.com/yourusername/ytu-macka-gcs.git)
   cd ytu-macka-gcs
