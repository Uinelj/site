---
tags: [ai-authored, api, sncf]
---

# SNCF TGV INOUI — Onboard WiFi API

> Documented live from train **6706** (Mulhouse → Paris Gare de Lyon) on 2026-07-02.

## Overview

When connected to the **TGV INOUI onboard WiFi**, the captive portal at `https://wifi.sncf` exposes a set of **unauthenticated JSON API endpoints** that power the journey page (`/en/journey`). They are served locally from the train's onboard router — **not reachable from the public internet**.

The base URL for all train endpoints is:

```
https://wifi.sncf/router/api
```

The frontend is a single-page app configured via `/assets/js/app.config.js`, which sets `trainApiUrl: "/router/api"`.

---

## Endpoints

### 1. `GET /router/api/train/gps`

Real-time GPS position of the train.

**Example response:**

```json
{
  "success": true,
  "fix": 10,
  "timestamp": 1783009495,
  "latitude": 48.7042598,
  "longitude": 2.6475387,
  "altitude": 103.315,
  "speed": 66.915,
  "heading": 271.4288
}
```

| Field | Type | Description |
|---|---|---|
| `success` | `boolean` | Whether a GPS fix was obtained |
| `fix` | `number` | Fix quality / satellite count |
| `timestamp` | `number` | Unix epoch (seconds) |
| `latitude` | `number` | Latitude (WGS84, decimal degrees) |
| `longitude` | `number` | Longitude (WGS84, decimal degrees) |
| `altitude` | `number` | Altitude in meters |
| `speed` | `number` | Speed in **m/s** (multiply by 3.6 for km/h) |
| `heading` | `number` | Bearing in degrees (0 = North, clockwise) |

> **Polling:** The frontend polls this every few seconds. Speed of ~67 m/s ≈ **241 km/h**.

---

### 2. `GET /router/api/train/details`

Full journey information: train number, stops, schedule, delays, progress.

**Example response:**

```json
{
  "number": "6706",
  "events": [],
  "onboardServices": [
    "OCEHP", "OCEWO", "OCEPI", "OCECM",
    "OCENY", "OCEBA", "OCEWF", "OCEPP"
  ],
  "additionalServices": {},
  "stops": [
    {
      "code": "FRAEK",
      "label": "Mulhouse",
      "services": { "DRIVER": true },
      "coordinates": {
        "latitude": 47.742691,
        "longitude": 7.34316
      },
      "progress": {
        "progressPercentage": 100,
        "traveledDistance": 37578.31,
        "remainingDistance": 0
      },
      "theoricDate": "2026-07-02T13:40:00.000Z",
      "realDate": "2026-07-02T13:40:00.000Z",
      "isRemoved": false,
      "isCreated": false,
      "isDiversion": false,
      "delay": 0,
      "isDelayed": false,
      "duration": 0
    }
  ],
  "stationUicCodes": {
    "departure": "87182063",
    "arrival": "87686006"
  },
  "trainId": "292"
}
```

#### Top-level fields

| Field | Type | Description |
|---|---|---|
| `number` | `string` | Train number (e.g. `"6706"`) |
| `trainId` | `string` | Internal train ID |
| `events` | `array` | Disruption / event messages (empty when nominal) |
| `onboardServices` | `string[]` | Service codes available onboard (see table below) |
| `additionalServices` | `object` | Extra services (usually empty) |
| `stops` | `Stop[]` | Ordered list of stops on the journey |
| `stationUicCodes.departure` | `string` | UIC code of departure station |
| `stationUicCodes.arrival` | `string` | UIC code of arrival/terminus station |

#### `Stop` object

| Field | Type | Description |
|---|---|---|
| `code` | `string` | Station code (SNCF internal, e.g. `"FRAEK"`) |
| `label` | `string` | Human-readable station name |
| `coordinates.latitude` | `number` | Station latitude |
| `coordinates.longitude` | `number` | Station longitude |
| `theoricDate` | `string` | Scheduled arrival/departure (ISO 8601 UTC) |
| `realDate` | `string` | Actual arrival/departure (ISO 8601 UTC) |
| `delay` | `number` | Delay in **minutes** |
| `isDelayed` | `boolean` | Whether the stop is delayed |
| `duration` | `number` | Dwell time at station in **minutes** (0 for origin/terminus) |
| `progress.progressPercentage` | `number` | 0–100, how far the train is past this segment |
| `progress.traveledDistance` | `number` | Distance traveled in this segment (meters) |
| `progress.remainingDistance` | `number` | Distance remaining to this stop (meters) |
| `isRemoved` | `boolean` | Stop has been removed from the journey |
| `isCreated` | `boolean` | Stop was added ad-hoc |
| `isDiversion` | `boolean` | Train is diverted through this stop |
| `services` | `object` | Per-stop services (e.g. `DRIVER: true`) |

#### Known onboard service codes

| Code | Likely meaning |
|---|---|
| `OCEHP` | Handicap / accessibility |
| `OCEWO` | Work area |
| `OCEPI` | Press / information |
| `OCECM` | Comfort |
| `OCENY` | Nursery |
| `OCEBA` | Bar / bistro car |
| `OCEWF` | WiFi |
| `OCEPP` | Power plugs |

---

### 3. `GET /router/api/train/graph`

GeoJSON `LineString` of the full train route (track geometry).

**Example response (truncated):**

```json
{
  "type": "LineString",
  "coordinates": [
    [7.342878, 47.741802],
    [7.340966, 47.740771],
    ...
    [2.378895, 48.842925],
    [2.37633, 48.844339]
  ]
}
```

| Field | Type | Description |
|---|---|---|
| `type` | `string` | Always `"LineString"` |
| `coordinates` | `[lon, lat][]` | Array of `[longitude, latitude]` pairs (GeoJSON order). Observed: **2330 points** for Mulhouse→Paris. |

> Useful for drawing the route on a map. Coordinates follow GeoJSON convention: **`[lon, lat]`**, not `[lat, lon]`.

---

### 4. `GET /router/api/connection/status`

Your WiFi connection status and data allowance.

```json
{
  "active": true,
  "status_code": 200,
  "status_description": "identifier has existing grant",
  "service_class": 5,
  "granted_bandwidth": 100000,
  "remaining_data": 962045,
  "consumed_data": 61954,
  "next_reset": 1783268884846,
  "profileId": "AUTO-LOGIN-PROFILE-ID"
}
```

| Field | Type | Description |
|---|---|---|
| `active` | `boolean` | Whether your connection is active |
| `service_class` | `number` | Service tier (5 observed for standard) |
| `granted_bandwidth` | `number` | Bandwidth allocation (units TBD) |
| `remaining_data` | `number` | Remaining data in your hourly fair-use quota |
| `consumed_data` | `number` | Data consumed this period |
| `next_reset` | `number` | Timestamp (ms) when the quota resets |
| `profileId` | `string` | Connection profile (e.g. `"AUTO-LOGIN-PROFILE-ID"`) |

---

### 5. `GET /router/api/connection/statistics`

Network quality and device count.

```json
{
  "quality": 5,
  "devices": 111
}
```

| Field | Type | Description |
|---|---|---|
| `quality` | `number` | Connection quality score (1–5) |
| `devices` | `number` | Number of devices connected to the onboard WiFi |

---

### 6. `GET /router/api/bar/attendance`

Bar/bistro car queue status.

```json
{
  "isBarQueueEmpty": false
}
```

---

### 7. `GET /router/api/configuration/modules`

Portal feature flags, map settings, UI configuration. Very large response (~16 KB). Key fields include:

| Key | Example | Description |
|---|---|---|
| `portal.status` | `"on"` | Portal active |
| `portal.journey.map.defaultZoom` | `"8"` | Default map zoom level |
| `portal.journey.map.helicopterZoom` | `"14"` | 3D helicopter view zoom |
| `portal.journey.map.helicopterPitch` | `"60"` | Helicopter view camera pitch |
| `portal.journey.map.routeUpcomingColor_dark` | `"#9C0C35"` | Upcoming route color (dark mode) |
| `portal.journey.map.routeTraversed_dark` | `"#C35E6D"` | Traversed route color (dark mode) |
| `portal.newFairUse.status` | `{isActive: true}` | Fair-use data limiting enabled |
| `portal.internet.autologin` | `"5"` | Auto-login delay (seconds?) |
| `portal.chat.status` | `"on"` | Onboard chat feature |
| `portal.streaming.status` | `{isActive: true}` | Live streaming feature |
| `portal.home.configuration` | `{FAMILY:[...], STANDARD:[...]}` | Home page card layout by service class |
| `portal.barNotification` | `{maxAge:150, duration:90, ...}` | Bar notification settings |
| `portal.tracking.ga` | `"UA-109129273-1"` | Google Analytics ID |

Also contains `portal.chat.intents` (chatbot intents in 5 languages), `portal.chat.quickMessages` (predefined chat messages), `portal.memory.difficulties` (memory game config), and `portal.chefabord.mission` (conductor mission train numbers).

---

### 8. `GET /router/api/media/videos`

Onboard video catalog (served locally, no internet needed).

Returns an array of video categories, each containing videos with multilingual poster/mp4 paths:

```json
[
  {
    "id": "inoui-coulisses",
    "title": "CATEGORY_INOUI-COULISSES_TITLE",
    "videos": [
      {
        "id": "Brut-CheffeDEscaleAGdn",
        "duration": "133",
        "files": {
          "fr": {
            "poster": "/videos/.../Brut-CheffeDEscaleAGdn.jpg",
            "video": "/videos/.../Brut-CheffeDEscaleAGdn.mp4"
          }
        }
      }
    ]
  }
]
```

### 9. `GET /router/api/media/wordings`

i18n keys for video titles and descriptions.

---

### 10. `GET /router/api/swagger.json`

Minimal Swagger 2.0 spec — just metadata, no route definitions:

```json
{
  "info": {
    "title": "kioskonboardrouter",
    "version": "1.27.39",
    "license": { "name": "ISC" },
    "description": "API /Routeur du portail Pepita à bord."
  },
  "swagger": "2.0"
}
```

> The API is called **kioskonboardrouter** v1.27.39, part of the **Pepita** onboard portal system.

---

### Static content JSON files

These are served as plain files (not through the `/router/api` prefix):

| Path | Description |
|---|---|
| `/bar/meta.json` | Bar car configuration |
| `/bistro/meta.json` | Bistro menu metadata |
| `/co2/meta.json` | CO₂ comparison data (train vs car/plane) |
| `/wifi/meta.json` | WiFi fair-use rules & i18n text (~12 KB) |
| `/premium/Tarifs.json` | Premium WiFi pricing |
| `/premium/Slider.json` | Premium promo carousel |
| `/sync/cards/meta.json` | All portal card definitions (~72 KB) |
| `/sync/faq/meta.json` | FAQ content |
| `/sync/kids/data.json` | Kids entertainment content |
| `/stationsConnections/stationsConnections.json` | Station connection info |
| `/mentions/data.json` | Legal mentions |
| `/mapViewMode/meta.json` | Map view configuration |

---

## Additional resources on the portal

### Map tiles

The portal serves offline map tiles (no internet needed):

| URL pattern | Description |
|---|---|
| `/sncf-maps/land/{z}/{x}/{y}.png` | Base land/terrain map tiles |
| `/sncf-maps/network/{z}/{x}/{y}.png` | Rail network overlay tiles |

Standard slippy map tile scheme (z/x/y). These work while onboard.

### WebSocket

The app config references a socket with namespaces:
- `/pepita` — default namespace
- `/chat` — chat namespace

Likely used for real-time push updates (chat feature, live notifications).

---

## External APIs referenced in `app.config.js`

These are **internet-facing** services (require connectivity beyond the train):

| Service                | URL                                       | Purpose                       |
| ---------------------- | ----------------------------------------- | ----------------------------- |
| Pepita / Kiosk API     | `https://pepita.vsct.fr/v1/kiosk-api/api` | SNCF network/kiosk backend    |
| StreamOnBoard / Moment | `https://sncf-api.streamonboard.com/`     | Onboard entertainment / media |
| Newrest (Bar)          | `https://api-v2.newrest.eu/api`           | Bistro car menu / ordering    |
| SNCF Auth              | `https://idp.sncf.fr/openam/oauth2/...`   | SSO login                     |
| Mon Identifiant SNCF   | `https://auth.monidentifiant.sncf`        | SNCF account auth (OIDC)      |

---

## Usage ideas

```bash
# Get current speed in km/h
curl -s https://wifi.sncf/router/api/train/gps | python3 -c \
  "import json,sys; d=json.load(sys.stdin); print(f'{d[\"speed\"]*3.6:.0f} km/h')"

# Get next stop
curl -s https://wifi.sncf/router/api/train/details | python3 -c \
  "import json,sys; stops=json.load(sys.stdin)['stops']
for s in stops:
  p=s.get('progress',{}).get('progressPercentage',0)
  if p < 100:
    print(f'Next: {s[\"label\"]} (ETA: {s[\"realDate\"]}, {s.get(\"delay\",0)} min delay)')
    break"

# Export route as GeoJSON file
curl -s https://wifi.sncf/router/api/train/graph \
  | python3 -c "import json,sys; print(json.dumps({'type':'Feature','geometry':json.load(sys.stdin),'properties':{}}))" \
  > route.geojson
```

---

## Getting train data from outside the train (public APIs)

The onboard `/router/api/train/gps` endpoint (live GPS coordinates, speed, heading) is **truly local-only** — there is no public equivalent. SNCF does not expose real-time vehicle positions for long-distance trains.

However, **schedule and delay data** equivalent to `/router/api/train/details` is available publicly via open-data feeds. No API key required.

### 1. GTFS-RT Trip Updates (Protobuf)

```
https://proxy.transport.data.gouv.fr/resource/sncf-gtfs-rt-trip-updates
```

- **Format:** Protocol Buffers (`application/x-protobuf`), ~1.6 MB
- **Auth:** None — completely open
- **Coverage:** All SNCF trains (TGV, Intercités, TER, Transilien) — ~2200+ active trip updates
- **Content:** Per-stop arrival/departure delay in seconds for every monitored train
- **Train ID format:** `OCESN{train_number}F1187_F:OUI:FR:Line::...::UIC_dep:UIC_arr:...:date`

**Example — finding train 6706:**

```python
from google.transit import gtfs_realtime_pb2
import urllib.request

feed = gtfs_realtime_pb2.FeedMessage()
resp = urllib.request.urlopen(
    "https://proxy.transport.data.gouv.fr/resource/sncf-gtfs-rt-trip-updates"
)
feed.ParseFromString(resp.read())

for entity in feed.entity:
    if entity.HasField('trip_update') and '6706' in entity.id:
        tu = entity.trip_update
        print(f"Trip: {tu.trip.trip_id}")
        for stu in tu.stop_time_update:
            print(f"  {stu.stop_id}: arr delay={stu.arrival.delay}s, dep delay={stu.departure.delay}s")
```

> `pip install gtfs-realtime-bindings`

### 2. SIRI Lite Estimated Timetable (XML)

```
https://proxy.transport.data.gouv.fr/resource/sncf-siri-lite-estimated-timetable
```

- **Format:** XML (SIRI), ~25 MB
- **Auth:** None
- **Coverage:** All monitored SNCF trains
- **Content:** Richer than GTFS-RT — includes stop names, aimed vs expected times, platform numbers, cancellation flags, origin/destination, train type (`highSpeedRail`, `regionalRail`, etc.)

**Example data for train 6706:**

```xml
<TrainNumbers>
  <TrainNumberRef>6706</TrainNumberRef>
</TrainNumbers>
<OriginName>Mulhouse</OriginName>
<DestinationName>Paris - Gare de Lyon - Hall 1 &amp; 2</DestinationName>
<ProductCategoryRef>FR:TypeOfProductCategory::highSpeedRail::</ProductCategoryRef>
<Monitored>true</Monitored>

<!-- Per stop: aimed vs expected times -->
<StopPointName>Dijon</StopPointName>
<AimedArrivalTime>2026-07-02T17:34:00+02:00</AimedArrivalTime>
<ExpectedArrivalTime>2026-07-02T17:34:00+02:00</ExpectedArrivalTime>
<ArrivalPlatformName>C</ArrivalPlatformName>
```

### 3. GTFS-RT Service Alerts (Protobuf)

```
https://proxy.transport.data.gouv.fr/resource/sncf-gtfs-rt-service-alerts
```

- **Format:** Protobuf, ~1.3 MB
- **Content:** Disruptions, cancellations, rerouting alerts

### 4. SIRI Lite Situation Exchange (XML)

```
https://proxy.transport.data.gouv.fr/resource/sncf-siri-lite-situation-exchange
```

- **Format:** XML, ~6.8 MB
- **Content:** Detailed disruption messages (same as service alerts but in SIRI XML format)

### 5. SNCF Navitia API (requires free API key)

```
https://api.sncf.com/v1/coverage/sncf/vehicle_journeys?headsign=6706
```

- **Auth:** Free API key from [numerique.sncf.com/startup/api](https://numerique.sncf.com/startup/api)
- **Content:** Schedules, journey planning, stop times, real-time disruptions (powered by Navitia/Hove)

### 6. Static GTFS Schedule

```
https://eu.ftp.opendatasoft.com/sncf/plandata/Export_OpenData_SNCF_GTFS_NewTripId.zip
```

- Full planned timetable for TGV, Intercités, TER — useful as baseline to apply GTFS-RT deltas against

### Summary: what's available where

| Data | Onboard API | Public feeds |
|---|---|---|
| **GPS position** (lat/lon/speed/heading) | ✅ `/train/gps` | ❌ Not available |
| **Route geometry** (track polyline) | ✅ `/train/graph` | ❌ Not available |
| **Stop list & schedule** | ✅ `/train/details` | ✅ GTFS-RT + SIRI |
| **Real-time delays** | ✅ `/train/details` | ✅ GTFS-RT + SIRI |
| **Platform numbers** | ❌ | ✅ SIRI only |
| **Cancellations / alerts** | ❌ | ✅ Service Alerts |
| **Onboard services** | ✅ `/train/details` | ❌ |
| **WiFi stats** (quality, device count) | ✅ `/connection/statistics` | ❌ |
| **Data quota** | ✅ `/connection/status` | ❌ |
| **Bar queue** | ✅ `/bar/attendance` | ❌ |
| **Portal config / feature flags** | ✅ `/configuration/modules` | ❌ |

---

## Notes

- **No auth** required for any `/router/api/` endpoint.
- **Local only** — `wifi.sncf` resolves to the onboard router; not accessible from the internet.
- **Public feeds** at `proxy.transport.data.gouv.fr` require no auth and cover all SNCF trains.
- Distances in the `details` response are in **meters**.
- Speed in the `gps` response is in **m/s**.
- The API is called **kioskonboardrouter** (v1.27.39), part of the **Pepita** portal system.
- The `stationUicCodes` use the international [UIC station code](https://en.wikipedia.org/wiki/List_of_UIC_country_codes) format — these match the stop IDs in the public SIRI/GTFS feeds (e.g. `87182063` = Mulhouse, `87686006` = Paris Gare de Lyon).

---

## Raw mock data

📂 **Project:** `file:///Users/ujperso/dev/wifi-sncf/`
📂 **Data:** `file:///Users/ujperso/dev/wifi-sncf/data/`

### Train 6706 — Mulhouse → Paris Gare de Lyon (2026-07-02)

| File | Description |
|---|---|
| `gps_timeseries/samples.jsonl` | ~158 GPS samples every 2s — final approach, 68→10 km/h |
| `details_timeseries/samples.jsonl` | ~32 details snapshots every 10s |
| `details.json` | Full `/train/details` response |
| `graph.json` | Route geometry (2330 coords) |
| `app.config.js` | Portal config |
| `journey_page.html` / `app.bundle.js` / `bundle.css` | Full frontend |
| `tiles/land/` + `tiles/network/` | 22+22 map tiles at zoom 10 |

### Train 6745 — Paris Gare de Lyon → Besançon Viotte (2026-07-05)

All files in `train_6745/`:

| File | Description |
|---|---|
| `details.json` | `/train/details` — different route, 5 stops |
| `graph.json` | Route geometry |
| `gps_snapshot.json` | GPS snapshot at departure |
| `connection_status.json` | WiFi connection & data quota |
| `connection_statistics.json` | Network quality + 111 devices connected |
| `bar_attendance.json` | Bar queue status |
| `configuration_modules.json` | Full portal config (~16 KB) |
| `media_videos.json` | Onboard video catalog |
| `swagger.json` | API metadata (kioskonboardrouter v1.27.39) |
| `static_*.json` | 18 static content files (bistro, CO₂, FAQ, cards, kids, wifi rules, etc.) |

### Differences observed between trains

| | Train 6706 | Train 6745 |
|---|---|---|
| `trainId` | `"292"` | `"729"` |
| Services | 8 codes | 10 codes (adds `OCEVP`, `OCEUB`) |
| Route | Mulhouse→Paris (6 stops) | Paris→Besançon (5 stops) |
| Graph points | 2330 | ~1800 |
