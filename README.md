# docker-streaming

This is a **ready-to-use streaming server in Docker**.  
It can take your live video from OBS (or another encoder) and:
- **RIST relay**: Keep your stream stable and clean (no audio pops, less stutter)
- **RIST forwarder**: Send it to another server (like a cloud relay or Twitch)
- **SRT / SRTLA**: Take streams from bonded backpacks or mobile encoders
- **NOALBS (optional)**: Automatically switch to a BRB scene in OBS if your bitrate drops
- Runs on **PCs, servers, Raspberry Pi, Orange Pi, Radxa boards**, and more.

---

## 1. What’s included

- `docker-compose.yml` → Tells Docker what to run  
- `.env` → Where you put versions and your stream key  
- `env_noalbs` → Login details for the NOALBS bot (optional)  
- `config_noalbs.json` → NOALBS rules for scene switching (optional)  
- `logfile.json` → Controls logging format  
- `LICENSE` → This project is MIT licensed (you can use it freely)

---

## 2. What you need before starting

- **Docker** and **Docker Compose** installed on your system  
  (Guide: https://docs.docker.com/get-docker/)  
- **An internet connection**  
- **Ports open** on your network/router for the services you want to use (see Ports section below)  
- **An encoder** (OBS, FFmpeg, or hardware backpack) that can send RIST, SRT, or SRTLA streams  

---

## 3. Quick Start (Step-by-step)

### Step 1: Download the project
```bash
git clone https://github.com/moo-the-cow/docker-streaming.git
cd docker-streaming
```

### Step 2: Edit your `.env` file
Open `.env` in a text editor. You’ll see something like:
```ini
GLOBAL_RIST_VERSION=0.0.11
NOALBS_VERSION=0.0.8
GLOBAL_SRT_VERSION=0.0.7
SECRET_HASH=secretabc123
```
- **GLOBAL_RIST_VERSION** — leave this unless you know you need a different image version
- **NOALBS_VERSION** — leave this unless you know you need a different version of NOALBS
- **GLOBAL_SRT_VERSION** — same as above
- **SECRET_HASH** — in this example, we set it to `secretabc123`. This is your RIST PSK (pre-shared key).

> If you change `SECRET_HASH`, you must also change it in OBS or your encoder.

### Step 3: (Optional) Enable NOALBS
If you want auto scene switching when your bitrate drops:
1. Open `env_noalbs`  
2. Add your Twitch bot username and OAuth token:
```dotenv
TWITCH_BOT_USERNAME=mybotname
TWITCH_BOT_OAUTH=oauth:abc123yourtokenhere
```
3. Open `config_noalbs.json` and change scene names to match your OBS setup (e.g., `"normal": "LIVE", "low": "BRB"`).

### Step 4: Start the server
```bash
docker compose up -d
```
`-d` means “run in the background.”

### Step 5: Send your stream to it

#### **RIST (with secret hash)**
In OBS:
- Output URL:
```
rist://YOUR_SERVER_IP:2030?secret=secretabc123
```
- Replace `YOUR_SERVER_IP` with your server’s public IP or domain
- Port `2030` is the default RIST listener in `docker-compose.yml`  
- `secretabc123` is your PSK from `.env` (`SECRET_HASH`)

#### **SRT**
In OBS or FFmpeg:
```
srt://YOUR_SERVER_IP:5000?streamid=secretabc123
```

#### **SRTLA**
In OBS or FFmpeg:
```
srtla://YOUR_SERVER_IP:5000?streamid=secretabc123
```

---

## 4. Ports (what to open on your router/firewall)

By default, the `docker-compose.yml` maps these:
- **RIST Relay**: `2030/udp`
- **RIST Forwarder**: `2031/udp` (optional)
- **SRTLA**: `5000–5001/udp`
- **SRT**: `5000–5001/udp`
- **Web UIs / APIs**: `8681`, `8683`, `8283`

> If you don’t know how to open ports, search “Port Forwarding” + your router model.

---

## 5. How it works

```
[OBS / Backpack] → (RIST / SRT / SRTLA) → [docker-streaming server] → (optional forwarder) → [Cloud or Twitch/YouTube]
                                      ↘ (optional NOALBS) → [Tells OBS to change scenes]
```

- **RIST Relay**: Keeps your stream smooth even with some packet loss  
- **RIST Forwarder**: Sends your feed to a second location (another server, CDN, etc.)  
- **SRT/SRTLA**: For bonded or single SRT connections  
- **NOALBS**: Watches your stats, tells OBS to switch to BRB if quality drops

---

## 6. Updating

When there’s an update to the images:
```bash
git pull
docker compose pull
docker compose up -d
```

---

## 7. Checking logs

If something doesn’t work, run:
```bash
docker compose logs -f
```
Press **CTRL + C** to exit.

---

## 8. Troubleshooting

**No video or connection refused**
- Make sure the port is open
- Check your encoder’s URL/port matches the compose file
- If using a firewall, allow the port (UDP for RIST/SRT)

**Stuttering or drops**
- Use RIST for the cleanest feed
- Avoid Wi-Fi for the first hop
- Increase buffer/recovery time in OBS if needed

**NOALBS not switching**
- Check your `config_noalbs.json` has the correct scene names
- Make sure OBS WebSocket is running and matches NOALBS settings

---

## 9. Security tips

- Never share your `.env` or `env_noalbs` files — they contain secrets
- Only open the ports you need
- Keep Docker and your images updated

---

## 10. License

This project is licensed under MIT — you can use it freely, but no warranty is provided.
