# Sun Position Camera Overlay — Implementation Plan

## Goal
Transform the Sunrise Tool from a dashboard-only app into an AR-capable tool that projects the sun's position onto a live camera view (mobile) or an interactive panoramic sky view (desktop), for any selected date/time at the user's location.

---

## Architecture Overview

```
┌─────────────────────────────────────────────────┐
│                   Header + Mode Nav             │
│              [Sun Position] [Analysis]           │
├─────────────────────────────────────────────────┤
│                                                 │
│   Sun Position Mode          Analysis Mode      │
│   ┌───────────────┐         ┌───────────────┐   │
│   │ Camera feed   │         │ Existing      │   │
│   │ OR Panoramic  │         │ charts,       │   │
│   │ canvas view   │         │ calendar,     │   │
│   │               │         │ summary       │   │
│   │ + Sun overlay │         │               │   │
│   │ + Sun path    │         │               │   │
│   │ + Compass     │         │               │   │
│   ├───────────────┤         │               │   │
│   │ Controls:     │         │               │   │
│   │ - Location    │         │               │   │
│   │ - Date/Time   │         │               │   │
│   │ - Camera tog. │         │               │   │
│   └───────────────┘         └───────────────┘   │
└─────────────────────────────────────────────────┘
```

---

## Implementation Steps

### Step 1: Restructure HTML — Mode Switcher
- Add navigation tabs: "Sun Position" (default) and "Yearly Analysis"
- Wrap existing analysis HTML in `<div id="analysis-view">`
- Create new `<div id="sun-view">` container for the AR/panoramic view
- Add mode switching logic (show/hide views)

### Step 2: Full Sun Position Algorithm
- Implement `getSunPosition(date, lat, lng)` → `{ azimuth, altitude }` for ANY time
  - Uses simplified Meeus algorithm (accurate to ~1°)
  - Julian date calculation
  - Solar ecliptic longitude, right ascension, declination
  - Local sidereal time → hour angle
  - Horizontal coordinates (altitude + azimuth)
- Implement `getSunPath(date, lat, lng)` → array of positions every 10 min
- Implement `findSunTimes(path)` → `{ sunrise, sunset, solarNoon, maxAltitude }`

### Step 3: Interactive Panoramic Sky View (Canvas)
- Full-viewport canvas showing a 360° pannable horizon view
- Render: sky gradient (changes with sun altitude), ground, horizon line
- Render: terrain profile silhouette (from elevation API data)
- Render: cardinal direction labels (N, NE, E, SE, S, SW, W, NW)
- Render: degree markers every 10° along horizon
- Render: sun path arc (dotted golden line from sunrise to sunset)
- Render: hour markers along the path with time labels
- Render: sun as a glowing circle at the correct position
- Mouse/touch drag to pan the view heading
- Field of view: ~90° horizontal, ~60° vertical
- This is the primary view — works on all devices

### Step 4: Date/Time Controls
- Date picker (defaults to today)
- Time slider (0:00–23:59, 1-minute resolution)
- Digital time display synced with slider
- "Now" checkbox — auto-updates time to current moment
- Display: sunrise time, sunset time, solar noon
- Sun info panel: current altitude, azimuth, compass direction

### Step 5: Location Controls + GPS
- GPS auto-detect button (`navigator.geolocation`)
- Manual lat/lng inputs (reuse existing pattern)
- "Go" button to apply manual coordinates
- Location persists across mode switches
- Fetch horizon profile when location changes (reuse existing API)

### Step 6: Camera AR Overlay (Mobile)
- "Camera" toggle button (only shown if `getUserMedia` is available)
- Rear camera feed via `<video>` element
- Canvas overlay on top of video (transparent background)
- Device orientation API for automatic heading/pitch tracking
  - iOS 13+: request permission via `DeviceOrientationEvent.requestPermission()`
  - Compass heading (alpha) → view azimuth
  - Pitch (beta) → view elevation angle
- Project sun position onto screen using device orientation + estimated camera FOV
- Draw sun, sun path, compass indicators on overlay
- Fallback: if no orientation sensors, camera shows with manual pan controls

### Step 7: Polish & Integration
- Smooth animations and transitions between modes
- Responsive design (mobile-first for camera, desktop for panoramic)
- Share location state between Sun Position and Analysis views
- Loading states for GPS and elevation data
- Error handling (camera denied, GPS unavailable, etc.)
- Update build version

### Step 8: Git — Commit & Push
- Commit all changes
- Push to branch `claude/sun-position-camera-overlay-pM2en`

---

## Key New Algorithms

### Sun Position (Simplified Meeus)
```
Input:  date (with time), latitude, longitude
Output: { azimuth: 0-360°, altitude: -90°..+90° }

1. Calculate days since J2000.0 (Jan 1, 2000 12:00 UTC)
2. Mean solar longitude L = (280.460 + 0.9856474 * n) mod 360
3. Mean anomaly g = (357.528 + 0.9856003 * n) mod 360
4. Ecliptic longitude = L + 1.915*sin(g) + 0.020*sin(2g)
5. Obliquity = 23.439 - 0.0000004*n
6. Right ascension RA = atan2(cos(obl)*sin(ecl), cos(ecl))
7. Declination = asin(sin(obl)*sin(ecl))
8. GMST = (280.461 + 360.986*n) mod 360
9. Hour angle HA = LST - RA  (where LST = GMST + longitude)
10. Altitude = asin(sin(lat)*sin(decl) + cos(lat)*cos(decl)*cos(HA))
11. Azimuth = atan2(-cos(decl)*sin(HA), sin(decl)*cos(lat) - cos(decl)*sin(lat)*cos(HA))
```

### Screen Projection (AR Mode)
```
Input:  sunAzimuth, sunAltitude, deviceHeading, devicePitch, cameraFOV
Output: { screenX, screenY, visible }

1. relativeAzimuth = normalize(sunAzimuth - deviceHeading)  // -180..180
2. relativeAltitude = sunAltitude - devicePitch
3. screenX = canvasWidth/2 + (relativeAz / hFOV) * canvasWidth
4. screenY = canvasHeight/2 - (relativeAlt / vFOV) * canvasHeight
5. visible = |relativeAz| < hFOV/2 && |relativeAlt| < vFOV/2
```

---

## Dependencies (No New Libraries)
- All new code is vanilla JS (same as existing)
- Camera: `navigator.mediaDevices.getUserMedia`
- GPS: `navigator.geolocation.getCurrentPosition`
- Orientation: `DeviceOrientationEvent`
- Rendering: Canvas 2D API
- Existing: Leaflet (analysis map), Chart.js (analysis charts)

---

## Progress Tracker
- [ ] Step 1: Restructure HTML — Mode Switcher
- [ ] Step 2: Full Sun Position Algorithm
- [ ] Step 3: Interactive Panoramic Sky View
- [ ] Step 4: Date/Time Controls
- [ ] Step 5: Location Controls + GPS
- [ ] Step 6: Camera AR Overlay
- [ ] Step 7: Polish & Integration
- [ ] Step 8: Git — Commit & Push
