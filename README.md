# ClassPulse-IoT-Enabled-AI-Powered-Smart-Classroom-Monitoring-and-Analytics-System

ClassPulse combines AI-based face recognition for automatic attendance with IoT sensors for classroom environment monitoring — unified through a single backend, database, and dashboard.

---

## 📖 Overview

Traditional classroom attendance is manual, time-consuming, and easy to manipulate. ClassPulse solves this by:

- **Recognizing students automatically** via webcam using face detection + recognition, and marking attendance in real time.
- **Monitoring the classroom environment** — temperature, humidity, light, noise, and door activity — using IoT sensors on a NodeMCU (ESP8266).
- **Bringing it all together** in a FastAPI backend, a database, and a dashboard that gives teachers live attendance, environment stats, and historical analytics/reports.

The system is built so the AI attendance pipeline works **independently** of the IoT hardware — if a sensor fails during a demo or in production, attendance still works.

---

## 🏗️ System Architecture

```
                     ┌───────────────┐
                     │    TEACHER    │
                     └───────┬───────┘
                             │
                    Start Class Session
                             │
              ┌──────────────┴──────────────┐
              │                             │
              ▼                             ▼
        CAMERA MODULE                  IoT MODULE
              │                             │
              ▼                             ▼
       YuNet Face Detector           NodeMCU (ESP8266)
              │                             │
              ▼                             ▼
       Face Recognition              Sensor Readings
    (face_recognition, 128-D)     (Temp/Humidity/Light/
              │                    Sound/Door)
              ▼                             │
      3 Confirmations                       │
              │                             │
              ▼                             │
     Attendance Marked                      │
              │                             │
              └─────────────┬───────────────┘
                             ▼
                      FastAPI Backend
                             │
                ┌────────────┼────────────┐
                │            │            │
                ▼            ▼            ▼
           Attendance    Students    Sensor Data
                │            │            │
                └────────────┼────────────┘
                             ▼
                          Database
                             │
                             ▼
                         Dashboard
                             │
                ┌────────────┼────────────┐
                │            │            │
                ▼            ▼            ▼
           Attendance   Environment   Analytics
                │            │            │
                └────────────┼────────────┘
                             ▼
                          Reports
```

---

## ✨ Features

### AI / Attendance
- Face **detection** using YuNet (`cv2.FaceDetectorYN`)
- Face **recognition** using 128-D embeddings (`face_recognition`)
- Multi-pose student registration (straight, left, right, up, down) for robust matching
- Confirmation-based marking (requires 3 consecutive recognitions) to prevent false/duplicate attendance
- Period-aware attendance — a built-in class timetable gates when attendance can be marked
- Duplicate-proof CSV/database writes — a student can't be marked present twice for the same period

### IoT / Environment Monitoring
- Temperature & humidity monitoring (KY-015)
- Ambient light level monitoring (KY-018)
- Classroom noise/activity monitoring (KY-038)
- Door open/close → entry/exit event logging (KY-025)
- *(Optional)* Flame sensor for safety monitoring (KY-026)
- *(Optional)* Relay-based automation — e.g. auto-trigger a fan on high temperature (KY-019)
- *(Optional)* OLED live readout, buzzer alerts, and manual session-start button

### Backend / Dashboard
- FastAPI backend unifying attendance, student, and sensor data
- Live dashboard: current period, present/absent counts, environment readings
- Attendance analytics: per-student %, period-wise trends, environment trends
- Daily / weekly / monthly / student / class reports

---

## 🛠️ Tech Stack

| Layer | Technology |
|---|---|
| Face detection | OpenCV `FaceDetectorYN` (YuNet, ONNX model) |
| Face recognition | `face_recognition` (dlib-based, 128-D embeddings) |
| Registration UI | Tkinter + Pillow |
| Backend | FastAPI + Uvicorn |
| Database | SQLite / PostgreSQL |
| IoT controller | NodeMCU (ESP8266 / ESP-12E), Arduino C++ |
| Dashboard | HTML / CSS / JavaScript |
| Language | Python 3.x |

---

## 📁 Project Structure

```
ClassPulse/
│
├── backend/
│   ├── main.py                     # FastAPI app entry point
│   ├── attendance_recognition.py   # Live face recognition + attendance marking
│   ├── student_registration.py     # Tkinter GUI for registering new students
│   ├── face_detection.py           # Standalone YuNet detection test/demo
│   ├── database.py                 # Database models / connection
│   └── attendance.csv              # Attendance log (CSV mode)
│
├── dataset/
│   └── <STUDENT_ID>_<STUDENT_NAME>/
│       ├── face_01.jpg   # Straight
│       ├── face_02.jpg   # Left
│       ├── face_03.jpg   # Right
│       ├── face_04.jpg   # Up
│       ├── face_05.jpg   # Down
│       └── student_info.txt
│
├── models/
│   └── face_detection_yunet_2023mar.onnx
│
├── iot/
│   └── sensor_code/        # NodeMCU (ESP8266) Arduino sketches
│
├── dashboard/
│   ├── index.html
│   ├── style.css
│   └── script.js
│
└── README.md
```

---

## ⚙️ Setup & Installation

### 1. Prerequisites
- Python 3.9+
- A webcam
- (For IoT) NodeMCU ESP8266 board + Arduino IDE
- Windows/macOS/Linux

### 2. Clone the repository
```bash
git clone https://github.com/<your-username>/ClassPulse.git
cd ClassPulse
```

### 3. Create and activate a virtual environment
```bash
python -m venv venv

# Windows
venv\Scripts\activate

# macOS/Linux
source venv/bin/activate
```

### 4. Install dependencies
```bash
pip install opencv-python numpy face_recognition fastapi uvicorn pillow
```

> **Note:** `face_recognition` depends on `dlib`, which can require build tools on some systems (especially Windows). If installation fails, install a prebuilt `dlib` wheel matching your Python version first, then retry.

### 5. Download the YuNet model
Place `face_detection_yunet_2023mar.onnx` inside the `models/` folder. It's available from the [OpenCV Zoo](https://github.com/opencv/opencv_zoo/tree/main/models/face_detection_yunet).

---

## 🚀 Usage

### 1. Register a student
```bash
cd backend
python student_registration.py
```
Enter the student's ID and name, then follow the on-screen prompts to capture 5 face poses (straight, left, right, up, down).

### 2. Run live attendance
```bash
python attendance_recognition.py
```
- Opens the webcam, detects and recognizes registered students.
- Marks attendance automatically once a student is confirmed 3 times, **but only during an active class period** (edit the `TIMETABLE` in `attendance_recognition.py` to match your school's schedule).
- Press **Q** to quit.

### 3. (Optional) Test face detection alone
```bash
python face_detection.py
```

### 4. (Optional) Diagnose a specific registration image
```bash
python test_face_image.py
```
Useful if a student's images fail to register — checks HOG, CNN, and YuNet detection on a single image and reports what each finds.

---

## 🔌 IoT Setup

**Hardware used:** NodeMCU (ESP8266/ESP-12E) — chosen over the Arduino Uno because of its built-in WiFi, letting it POST sensor data directly to the FastAPI backend without a serial relay.

| Sensor | Purpose |
|---|---|
| KY-015 (Temp & Humidity) | Classroom environment comfort |
| KY-018 (Photoresistor) | Room illumination level |
| KY-038 (Sound) | Classroom noise/activity proxy |
| KY-025 (Reed switch) | Door open/close → entry/exit events |

1. Flash the sketch in `iot/sensor_code/` to the NodeMCU using the Arduino IDE.
2. Update the WiFi SSID/password and backend endpoint URL in the sketch.
3. Power the NodeMCU — it will begin sending sensor readings to the backend at a fixed interval.

---

## 🗺️ Roadmap

- [ ] Migrate attendance storage from CSV to a full SQL database
- [ ] Person/mobile-phone/laptop detection (YOLO) for classroom activity analytics
- [ ] Emotion analysis (optional/future)
- [ ] Relay-based automation (e.g. auto-fan on high temperature)
- [ ] OLED live readout on the IoT controller
- [ ] Full analytics dashboard with historical trends

---

## 📊 Sample Attendance Output

| Student ID | Name | Date | Period | Start | End | Marked | Status |
|---|---|---|---|---|---|---|---|
| 101 | Nithish | 13/09/2026 | P1 | 09:30 | 10:20 | 09:30:41 | Present |
| 102 | Rajesh | 13/09/2026 | P1 | 09:30 | 10:20 | 09:31:05 | Present |
| 103 | Sai | 13/09/2026 | P1 | — | — | — | Absent |

---

## 👤 Author

NITHISH DUDDE
Project built as an AI + IoT smart classroom monitoring system.
