<p align="center">
  <img src="assets/mask.svg" alt="Respirator" width="150">
</p>

# Level 4 — Free Air-Quality Alerts for Home Assistant

Spoken air-quality alerts for Home Assistant (https://www.home-assistant.io/): when the AQI rises above a threshold you
choose, your speakers announce it — and again when it drops back below. No PurpleAir, **no
API key, no account, no device to pick**. The reading comes from the free
[Open-Meteo Air-Quality API](https://open-meteo.com/en/docs/air-quality-api) using a
location you set. I am not responsible for the air quality, the accuracy of this information, or its effects on you. 

> ⚠️ **Disclaimer:** This code has not been tested beyond my personal Home Assistant setup. Use at your own risk.

## What it uses (data & APIs)

- **API:** [Open-Meteo Air-Quality API](https://open-meteo.com/en/docs/air-quality-api) —
  free, **no API key, no account, no sign-up**.
- **What's sent:** only a **latitude/longitude** (the location you set), as an HTTPS GET.
  No personal data, no credentials, nothing else leaves Home Assistant.
- **What's returned:** the current **US AQI** and **PM2.5** for that point; polled every ~30 min.
- **Requirements:** Home Assistant with outbound internet, a working **TTS service**
  (e.g. `tts.google_translate_say`), and one or more **cast-capable speakers**.
- **No other cloud:** the sensor is the only external call; triggering and speech are all local to HA.

Two pieces:

1. **`packages/outdoor_aqi.yaml`** — a keyless REST sensor, `sensor.outdoor_aqi` (US AQI).
2. **`blueprints/automation/aqi_speaker_alert.yaml`** — an automation blueprint where you pick
   the **threshold**, **speakers**, **messages**, and **debounce** at setup.

The blueprint works with *any* numeric AQI sensor, so you can also point it at a PurpleAir or
AirNow sensor if you already have one — the Open-Meteo sensor is just the zero-key default.

---

## 1. Add the Outdoor AQI sensor

**Option A — packages (recommended).** In your `configuration.yaml`, enable packages once:

```yaml
homeassistant:
  packages: !include_dir_named packages
```

Then copy `packages/outdoor_aqi.yaml` into a `packages/` folder next to your
`configuration.yaml`.

**Option B — paste inline.** Copy just the `rest:` block from `packages/outdoor_aqi.yaml`
into your `configuration.yaml`.

**Set your location (required):** edit the `latitude=` / `longitude=` values in the sensor's
URL. Find yours in **Settings → System → General**, or look up a ZIP/postcode at
[latlong.net](https://www.latlong.net/). (An optional auto-use-your-home-location variant is
documented in a comment inside the file.)

Then **Developer Tools → YAML → Check Configuration**, then **Reload all YAML
configuration** (or restart HA). You should get `sensor.outdoor_aqi` with a number.

## 2. Import the blueprint

Click this, or copy the raw URL into
**Settings → Automations & Scenes → Blueprints → Import Blueprint**:

[![Open your Home Assistant instance and show the blueprint import dialog.](https://my.home-assistant.io/badges/blueprint_import.svg)](https://my.home-assistant.io/redirect/blueprint_import/?blueprint_url=https%3A%2F%2Fgithub.com%2Fnganeshcode%2Flevel-4-air-quality-alert%2Fblob%2Fmain%2Fblueprints%2Fautomation%2Faqi_speaker_alert.yaml)

## 3. Create the automation

**Settings → Automations & Scenes → Create Automation → Use blueprint → "Level 4 — AQI speaker alert".**
Fill in:

| Input | What it does |
|---|---|
| **AQI sensor** | `sensor.outdoor_aqi` (or your own AQI sensor) |
| **AQI threshold** | the number to alert on — e.g. `80` |
| **Speaker(s)** | the media players to announce on |
| **Text-to-speech service** | `tts.google_translate_say` (default), or `tts.cloud_say`, etc. |
| **Debounce (minutes)** | how long it must stay across the line before speaking (default `10`; `0` = instant) |
| **Messages** | what to say above / below (templates allowed — `{{ states(aqi_sensor) }}`, `{{ threshold }}`) |

Save, and you're done.

---

## How it behaves

- **One alert up, one alert down.** The triggers are *edge-triggered* — they fire once when the
  AQI crosses the threshold, not repeatedly while it stays there. They re-arm only after it
  crosses back.
- **Debounce** (the `for:` delay) means a value flickering around the threshold, or a brief
  sensor dropout, won't spam you — it must hold across the line for the chosen minutes.

## Testing

To hear it immediately without waiting for real air quality: **Developer Tools → Actions →
`Automation: Trigger`**, pick your automation, and run it. (That bypasses the trigger/debounce
and just plays the announcement on your speakers.)

## Notes

- The sensor reports **Open-Meteo's US AQI**. Different providers compute AQI with slightly
  different methods, so it may not match a neighbor's PurpleAir to the point.
- Requires a working TTS service and cast-capable speakers already set up in HA.

## Acknowledgments

Special thanks to Curtis Markson.

## License

MIT — see [LICENSE](LICENSE).
