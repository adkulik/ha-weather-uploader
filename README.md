🇬🇧 **English** | 🇭🇺 [Magyar](README.hu.md)

# ⛅ Weather Station Uploader – Időkép, Metnet, Weather Underground (Home Assistant Blueprint)

[![Buy Me a Coffee](https://img.shields.io/badge/Buy%20me%20a%20coffee-FFDD00?logo=buymeacoffee&logoColor=black)](https://buymeacoffee.com/adkulik)

This Home Assistant blueprint sends your weather sensors to several weather networks at once, configured entirely from the UI:

* **Időkép** (Hungary's largest weather network; Normál and Pro accounts, native or Weather Underground format)
* **Metnet** (Hungarian weather community)
* **Weather Underground**

Each site can be switched on separately, and you pick your sensors only once. The blueprint takes care of the rest:

* Every site gets the data in its own format and units (°C/°F, hPa/inHg, m/s/mph, mm/inch, decimal comma or point).
* Enabled sites are uploaded in parallel. If one site is down, the others still get the data.
* Unavailable/unknown sensors are left out of the upload, so no fake zeros are sent.

> **Note:** the blueprint's UI (field names and descriptions) is in Hungarian, since most of its users upload to the Hungarian networks. The table below lists every field with its English meaning.

> The previous, Időkép-only **2.x blueprint** (`idokep.hu-uploader.yaml`) is still in this repository and keeps working, but new features only go into 3.x. See [Migrating from the 2.x Időkép uploader](#-migrating-from-the-2x-időkép-uploader).

---

## ❗ Please upload only reliable data

These networks' maps and forecasts are used by a lot of people every day. **Only start uploading if your sensors are sited according to meteorological standards.** Badly placed instruments (a thermometer in the sun, an anemometer next to a wall) produce wrong data and hurt the whole network.

* **Temperature / humidity:** 2 m above ground, in the shade, in a proper radiation shield. Never mounted on a wall, concrete or any surface that radiates heat.
* **Anemometer:** in open terrain, ideally 10 m high, or at least 2–3 m above nearby obstacles (roofs, trees).
* **Rain gauge:** on open, level ground, about 1–1.5 m high, not shielded by buildings or trees.
* **Barometer:** may be indoors, but always send **sea-level (relative) pressure**, not the raw station pressure.

---

## ⚠️ Step 1: Prerequisites

Requires **Home Assistant 2024.10 or newer**.

An account on each site you want to upload to:

* **Időkép:** an advanced-level account, [register here](https://www.idokep.hu/regisztracio?szint=halado) (this creates a *Normál* account).
* **Metnet:** your Station ID and key are on the station settings page, under *Weather Underground protokollal küldő állomás*.
* **Weather Underground:** your Station ID and Station Key are on the *My Devices* page.

The blueprint sends data through Home Assistant's built-in `rest_command` integration, which needs one small entry in `configuration.yaml`:

```yaml
rest_command:
  weather_upload:
    url: "{{ url }}"
    method: get
    headers:
      User-Agent: "{{ user_agent | default('HA-Weather-Uploader') }}"
    timeout: 15
```

Save the file and **restart Home Assistant**.

> **What does this do?** It creates a generic `rest_command.weather_upload` action. The blueprint passes the URL and the `User-Agent` header per site, so when a new site is added to the blueprint later, this entry doesn't need to change.

> **Security note:** passwords and keys are part of the URL, so if an upload fails, Home Assistant may write the full URL to the system log, which ends up in diagnostics and backups too. Use a separate password for each site that you don't use anywhere else. All three sites are uploaded over HTTPS (HTTP is optional for Metnet).

---

## 🚀 Step 2: Install

[![Open your Home Assistant instance and show the blueprint import dialog with a specific blueprint pre-filled.](https://my.home-assistant.io/badges/blueprint_import.svg)](https://my.home-assistant.io/redirect/blueprint_import/?blueprint_url=https://raw.githubusercontent.com/adkulik/ha-weather-uploader/refs/heads/main/weather-uploader.yaml)

Or import this URL manually under **Settings → Automations & scenes → Blueprints → Import blueprint**:
`https://raw.githubusercontent.com/adkulik/ha-weather-uploader/refs/heads/main/weather-uploader.yaml`

---

## ⚙️ Step 3: Configure

Create an automation from the *Időjárás-állomás Feltöltő (Időkép, Metnet, Wunderground)* blueprint:

* **Időkép / Metnet / Weather Underground:** switch on (with the *Feltöltés a …* toggle) the sites you want and fill in their credentials. Disabled or incomplete sites are skipped.
  * *Időkép – Fióktípus (account type):* Normál or Pro.
  * *Időkép – Feltöltési formátum (upload format, Normál only):* Időkép format (24-hour rain) or Weather Underground format (daily rain; only this one shows solar radiation as global radiation on Időkép).
  * *Metnet – Kapcsolat (connection):* HTTPS (default) or HTTP.
* **Ütemezés (schedule):** every 1, 2, 5, 10 or 15 minutes (default 5), shared by all sites.
* **Sensors:** only temperature is required; everything else is optional.

### Formats and units

Metnet and Weather Underground always use the Weather Underground (WU) protocol; Időkép can use either. Units are read from each sensor and converted. Wind direction can be degrees (0–360) or a compass point in English (N, NNE …) or Hungarian (É, ÉK, K, DK, D, DNY, NY, ÉNY).

| Data (Hungarian field) | Időkép format | WU format (Metnet, Weather Underground, Időkép WU) |
|---|---|---|
| Temperature (Hőmérséklet) | `hom` (°C) | `tempf` (°F) |
| Humidity (Páratartalom) | `rh` (%) | `humidity` (%) |
| Dew point (calculated) | – | `dewptf` (°F) |
| Pressure (Légnyomás) | `p` (hPa) | `baromin` (inHg) |
| Indoor temperature (Beltéri hőmérséklet) | `inhom` (°C) | `indoortempf` (°F) |
| Wind direction (Szélirány) | `szelirany` (°) | `winddir` (°) |
| Wind speed (Szélsebesség) | `szelero` (m/s) | `windspeedmph` (mph) |
| Wind gust (Széllökés) | `szellokes` (m/s) | `windgustmph` (mph) |
| 1-hour rain (1 órás csapadék) | `csap1h` (mm) | `rainin` (in) |
| 24-hour rain (24 órás csapadék) | `csap` (mm) | – |
| Daily rain since midnight (Napi csapadék – éjfél óta) | – | `dailyrainin` (in) |
| Rain counter (Összes csapadék) | `csaptotal` (mm) | – |
| Solar radiation (Napsugárzás) | `napfeny` (integer W/m²) | `solarradiation` (W/m²) |
| UV index (UV-index) | `uv` | `UV` |

If you upload to Metnet or Weather Underground, set the *daily rain since midnight* sensor. Lux sensors are approximately converted to W/m².

---

## 🔁 Migrating from the 2.x Időkép uploader

3.0 is a new blueprint, so your old automation keeps running until you turn it off:

1. Add the `rest_command.weather_upload` entry from Step 1. Keep the old `idokep_get` entry for now.
2. Restart Home Assistant.
3. Import the 3.0 blueprint and create a new automation from it with the same Időkép credentials and sensors.
4. Disable (or delete) the old 2.x automation.
5. Once nothing uses it, remove the old `idokep_get` entry and restart Home Assistant.

---

## 🔔 Recommended: update notifications (Blueprints Updater)

Home Assistant doesn't tell you when a blueprint has a new version. The [Blueprints Updater](https://github.com/luuquangvu/blueprints-updater) integration adds an update entity for each installed blueprint; it follows this repository automatically. Keep auto-update off and read the changelog first, since a new version may need a `configuration.yaml` change.

---

## ☕ Support

This blueprint is free and will stay free. If you find it useful, you can support further development:

<a href="https://buymeacoffee.com/adkulik"><img src="https://cdn.buymeacoffee.com/buttons/v2/default-yellow.png" alt="Buy Me a Coffee" height="50"></a>

Recurring support is available through [GitHub Sponsors](https://github.com/sponsors/adkulik).

---

## 📋 Changelog

The full changelog, including the 1.x–2.x Időkép uploader history, is in the [Hungarian README](README.hu.md#-változásnapló).

### 3.0 (2026-11-20) – Weather Station Uploader

New blueprint (`weather-uploader.yaml`); the repository is renamed to `ha-weather-uploader`.

* Uploads to **Időkép**, **Metnet** and **Weather Underground** at once, each switchable.
* Sites are uploaded in parallel; one failing site doesn't stop the others.
* New generic `rest_command.weather_upload` (the blueprint sets the `User-Agent` per site), so new sites need no `configuration.yaml` change.
* The Weather Underground measurement part is built once and shared by Metnet, Weather Underground and Időkép's WU format.
* Metnet: HTTPS by default, HTTP optional.
* In WU format the dew point (`dewptf`) is calculated from temperature and humidity.

### 2.8 (2026-11-20)

* The 2.x blueprint's description points to 3.0. Uploads are unchanged.
