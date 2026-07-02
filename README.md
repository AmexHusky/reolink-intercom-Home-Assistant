# Reolink Doorbell 2-Way Intercom for Home Assistant (Low-Latency)

A ready-to-use, low-latency 2-way audio intercom for **Reolink Video Doorbells** in
Home Assistant. Opens a popup on your dashboard/tablet when someone rings, shows the
live video, and lets you talk back — with the **backchannel latency tuned down from
3–5 s to ~1–2 s** (on par with the official Reolink app).

Built on top of the excellent community work by
[victorigualada](https://community.home-assistant.io/t/2-way-audio-intercom-for-reolink-doorbell-made-easy/832189),
with the audio path re-tuned for real-world latency and packaged as an importable
Blueprint.

> 🇩🇪 **Deutsche Anleitung weiter unten** → [Deutsch](#deutsch)

---

## Why this is faster

The Reolink backchannel uses a proprietary protocol that only accepts **G.711
(PCMA / PCMU)** audio. If go2rtc has to transcode your browser's Opus into that format
inside a large buffer, you get several seconds of delay. This project fixes two things:

1. **Direct codec match** — transcode straight to `pcma`/`pcmu`, no double conversion.
2. **`#async` flag** — decouples audio/video timing so ffmpeg stops waiting on a full
   buffer before forwarding.

Result: backchannel drops from **3–5 s → ~1–2 s**. This is the practical floor for
Reolink hardware; the remaining latency is inherent to their protocol.

---

## Requirements

- Home Assistant with the **Reolink** integration set up
- **go2rtc** (bundled with the WebRTC Camera add-on, or standalone) — v1.9.x tested
- [WebRTC Camera](https://github.com/AlexxIT/WebRTC) (`custom:webrtc-camera`) via HACS
- [Browser Mod](https://github.com/thomasloven/hass-browser_mod) via HACS
- **HTTPS access to Home Assistant** — mandatory (see below)

### ⚠️ HTTPS is mandatory for the microphone

Browsers only allow microphone access (`getUserMedia`) in a **secure context**:
HTTPS or `localhost`. Over plain HTTP the mic is silently blocked and you'll get
one-way audio only. Use **any** of:

- **Nabu Casa Home Assistant Cloud** (easiest)
- A **reverse proxy** (Nginx Proxy Manager / Caddy / Traefik) serving HA over HTTPS
  on your LAN
- Your existing **external HTTPS URL**

Access the dashboard through the HTTPS URL on the tablet, not the internal `http://…:8123`.

---

## Setup

### 1. go2rtc stream

Add this stream to your `go2rtc.yaml` (adjust IP / credentials):

```yaml
streams:
  reolink_doorbell:
    - rtsp://USER:PASSWORD@DOORBELL_IP:554/h264Preview_01_sub
    - "ffmpeg:reolink_doorbell#audio=pcmu#async"
```

- Use the **`_sub`** stream — lower resolution, lower latency.
- Start with `#audio=pcma`. If your firmware refuses it (no audio, backchannel errors),
  switch to `#audio=pcmu`. Newer firmwares (e.g. `v3.0.0.4662`) often need **pcmu**.
- Restart go2rtc after changes.

Optional — verify the negotiated codec:

```yaml
log:
  level: info
  exec: trace
```

Look for `[backchannel]` lines in the go2rtc log.

### 2. Import the Blueprint

In HA: **Settings → Automations & Scenes → Blueprints → Import Blueprint**, paste the
raw URL of `blueprints/reolink_2way_intercom.yaml`, then create an automation from it
and fill in:

- **Ring sensor** — your doorbell's visitor `binary_sensor`
- **go2rtc stream name** — e.g. `reolink_doorbell`
- **Target browser(s)** — the `browser_id` of your tablet(s)
- **Unlock script** (optional)
- **Auto-close timeout** (default 2 min)

Done. Ring the bell → popup opens → tap 📞 to talk.

---

## Credits & License

- Original concept: victorigualada & the Home Assistant community thread linked above
- Latency tuning & Blueprint packaging: this repo
- MIT License — see [LICENSE](LICENSE)

---

<a name="deutsch"></a>
# 🇩🇪 Deutsch

Fertige, latenzarme 2-Wege-Gegensprechanlage für **Reolink Video-Türklingeln** in
Home Assistant. Klingelt es, öffnet sich ein Popup auf Dashboard/Tablet mit Live-Video,
und du kannst zurücksprechen — mit **Rückkanal-Latenz von 3–5 s auf ~1–2 s gesenkt**
(etwa gleichauf mit der offiziellen Reolink-App).

## Warum das schneller ist

Der Reolink-Rückkanal nutzt ein proprietäres Protokoll, das nur **G.711 (PCMA / PCMU)**
akzeptiert. Muss go2rtc das Browser-Opus in einem großen Puffer dorthin transcodieren,
entstehen mehrere Sekunden Verzögerung. Dieses Projekt behebt zwei Dinge:

1. **Direkter Codec-Match** — direkt nach `pcma`/`pcmu`, keine doppelte Umwandlung.
2. **`#async`-Flag** — entkoppelt Audio/Video-Timing, ffmpeg wartet nicht mehr auf
   einen vollen Puffer.

Ergebnis: Rückkanal **3–5 s → ~1–2 s**. Das ist die praktische Untergrenze der
Reolink-Hardware — der Rest ist protokollbedingt.

## Voraussetzungen

- Home Assistant mit eingerichteter **Reolink**-Integration
- **go2rtc** (im WebRTC-Camera-Add-on enthalten oder standalone) — v1.9.x getestet
- [WebRTC Camera](https://github.com/AlexxIT/WebRTC) über HACS
- [Browser Mod](https://github.com/thomasloven/hass-browser_mod) über HACS
- **HTTPS-Zugriff auf Home Assistant** — zwingend

### ⚠️ HTTPS ist Pflicht fürs Mikrofon

Browser erlauben Mikrofon-Zugriff nur im **sicheren Kontext**: HTTPS oder `localhost`.
Über reines HTTP wird das Mikro blockiert → nur Einweg-Audio. Nutze eines von:

- **Nabu Casa Home Assistant Cloud** (am einfachsten)
- **Reverse Proxy** (Nginx Proxy Manager / Caddy / Traefik) für HTTPS im LAN
- Deine bestehende **externe HTTPS-URL**

Das Dashboard auf dem Tablet über die HTTPS-URL öffnen, **nicht** über `http://…:8123`.

## Einrichtung

**1. go2rtc-Stream** in `go2rtc.yaml` (IP/Zugangsdaten anpassen):

```yaml
streams:
  reolink_doorbell:
    - rtsp://USER:PASSWORD@DOORBELL_IP:554/h264Preview_01_sub
    - "ffmpeg:reolink_doorbell#audio=pcmu#async"
```

- Immer den **`_sub`**-Stream nutzen (niedrigere Auflösung, weniger Latenz).
- Erst `#audio=pcma` testen. Klappt es nicht (kein Ton, Backchannel-Fehler), auf
  `#audio=pcmu` wechseln. Neuere Firmwares (z. B. `v3.0.0.4662`) brauchen meist **pcmu**.
- go2rtc danach neu starten.

**2. Blueprint importieren:** Einstellungen → Automatisierungen & Szenen → Blueprints →
Blueprint importieren, Raw-URL von `blueprints/reolink_2way_intercom.yaml` einfügen,
Automatisierung erstellen und ausfüllen:

- **Klingel-Sensor** (Besucher-`binary_sensor`)
- **go2rtc-Streamname** (z. B. `reolink_doorbell`)
- **Ziel-Browser** (`browser_id` des Tablets)
- **Türöffner-Skript** (optional)
- **Auto-Schließen-Timeout** (Standard 2 Min)

Fertig. Klingeln → Popup öffnet sich → 📞 antippen zum Sprechen.

## Lizenz

MIT — siehe [LICENSE](LICENSE). Ursprungskonzept: victorigualada & HA-Community.
