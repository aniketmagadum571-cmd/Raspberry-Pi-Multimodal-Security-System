# Secure Drive: Intelligent Vehicle Protection & Safety System

**Domain:** Embedded Systems, IoT, Computer Vision, Biometric Authentication  
**Platform:** Raspberry Pi 4 / Raspberry Pi OS  
**Academic year:** 2025–2026

Secure Drive is a Raspberry Pi-based vehicle-security prototype intended to reduce unauthorized vehicle access by combining face recognition, fingerprint verification, a voice command, an emergency passcode, GPS location reporting, GSM SMS alerts, and image capture. The system keeps the ignition relay disabled until the required authentication sequence succeeds.

> **Implementation note:** This README follows the submitted project report. The report describes the intended complete system and names the libraries and hardware below. Availability of a feature in the report does not, by itself, prove that every feature is implemented or tested in the Python source. Verify the actual source file, wiring, and installed package versions before deployment.

## Contents

- [Objectives](#objectives)
- [Features and working model](#features-and-working-model)
- [Authentication workflow](#authentication-workflow)
- [Hardware requirements](#hardware-requirements)
- [Software and Python libraries](#software-and-python-libraries)
- [Repository structure](#repository-structure)
- [Setup on Raspberry Pi OS](#setup-on-raspberry-pi-os)
- [Dataset preparation](#dataset-preparation)
- [Configuration checklist](#configuration-checklist)
- [Run the program](#run-the-program)
- [Test plan](#test-plan)
- [Safety and privacy](#safety-and-privacy)
- [Known limitations](#known-limitations)

## Objectives

- Authenticate the driver through multiple independent checks.
- Keep ignition disabled when authentication fails.
- Capture evidence when an unauthorized attempt is detected.
- Send the owner an SMS alert with available GPS coordinates and a map link.
- Provide an emergency passcode route with alerting.
- Use modular embedded software that can be extended with additional vehicle-safety features.

## Features and working model

### 1. Face recognition

The Raspberry Pi camera captures a driver's face. The vision pipeline described in the report uses OpenCV and MTCNN for image handling and face localization, and `face_recognition`/dlib for face encodings and comparison. Authorized face encodings can be saved locally for reuse. A face that does not match the authorized dataset should keep ignition locked and initiate the security-event workflow.

### 2. Fingerprint verification

After face verification, the R307S optical fingerprint sensor is queried over UART. The sensor captures and compares a fingerprint against templates enrolled in its internal memory. A successful match advances the workflow; a mismatch keeps ignition locked and may trigger an alert.

### 3. Voice command

After face and fingerprint checks succeed, a USB microphone captures the user's command. The report describes using `SpeechRecognition` and recognizing the word **“start”** as a command-intent check. This is command recognition, not speaker-identity authentication. The report mentions `recognize_google()`, which requires network access; do not describe this specific recognition path as fully offline.

### 4. Emergency passcode

A keypad or configured emergency-input path accepts a passcode as a controlled fallback. The report describes temporary relay activation after a correct code and alerting on an incorrect code. Confirm the implemented key mapping, passcode policy, retry limits, and timing in the source before using it.

### 5. GPS location and GSM alerts

The NEO-6M GPS module supplies coordinates using NMEA serial sentences. The SIM900A GSM modem sends SMS messages through AT commands. Alerts may contain the event type, coordinates, a Google Maps URL, and an image link when cloud upload is configured. GPS reception and cellular service are required for these functions.

### 6. Intruder image capture and optional Google Drive upload

When authentication fails, the camera can save an evidence image locally. The report describes optional upload through the Google Drive API using OAuth 2.0. Uploading requires valid Google Cloud OAuth configuration and network access. Keep credentials and tokens private.

### 7. Relay, LCD, buzzer, and parallel tasks

The relay acts as an ignition-control output and should remain in its safe OFF state until authorization. The I2C 16×2 LCD displays status messages; a buzzer can provide success/failure feedback. The report describes parallel task handling for camera, keypad, and telematics work so slower GPS/SMS operations do not unnecessarily block authentication.

## Authentication workflow

```text
System boot
   |
Initialize camera, GPIO, UART, LCD and other configured modules
   |
Face recognition
   |-- No match --> Capture evidence --> Read GPS --> Send SMS --> Keep ignition OFF
   |
Face matched
   |
Fingerprint verification
   |-- No match --> Capture evidence --> Read GPS --> Send SMS --> Keep ignition OFF
   |
Fingerprint matched
   |
Listen for the voice command "start"
   |-- Not accepted --> Keep ignition OFF and handle failure/alert
   |
Command accepted
   |
Enable relay according to the configured safe control logic
   |
Return to locked/monitoring state when the authorized interval ends
```

The emergency-passcode path is a separate fallback described in the report. It should be tested independently and should never leave the relay energized after a timeout or software failure.

## Hardware requirements

| Component | Purpose |
|---|---|
| Raspberry Pi 4 | Main controller and local processing |
| Raspberry Pi 5 MP camera | Driver verification and evidence capture |
| R307S fingerprint sensor | Fingerprint template matching |
| USB microphone | Voice command input |
| NEO-6M GPS module | Latitude/longitude acquisition |
| SIM900A GSM module and SIM | SMS notifications |
| 5 V relay module and test motor/load | Ignition-control demonstration |
| 16×2 LCD with I2C backpack | Status feedback |
| 4×4 keypad (if implemented in the selected source) | Emergency passcode input |
| Buzzer (if wired/implemented) | Audible status indication |
| Regulated power supply, level shifters and common ground | Safe and stable interfacing |

**Wiring warning:** Raspberry Pi GPIO is 3.3 V logic and is not 5 V tolerant. Use suitable level shifting/driver circuitry where required. The report contains pin/interface descriptions that should be reconciled against the actual source and wiring diagram before powering the system; do not assume every pin assignment in the report is conflict-free.

## Software and Python libraries

The following libraries/tools are named in the report. Install only the dependencies required by the actual Python source and your Raspberry Pi OS version.

| Library / tool | Role described in the report |
|---|---|
| Python 3 | Main application language |
| OpenCV (`opencv-python` or OS package `python3-opencv`) | Camera frames, image conversion, preprocessing, display and saving evidence |
| NumPy | Numerical arrays and image/model data handling |
| `face_recognition` | Face encodings, matching and face-distance comparison |
| dlib | Face-recognition model backend used by `face_recognition` |
| MTCNN | CNN-based face detection/localization |
| PyFingerprint (`pyfingerprint`) | Communication and operations for supported fingerprint sensors |
| pySerial (`pyserial`) | UART communication with GPS, GSM and fingerprint hardware |
| SpeechRecognition (`SpeechRecognition`) | Microphone capture and speech-to-text interface |
| PyAudio (`pyaudio`) | Microphone/audio input backend used by SpeechRecognition in common setups |
| Requests (`requests`) | HTTP requests, including reverse-geocoding/API operations described in the report |
| Google API Python Client (`google-api-python-client`) | Google Drive API calls and uploads |
| Google Auth (`google-auth`) | Google API authentication support |
| Google Auth OAuth Library (`google-auth-oauthlib`) | OAuth 2.0 installed-app/browser authorization flow where used |
| Google Auth HTTP (`google-auth-httplib2`) | HTTP transport integration for Google API clients where used |
| `googleapiclient.http.MediaFileUpload` | File upload helper for Drive API, if used by the source |
| `pickle` (Python standard library) | Saving/loading face-encoding data such as `encodings.pkl`; pickle files must only be loaded from trusted sources |
| `threading`, `queue`, `time`, `os`, `sys`, `json`, `hashlib`, `datetime` (standard library, where imported) | Concurrent tasks, timing, paths, configuration, serialization and event timestamps |
| Raspberry Pi OS camera tools such as `rpicam-vid` | Camera capture/streaming on supported Raspberry Pi OS versions |
| GPIO library (for example `RPi.GPIO`, if imported by the source) | Relay, buzzer, keypad and other GPIO operations |
| I2C LCD driver (specific package depends on the source) | LCD output |
| Google Drive API and OAuth 2.0 | Optional cloud storage and authorization |

### Installation caveats

- `face_recognition` and `dlib` can be difficult to install on some Raspberry Pi OS/Python combinations. Use a compatible Python version and platform-specific installation instructions.
- `PyAudio` may need PortAudio development packages before installation.
- GPIO and camera packages are platform-specific and should generally be installed on Raspberry Pi OS, not Windows.
- The exact LCD library and GPIO package must match the imports in your source file.
- Do not assume every package in the table is imported by the current `.py` file. The report describes the design; confirm imports in the source before freezing requirements.

## Repository structure

A recommended structure is:

```text
Raspberry-Pi-Multimodal-Security-System/
├── Raspberry_Pi_Multimodal_Security_System.py
├── README.md
├── requirements_raspberry_pi.txt
├── .gitignore
├── dataset/                 # Local authorized-user images; do not commit
├── unknown_faces/           # Captured evidence; do not commit
├── encodings.pkl            # Generated face encodings; do not commit unless reviewed
├── credentials.json         # Google OAuth client file; never commit
└── token.json               # OAuth token; never commit
```

The program may use different filenames or folders. Keep the names consistent with the actual source code.

## Setup on Raspberry Pi OS

These are baseline setup commands. Review the dependency file and source imports before installing optional packages.

```bash
sudo apt update
sudo apt install -y python3-venv python3-pip python3-dev build-essential cmake
```

Create and activate a virtual environment:

```bash
python3 -m venv --system-site-packages .venv
source .venv/bin/activate
python -m pip install --upgrade pip
```

If the project contains a dependency file, install from it:

```bash
python -m pip install -r requirements_raspberry_pi.txt
```

Install camera/GPIO/system libraries using the instructions for your Raspberry Pi OS release. Do not install the Raspberry Pi hardware stack into a Windows environment expecting the GPIO/camera hardware to work there.

## Dataset preparation

Create a separate folder for each authorized user and add clear, consented face images. For example:

```text
dataset/
├── AuthorizedUser1/
│   ├── face_01.jpg
│   └── face_02.jpg
└── AuthorizedUser2/
    ├── face_01.jpg
    └── face_02.jpg
```

Use the exact dataset path expected by the Python source. If the source generates `encodings.pkl`, create it using the project's own enrollment/encoding workflow. Do not download or load untrusted pickle files.

Fingerprint enrollment is separate: enroll fingerprints on the supported sensor and store only the user/template mapping needed by the application.

## Configuration checklist

Before running, inspect the Python source and verify:

- GPIO pin numbers and relay active level (active-high vs active-low).
- UART device paths and baud rates for GPS, GSM and fingerprint sensor.
- GPS NMEA parsing and timeout behavior.
- GSM SIM readiness, antenna, network registration and SMS settings.
- Camera availability and `rpicam-*` command compatibility with your OS version.
- Microphone device and speech-recognition backend.
- LCD I2C address and driver.
- Emergency passcode, retry behavior and timeout.
- Owner phone number and any alert destinations.
- Google OAuth credentials, Drive folder ID and sharing policy, if Drive upload is enabled.
- Local storage permissions for face encodings and evidence images.

Keep secrets in local configuration/environment variables where possible. Do not hard-code real passwords, personal phone numbers, OAuth secrets, or tokens in a public repository.

## Run the program

From the project directory on the Raspberry Pi, with the intended environment activated:

```bash
python Raspberry_Pi_Multimodal_Security_System.py
```

Use the exact filename present in your repository. The command starts the application; it does not automatically enroll faces or fingerprints unless the source explicitly implements that behavior.

## Test plan

Test on a bench setup before connecting anything to a real vehicle:

1. Boot with the relay disconnected and confirm the default output is safe.
2. Test camera capture and authorized/unknown face paths.
3. Test fingerprint match and mismatch paths.
4. Test voice command acceptance and rejection.
5. Test emergency passcode success/failure and timeout behavior.
6. Test GPS coordinate parsing with valid and missing-fix cases.
7. Test GSM SMS sending independently.
8. Test evidence capture and optional Drive upload.
9. Test LCD/buzzer messages and concurrent-task shutdown.
10. Simulate module disconnects and power cycles; verify relay returns to OFF after exceptions or reboot.

Record observed results; do not claim a test passed unless you actually ran it on the target hardware.

## Safety and privacy

- **Prototype only:** Do not connect this project directly to a road-going vehicle ignition without qualified automotive engineering review, proper isolation, fusing, fail-safe design, and legal/safety assessment.
- Test relay logic using a low-voltage lamp or isolated test motor first.
- Confirm the relay defaults to OFF at startup, crash, shutdown, and loss of power.
- Keep biometric images, face encodings, GPS logs, SMS recipient details, OAuth credentials, and tokens out of public Git repositories.
- Obtain consent before enrolling faces or fingerprints.
- The voice stage described in the report checks for a command word; it is not a secure speaker identity check.
- GPS accuracy depends on satellite reception. SMS depends on GSM coverage and an active SIM. Cloud upload depends on network connectivity and valid OAuth authorization.

## Known limitations

- This system is hardware-dependent and cannot be fully executed on a Windows laptop without a Raspberry Pi and connected modules.
- Installation support varies by Raspberry Pi OS and Python version, particularly for dlib/face-recognition and audio packages.
- Speech recognition via `recognize_google()` requires network access, despite the report's general discussion of offline processing.
- The report describes optional driver-safety extensions such as drowsiness detection, obstacle sensing and speed regulation; these should be treated as future/optional modules unless they are present and verified in the source code.
- This README documents the reported design. Actual features, pin mappings, library imports, and test results must be confirmed against the final source and physical prototype.
