# Raspberry Pi Multimodal Security System

A Raspberry Pi-based vehicle security prototype that combines face recognition, fingerprint verification, a voice command, GPS/GSM alerts, and an emergency password mode.

## Features

- Face recognition using a local image dataset.
- Fingerprint verification through a UART fingerprint sensor.
- Voice-command check requiring the word `start`.
- GPS location retrieval and reverse geocoding.
- SMS alerts through a GSM modem.
- Optional Google Drive upload for unknown-face images.
- Emergency password flow and relay output control.

## Project files

- `Raspberry_Pi_Multimodal_Security_System.py` — main program.
- `requirements_raspberry_pi.txt` — Python dependency list.
- `dataset/` — create this folder and add one subfolder per authorized person, containing their face images.
- `credentials.json` — optional Google OAuth client credentials; do not commit this file.
- `token.json` — OAuth token created during authorization; do not commit this file.

## Requirements

This project is intended for a compatible Raspberry Pi OS installation with the required camera, GPIO, UART sensors, microphone, fingerprint sensor, GSM modem, GPS module, and relay wired and configured.

It does **not** run as a complete hardware system on ordinary Windows. Windows can be used to edit the source and check Python syntax, but Raspberry Pi hardware-specific components such as `RPi.GPIO`, `rpicam-vid`, and UART peripherals require the Raspberry Pi environment.

## Setup on Raspberry Pi

1. Copy this repository to your Raspberry Pi and open a terminal in the project directory.
2. Install any OS packages required by your Raspberry Pi OS version and hardware.
3. Install Python dependencies using a virtual environment where possible:

   ```bash
   python3 -m venv --system-site-packages .venv
   source .venv/bin/activate
   python -m pip install -r requirements_raspberry_pi.txt
   ```

   Some dependencies, especially `face-recognition`/`dlib` and `PyAudio`, may require extra system libraries or compatible versions.

4. Create the face dataset structure:

   ```text
   dataset/
   ├── Person1/
   │   ├── face1.jpg
   │   └── face2.jpg
   └── Person2/
       └── face1.jpg
   ```

5. Review the configuration in the Python source before running. Confirm `RELAY_PIN`, `GPS_PORT`, `GSM_PORT`, `FINGER_UART`, baud rates, phone number, password, and relay polarity match your actual hardware.
6. If using Google Drive upload, add your own OAuth `credentials.json` locally and authorize access. Never commit credentials or tokens.
7. Run the program:

   ```bash
   python Raspberry_Pi_Multimodal_Security_System.py
   ```

## Safety notes

- Test first with the relay and motor disconnected.
- Confirm the relay's real ON/OFF polarity and GPIO wiring before connecting a vehicle circuit.
- Use only authorized people's face images and protect the dataset.
- Do not publish personal phone numbers, passwords, Google credentials, OAuth tokens, or private images.
- Syntax checking does not confirm correct behavior with physical hardware. Test each module independently before integrated use.

## Limitations

This is a hardware-dependent prototype. Successful execution depends on compatible Raspberry Pi OS packages, connected devices, correct serial-port assignments, camera configuration, network availability, and valid Google OAuth setup when Drive upload is enabled.
