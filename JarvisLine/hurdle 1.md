## Current Problem

Linphone (Windows) can't register to Asterisk (WSL2/Docker) via SIP port 5060.

---

## Why It's Hard

WSL2 is a Hyper-V VM with NAT. Windows and WSL2 are on **different subnets**:

- Windows: `172.24.16.1`
- WSL2/Asterisk: `172.24.24.17`

Linphone sends SIP registration packets from Windows → they don't reliably reach WSL2.

---

## Approaches Tried

|Approach|Result|
|---|---|
|Linphone → `172.24.24.17` UDP|Timeout|
|Linphone → `172.24.24.17` TCP|Error during connection|
|Windows Firewall rules (inbound UDP/TCP 5060)|Didn't fix it|
|`netsh portproxy` UDP forward|Not supported (UDP only works TCP)|
|`socat` UDP forward in WSL|Conflicted with Asterisk on same port, Connection refused|
|PowerShell raw UDP packet to `172.24.24.17:5060`|**Succeeded (36 bytes sent)** — proves UDP routing works|
|`netsh portproxy` TCP `127.0.0.1:5060` → `172.24.24.17:5060`|Error during connection|
|Added TCP transport to Asterisk pjsip.conf|Both UDP+TCP now running, didn't fix Linphone|

---

## Key Facts

- Asterisk IS running correctly (PJSIP loaded, both transports up)
- Raw UDP packets from PowerShell DO reach Asterisk
- Linphone SIP REGISTER packets are NOT reaching Asterisk
- `docker logs -f asterisk` shows zero activity when Linphone tries to connect
- Asterisk endpoint `linphone` shows "Unavailable"

---

## Root Cause (likely)

Linphone is probably sending SIP REGISTER to the right IP but the **response from Asterisk** can't get back to Linphone because:

- Asterisk responds from `172.24.24.17`
- Linphone is on Windows at `172.24.16.x`
- The return path is blocked by Hyper-V NAT

---

## Options Going Forward

**Option A — Mirror port to Windows using a UDP proxy tool** Use a proper UDP proxy like `udpxy` or write a tiny Node.js UDP bridge that runs on Windows and forwards to WSL.

**Option B — Run Linphone inside WSL using X11/GUI** Both Linphone and Asterisk on same network — guaranteed to work. But needs X11 setup (complex on Windows).

**Option C — Use a SIP softphone that runs inside WSL terminal** `pjsua` is a command-line SIP client that runs in WSL directly — same network as Asterisk, no NAT issues.

**Option D — Switch Asterisk to run on Windows directly** Run Asterisk natively on Windows (via WSL port-forward trick or a Windows Asterisk build) so Linphone and Asterisk are on the same network.

**Option E — Use `wsl --mirror` networking mode** WSL2 has a mirrored networking mode where WSL and Windows share the same IP. This would make `127.0.0.1` work for both. Requires Windows 11 22H2+.

---

**My recommendation: Option E first (5 min fix if supported), then Option C as backup.**

What's your Windows version? Run in PowerShell:

```powershell
winver
```