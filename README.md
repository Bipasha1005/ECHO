# Echo Voice Assistant

A Windows-first voice assistant that pairs a web UI (Eel) with wake-word detection, face authentication, and command execution (apps, web, YouTube, WhatsApp/phone actions) with optional Gemini responses.

## Features
- Wake word "echo" using Porcupine, then continuous listening and speech synthesis.
- Face authentication via OpenCV LBPH before enabling the UI.
- Voice commands: open local apps/URLs, play YouTube terms, send/launch WhatsApp calls/messages, basic phone automation hooks, performing computer shortcuts over voice and fallback to Gemini for general prompts.

## Project layout
- [main.py](main.py): boots the Eel app, triggers face auth, opens the UI.
- [run.py](run.py): spawns the UI and hotword listener in separate processes.
- [engine/command.py](engine/command.py): speech I/O and command router; handles continuous mode.
- [engine/features.py](engine/features.py): command handlers (open apps/web, YouTube, WhatsApp/phone, Gemini) and Porcupine loop.
- [engine/auth/](engine/auth/): face auth (LBPH model, cascade, trainer artifacts).
- [www/](www/): front-end assets.

## Prerequisites
- Python 3.12 (matching the included `envecho` venv) on Windows.
- Microphone and camera access (wake word + face auth).
- Microsoft Edge available (the UI is opened via `start msedge.exe`).
- For `pyaudio`, ensure PortAudio-compatible drivers; on Windows the wheel usually installs directly.

## Setup
```bash
python -m venv .venv
.venv\Scripts\activate
pip install --upgrade pip
pip install -r requirements.txt
```

## Configuration
- Update [engine/config.py](engine/config.py) with your own Gemini API key (`LLM_KEY`). Avoid committing secrets; consider loading from environment variables in production.
- Change `ASSISTANT_NAME` if you want a different wake name in responses.
- Ensure the trained face model exists at [engine/auth/trainer/trainer.yml](engine/auth/trainer/trainer.yml). To retrain, capture samples (see `engine/auth/sample.py`) and run `engine/auth/trainer.py`.
- SQLite data is stored in `echo.db` (commands, contacts). Seed data can be imported from [contacts.csv](contacts.csv).

## Running
```bash
# Start UI + hotword listener
python run.py
```
This opens the UI at `http://localhost:8000/index.html` in Edge and keeps the hotword listener alive. If you only need the UI without hotword detection, run `python main.py`.

## Usage notes
- First-time flow: app plays a start sound, prompts for face auth, then responds to wake word "echo". Say commands like "open calculator", "play lofi on youtube", "send message to Alice", or ask a question to hit Gemini.
- WhatsApp actions rely on the desktop app; the automation uses simulated key events/tabs.
- Phone automation helpers use ADB; connect an Android device with USB debugging if you plan to call or SMS via the device.

## Troubleshooting
- No audio input: check microphone permissions and that `pyaudio` can open the device.
- Hotword not detected: verify Porcupine installation and microphone sample rate; ensure background noise is low.
- Face auth fails: confirm the camera works and that `trainer.yml` matches your captured samples.
- Edge does not open: change the launcher in [main.py](main.py) or manually open `http://localhost:8000/index.html`.

## Security
- Replace the placeholder Gemini key with your own and keep it out of version control.
- Stored data (contacts, commands) lives in `echo.db`; handle it as you would any personal data.
