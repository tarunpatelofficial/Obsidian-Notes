## What We've Built So Far

### Completed Components

**1. Ollama API Server** (`~/claude-phone-free/ollama-api-server/server.js`)

- Runs on port 3333
- Replaces: Claude Code CLI (paid, $20/month Max subscription)
- Using: Ollama + qwen2.5:7b (free, local, runs on your RTX 3050)
- Handles conversation sessions per call, keeps message history

**2. Piper TTS Server** (`~/claude-phone-free/tts-service/tts_server.py`)

- Runs on port 5500
- Replaces: ElevenLabs API (paid, ~$5-22/month)
- Using: Piper TTS with en_US-lessac-medium voice (free, local)
- Takes text → returns WAV file URL

**3. Faster-Whisper STT Server** (`~/claude-phone-free/stt-service/stt_server.py`)

- Runs on port 5600
- Replaces: OpenAI Whisper API (paid, $0.006/min)
- Using: faster-whisper base model (free, local)
- Takes audio → returns transcribed text

**4. Voice App** (`~/claude-phone-free/voice-app/`)

- Taken from NetworkChuck's repo
- Patched `tts-service.js` → points to Piper (port 5500) instead of ElevenLabs
- Patched `whisper-client.js` → points to faster-whisper (port 5600) instead of OpenAI
- Runs in Docker, handles SIP call logic via Drachtio + FreeSWITCH

---

### What's NOT Done Yet

- SIP phone system (was 3CX, switching to Asterisk)
- Docker stack not started yet
- End-to-end call not tested yet

---

### Why Asterisk Instead of 3CX

3CX modern UI deliberately hides SIP credentials — designed for QR provisioning and managed phones, not custom SIP clients like FreeSWITCH. We hit a dead end trying to get Auth ID/Password.

Asterisk is fully open source, config is plain text files, credentials are whatever we define, runs in Docker on WSL, free forever, no account/license needed. For R&D it's actually better — full visibility and control over everything.

---

### Full Architecture & Flow

```
YOUR PHONE (Linphone app, free SIP softphone)
         |
         | SIP (port 5060)
         v
+------------------+
|    ASTERISK      |  <-- runs in Docker (WSL)
|  (replaces 3CX)  |      plain text config
|  extension 9000  |      we control everything
+--------+---------+
         |
         | SIP (internal)
         v
+------------------+
|   DRACHTIO       |  <-- SIP middleware (Docker, WSL)
|   port 9022      |      bridges Asterisk <-> FreeSWITCH
+--------+---------+
         |
         v
+------------------+
|  FREESWITCH      |  <-- media server (Docker, WSL)
|  port 5080       |      handles audio streaming
+--------+---------+
         |
         | raw PCM audio
         v
+------------------+
|   VOICE-APP      |  <-- Node.js (Docker, WSL)
|   port 3000      |      conversation loop
+---+---------+----+
    |         |
    |         | text (transcript)
    v         v
+-------+  +------------------+
| Piper |  | faster-whisper   |
|  TTS  |  |      STT         |
| :5500 |  |     :5600        |
+-------+  +------------------+
    |               |
    | WAV URL       | transcribed text
    v               v
         +------------------+
         |  OLLAMA API      |
         |    SERVER        |
         |     :3333        |
         +--------+---------+
                  |
                  | HTTP
                  v
         +------------------+
         |  OLLAMA          |  <-- runs on WINDOWS
         |  qwen2.5:7b      |      uses RTX 3050 GPU
         |  port 11434      |
         +------------------+
```

### Call Flow (what happens when you call extension 9000)

```
1. You call 9000 on Linphone
2. Asterisk receives call, routes to FreeSWITCH via Drachtio
3. FreeSWITCH opens audio stream
4. Voice-app starts conversation loop:
   a. Plays greeting (via Piper TTS → WAV)
   b. Listens for your speech (raw PCM audio)
   c. Sends audio to faster-whisper → gets text
   d. Sends text to Ollama API server → gets AI response
   e. Sends response to Piper TTS → gets WAV
   f. Plays WAV back to you
   g. Loop back to (b)
5. You hang up → session ends
```

---

Ready to set up Asterisk?