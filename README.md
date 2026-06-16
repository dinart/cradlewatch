# CradleWatch — Frigate Baby Monitor Stream Dashboard

CradleWatch is a premium, zero-dependency, static baby monitor dashboard designed to connect directly to an existing **Frigate NVR** server on your local network. It pulls both **live video and live audio** using Media Source Extensions (MSE) over WebSockets, manages auto-reconnections, and triggers an offline audio alarm if the connection drops.

## Features

- **Strict MSE Video + Audio Streaming**: Pulls low-latency (~100ms) synchronized video and audio feeds directly from Frigate's built-in **go2rtc** engine over WebSockets (`/live/mse/api/ws`). No WebRTC required.
- **Zero Extra Ports**: Operates entirely over standard HTTPS port `8971`, removing any need to expose the WebRTC UDP/TCP port `8555` inside the container.
- **Fail-safe Audio Alarm**: If the camera goes offline for **more than 1 minute (60 seconds)**, the dashboard triggers a repeating chime alert (synthesized locally using the Web Audio API).
- **Auto-Silence on Recovery**: The safety alarm is persistent but smart: if the camera feed recovers, the chime automatically stops playing, and the UI resets back to green Live.
- **Default Active Audio**: Audio monitoring starts active by default. Simply click, touch, or press any key on the dashboard to unlock browser audio autoplay blocks.
- **Persistent Local Config**: Saves your Frigate Host and Camera/Stream names directly in the dashboard UI settings, saving them to your browser's `localStorage` for future visits.
- **True Fullscreen Mode**: Double-tap or click the fullscreen button to expand the feed.
- **Diagnostics Panel & Live Logs**: Tracks camera capture FPS, API latency, and displays a scrolling logger console of connection attempts.

---

## Hosting Inside the Container (Same-Origin)

Because Frigate uses session-based authentication cookies, the most reliable way to run the dashboard without cookie-blocking errors is to host it **same-origin** by mounting it directly into the Frigate Docker container's web root.

### 1. Edit Your `docker-compose.yml`
Open your `docker-compose.yml` file on your server. In the `volumes:` section of your `frigate` container config, mount `cw.html` directly into `/opt/frigate/web/assets/cw.html` (mounting into `assets/` successfully bypasses Frigate's React SPA catch-all router):

```yaml
services:
  frigate:
    ...
    volumes:
      - /path/to/config.yml:/config/config.yml
      # Mount cw.html in the assets directory
      - ./cw.html:/opt/frigate/web/assets/cw.html:ro
```

### 2. Restart Frigate
Restart the container to apply the volume mount:
```bash
docker compose up -d --force-recreate
```

### 3. Open the Page in Your Browser
Open your browser and navigate to:
```
https://192.168.31.10:8971/assets/cw.html
```
*(If prompted, log into your Frigate dashboard first. Once logged in, Nginx will serve the file directly, and your browser will automatically pass your existing session cookies to authorize the WebSocket streams.)*

---

## Settings Configuration

Once the page loads, open the **Frigate Configuration** sidebar:
1. **Frigate Host URL**: This will automatically detect your host and default to `https://192.168.31.10:8971`.
2. **MSE Stream Name**: Enter the go2rtc stream name you wish to monitor. Based on your Frigate config:
   - Use `berco_main` for high-quality main resolution stream.
   - Use `berco_sub` for lower resolution sub-stream.
3. **Frigate Camera Name**: Enter the base camera name defined in Frigate: `berco` (used to query the `/api/stats` diagnostics health check).
4. Click **"Save & Connect Feed"**.

---

## Audio Autoplay Notice

Browser security policies block audio from playing automatically without user interaction.
1. The badge in the header shows green **"MONITOR ON"** by default.
2. Tap/click anywhere on the page, touch the screen, or press any key to unlock the browser's autoplay block.
3. Use the volume slider in the sidebar to adjust the volume of the camera's live audio.
4. Click **"Test Alarm Chime"** to ensure that your browser's Web Audio API is working and you can hear the safety alert.
