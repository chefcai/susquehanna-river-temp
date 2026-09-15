# Susquehanna River Daily Temperature Alert

Automated daily push notification of water temperature at the Susquehanna River (Columbia, PA) delivered **at local sunrise** via [GitHub Actions](https://docs.github.com/en/actions) and [ntfy.sh](https://ntfy.sh). Zero infrastructure, zero cost.

## How It Works

1. **Triggers at 9 AM UTC** (5 AM EDT) every day — always before sunrise in Columbia, PA year-round
2. **Calculates today's exact sunrise** using the [sunrise-sunset.org API](https://api.sunrise-sunset.org) for the gauge coordinates (40.028°N, 76.518°W)
3. **Sleeps until sunrise**, then wakes and fetches the latest water temperature from [USGS Water Services](https://waterservices.usgs.gov) (site 01576003, parameter 00011)
4. **Converts °C → °F** and sends a push notification to [ntfy.sh/susquehanna-river-temp](https://ntfy.sh/susquehanna-river-temp)
5. **Sends an error alert** to the same topic if anything fails

---

## Subscribe to Notifications

### Mobile App (Recommended)
1. Install the [ntfy app](https://ntfy.sh/#subscribe) — [Android](https://play.google.com/store/apps/details?id=io.heckel.ntfy) | [iOS](https://apps.apple.com/us/app/ntfy/id1625396347)
2. Tap **+** → enter topic: `susquehanna-river-temp`
3. Tap **Subscribe** — done

### Web Browser
Visit [https://ntfy.sh/susquehanna-river-temp](https://ntfy.sh/susquehanna-river-temp) and click **Subscribe**

### Latest reading via curl
```bash
curl -s "https://ntfy.sh/susquehanna-river-temp/json?poll=1&since=1d" | jq -r '.message'
```

---

## USGS Monitoring Station

| Field | Value |
|---|---|
| Site name | Susquehanna River at Columbia, PA |
| USGS Site ID | 01576003 |
| Parameter | 00011 — Water temperature (°C → converted to °F) |
| Coordinates | 40.028°N, 76.518°W |
| Data interval | 15-minute readings |
| Live data | [waterdata.usgs.gov](https://waterdata.usgs.gov/monitoring-location/01576003/) |

---

## Customization

### Change the monitoring station
Edit `USGS_URL`, `LAT`, and `LON` in `.github/workflows/susquehanna-temp.yml`:
```yaml
USGS_URL="https://waterservices.usgs.gov/nwis/iv/?sites=YOUR_SITE_ID&parameterCd=00011&format=json"
LAT=YOUR_LAT
LON=YOUR_LON
```
Find USGS site IDs at the [USGS Water Resources Mapper](https://maps.waterdata.usgs.gov/mapper/index.html).

### Change the ntfy.sh topic
Replace every occurrence of `susquehanna-river-temp` in the workflow file with your preferred topic name.

### Change trigger time
The cron fires at `0 9 * * *` UTC. If you relocate to a different timezone, adjust so the job starts at least 2 hours before local sunrise.

---

## Troubleshooting

| Symptom | What to check |
|---|---|
| No notification today | [Actions runs](https://github.com/chefcai/susquehanna-river-temp/actions) — did the workflow trigger? |
| Workflow shows failure | Expand the **Fetch water temperature** step for USGS error details |
| Temperature looks wrong | Check [USGS live data](https://waterdata.usgs.gov/monitoring-location/01576003/) for sensor status |
| Notification at wrong time | Sunrise API was unreachable — job falls through immediately in that case |

---

## Manual Trigger (Test Now)

```bash
gh workflow run susquehanna-temp.yml \
  --repo chefcai/susquehanna-river-temp \
  -f skip_sunrise_wait=true
```

Or via GitHub UI:
1. [Actions → Susquehanna River Temperature Alert](https://github.com/chefcai/susquehanna-river-temp/actions/workflows/susquehanna-temp.yml)
2. **Run workflow** → check **Skip sunrise wait** → **Run workflow**

---

## License

MIT
