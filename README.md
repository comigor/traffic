# Live Traffic ETA Kiosk

A single-file, zero-dependency public display kiosk for showing live Google Maps ETAs from a central HQ to a list of places. Designed for wall displays with a dark, transit-board aesthetic.

## How to run locally

No build step required. Just serve the directory:

```bash
python3 -m http.server 8000
```

Then open your browser to:
`http://localhost:8000/?key=YOUR_GOOGLE_MAPS_API_KEY`

## Configuration

Edit the top of the `<script>` tag in `index.html` to configure your `HQ` and `PLACES`:

```javascript
const HQ = { name: 'HQ', address: '1600 Amphitheatre Parkway, Mountain View, CA' };
const PLACES = [
  { name: 'San Francisco', address: 'San Francisco, CA', dir: 'both' },
  // ...
];
```

The `dir` property can be `'from-hq'`, `'to-hq'`, or `'both'`. This dictates which directions are requested and displayed for that place.

## Refresh schedule

The kiosk uses a time-window schedule in **BRT (America/Sao_Paulo)**. Windows and intervals are hardcoded in the `SCHEDULE` constant near the top of `index.html` and are easy to edit.

| Window | Hours (BRT) | Interval |
|---|---|---|
| Peak | 08:00 – 09:30 | `?refresh=` param (default 10 min) |
| Peak | 11:30 – 13:00 | `?refresh=` param (default 10 min) |
| Peak | 18:30 – 21:00 | `?refresh=` param (default 10 min) |
| Off-peak | all other hours within 06:00 – 22:00 | 60 min (hardcoded) |
| Sleep | 22:00 – 06:00 | no API calls |

During the sleep window the kiosk shows the last known data and displays "PAUSED / Resumes 06:00 BRT" in the top bar.

## URL Parameters

*   `key` (Required): Your Google Maps API Key.
*   `refresh`: Peak-window refresh interval in minutes (default `10`, clamped 1–30). Has no effect on the off-peak 60-minute cadence or the sleep window.
*   `units`: Set to `imperial` or `metric`. Default is `metric`.

Example:
`/?key=AIza...&refresh=5&units=imperial`

## Google Maps API Key Setup

1. Go to the [Google Cloud Console](https://console.cloud.google.com/).
2. Enable the **Maps JavaScript API** and the **Distance Matrix API**.
3. Generate an API Key.
4. **Important**: Restrict your API key to prevent unauthorized use. Under "Application restrictions", choose "HTTP referrers" and add your GitHub Pages domain (e.g., `https://yourusername.github.io/traffic/*`).

## Estimated API cost

Each refresh calls Distance Matrix with 9 elements (origins × destinations across both directions). At $10 per 1,000 elements:

- **Peak hours** (3 windows totalling ~4.5 h/day, 10-min interval): ~27 refreshes × 9 elements = 243 elements/day
- **Off-peak** (active hours outside peak, ~11.5 h/day, 60-min interval): ~11 refreshes × 9 elements = 99 elements/day
- **Sleep** (22:00 – 06:00): 0 elements

**Total: ~342 elements/day ≈ $0.0034/day (~$1.03/year).**
