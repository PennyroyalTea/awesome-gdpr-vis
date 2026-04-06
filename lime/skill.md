# Lime Data Visualiser -- Skill Guide

Instructions for Claude to build interactive dashboards from a Lime GDPR data export.

## Context

The user has requested their personal data from Lime (ebike/scooter company) via GDPR Subject Access Request. The export typically contains:

- **Trip history CSV**: every ride with status, start/end timestamps, start/end GPS coordinates (lat/lng), distance in meters, cost breakdown (actual, base, discount, refund cents), currency, rating, review
- **Payment history CSV**: every charge with amount in cents, currency, category (subscription_cost, trip_cost, settle_arrears, auth_capture), status, order type
- **App event log CSV**: every in-app interaction with event type, timestamp, IP address, IP-derived city, device model, OS version, platform
- **User profile CSV**: name, email, payment card details, sign-up date, per-currency wallets
- **CRM profile TSV**: internal segmentation fields (rider tier, loyalty scores, RFM, commuter classification, A/B experiment assignments)
- **Identity verification PDFs**: selfie photos and government ID scans

## Approach

Build everything as plain HTML + JS files with CDN dependencies (no build step, no npm). Use a Python script to preprocess CSVs into a single `data.json`, then have the HTML pages fetch it.

### Step 1: Data pipeline

Write a `build.py` that:
- Reads the trip CSV and payment CSV
- Filters to completed rides in the user's primary currency/city
- Filters out rides with missing coordinates or zero distance
- Computes aggregated data: monthly rides, weekday/hourly distributions, cumulative distance, speed stats, calendar data, spending breakdown
- Also processes the app event log for surveillance-related visualisations
- Outputs everything as a single `data.json`

### Step 2: Main dashboard (`index.html`)

Dark-themed dashboard using Chart.js (CDN) and Leaflet (CDN) with:
- Top stats row: total rides, distance, hours, spending, savings, streak, averages
- Map with ride start points (green) and end points (red) using Leaflet canvas renderer for performance
- Monthly rides bar chart
- Day of week and hour of day bar charts
- Cumulative and monthly spending (dual y-axis: monthly bars + cumulative line)
- Trip distance and duration distribution histograms

### Step 3: Interactive map (`map.html`)

Advanced map using deck.gl (CDN) + MapLibre (CDN) with multiple visualisation modes, switchable via buttons or keyboard shortcuts:

1. **Arcs** -- 3D curved arcs from ride start to end, green-to-red gradient
2. **Time of day** -- arcs coloured by hour (gold morning, blue midday, red evening, purple night)
3. **Density lines** -- flat semi-transparent straight lines; overlapping corridors glow brighter
4. **Heatmap** -- deck.gl HeatmapLayer of all start + end points
5. **3D Hexbin** -- extruded hexagonal columns showing ride density per area
6. **Animated** -- rides appear and fade through a 24-hour cycle, grouped by hour with trailing opacity

Additional features:
- 2D/3D view toggle (controls pitch independently of mode)
- Light/dark basemap toggle (CARTO dark-matter / positron tiles, no API key needed)
- Screenshot capture (read WebGL canvas, composite all layers, download as PNG)
- GIF recording for animated mode (step through hours, capture each frame, encode with gif.js)
- Data filter toggle (e.g. pre/post move date)

### Step 4: Deep visualisations (`viz.html`)

Additional analysis charts:
- **Calendar heatmap**: GitHub contributions style, every day coloured by ride count, grouped by year
- **Weekly rhythm heatmap**: 7x24 grid (Mon-Sun x 00-23h), cell colour = ride count. Side-by-side for rides and app events
- **Cumulative distance**: line chart showing total km over time
- **Speed trend**: monthly avg/median/p90/max speed lines, with commute-only max overlay. Useful for detecting throttling (speed limit changes show as sudden drops in max/p90)
- **Daily rhythm**: two lines showing monthly average first departure and last return home (weekdays). The gap between them = time spent out. Reveals work pattern changes
- **Personal records timeline**: vertical timeline of milestones (longest ride, fastest ride, busiest day, distance milestones, ride count milestones)
- **App event surveillance**: top event types bar chart, hourly distribution, city attribution from IPs, daily unique IP count

## Analysis prompts

Once the data is loaded, the user can ask Claude to derive insights. Key analyses:

**Location identification:**
- "Where do I live?" -- cluster morning weekday ride starts + late evening ride ends
- "Where do I work?" -- cluster morning weekday ride ends + evening weekday ride starts (excluding home)
- "When did I move/change jobs?" -- track top clusters by month, look for abrupt shifts
- "What are my regular spots?" -- find top non-home/work locations, categorise by time patterns:
  - Every day 5-8pm = gym
  - Weekend mornings = brunch spot
  - Single weekday, fixed time = recurring appointment
  - Late night departures = friend's place or pub
  - Weekend afternoons = leisure/park

**Ride patterns:**
- Speed analysis: monthly avg/p90/max, check for sudden drops (throttling evidence)
- Commute consistency: filter to home<->work rides, measure duration std dev
- Seasonal patterns: rides per calendar month averaged across years
- Holiday detection: gaps > 7 days between ride days
- Wake/sleep proxy: first ride from home and last ride to home, by month
- Work-from-home detection: weekdays with rides but no work location visits

**Financial:**
- Subscription ROI: monthly base_cost / actual_cost ratio
- Subscription evolution: track which pass types were purchased over time
- Overage analysis: how much extra beyond subscription
- Transport comparison: estimated equivalent costs for Tube, car, Uber

**Surveillance:**
- App event types: what Lime tracks (209+ distinct event types in a typical export)
- IP logging: unique IPs per day, subnet analysis
- Missing data: GPS traces exist (distance is calculated) but aren't included in the export

## Technical notes

- Use CARTO basemap tiles (free, no API key): `https://basemaps.cartocdn.com/gl/dark-matter-nolabels-gl-style/style.json`
- For map performance with 2000+ points, use deck.gl's WebGL rendering or Leaflet's canvas renderer
- Chart.js `is:inline` scripts for Astro components (not module scripts)
- For theme support: use `MutationObserver` on `data-theme` attribute to re-render charts on toggle
- WebGL canvas capture requires `preserveDrawingBuffer: true` on context creation
- For GIF recording: use gif.js with a local worker file (CDN worker fails due to CORS)
