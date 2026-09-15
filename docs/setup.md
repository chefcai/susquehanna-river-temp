# Setup Guide

## Prerequisites

- GitHub account (to fork or trigger the workflow)
- [ntfy.sh](https://ntfy.sh) app on your device — free, no account needed

---

## 1. Subscribe to Notifications

### Android / iOS
1. Download the ntfy app:
   - [Android (Google Play)](https://play.google.com/store/apps/details?id=io.heckel.ntfy)
   - [iOS (App Store)](https://apps.apple.com/us/app/ntfy/id1625396347)
2. Tap **+** → Topic name: `susquehanna-river-temp` → **Subscribe**
3. Enable notifications when prompted

### Web browser (desktop)
1. Visit [https://ntfy.sh/susquehanna-river-temp](https://ntfy.sh/susquehanna-river-temp)
2. Click **Subscribe** and allow browser notifications

---

## 2. Verify the Topic Works

Send a test message to confirm your subscription is active:

```bash
curl -H "Title: Test" \
     -d "Test notification — setup complete" \
     https://ntfy.sh/susquehanna-river-temp
```

You should receive a push notification within seconds.

---

## 3. Trigger the Workflow Now (Skip Sunrise Wait)

The workflow normally sleeps until sunrise. To test immediately:

**Via GitHub CLI:**
```bash
gh workflow run susquehanna-temp.yml \
  --repo chefcai/susquehanna-river-temp \
  -f skip_sunrise_wait=true
```

**Via GitHub UI:**
1. Go to [Actions → Susquehanna River Temperature Alert](https://github.com/chefcai/susquehanna-river-temp/actions/workflows/susquehanna-temp.yml)
2. Click **Run workflow** (top right)
3. Check **Skip sunrise wait** → **Run workflow**
4. Watch the run complete in ~30 seconds

---

## 4. Local API Testing

Test the USGS data fetch:
```bash
curl -s "https://waterservices.usgs.gov/nwis/iv/?sites=01576003&parameterCd=00011&format=json" | \
  jq '{
    site: .value.timeSeries[0].sourceInfo.siteName,
    latest: (.value.timeSeries[0].values[0].value | sort_by(.dateTime) | last)
  }'
```

Test the sunrise calculation:
```bash
curl -s "https://api.sunrise-sunset.org/json?lat=40.0284308&lng=-76.5177432&formatted=0" | \
  jq '{sunrise: .results.sunrise, sunset: .results.sunset}'
```

Full workflow simulation (bash):
```bash
RESPONSE=$(curl -s "https://waterservices.usgs.gov/nwis/iv/?sites=01576003&parameterCd=00011&format=json")
TEMP_C=$(echo "$RESPONSE" | jq -r '.value.timeSeries[0].values[0].value | sort_by(.dateTime) | last | .value')
TEMP_F=$(echo "scale=1; $TEMP_C * 9 / 5 + 32" | bc)
echo "Current temp: ${TEMP_F}F (${TEMP_C}C)"
```

---

## 5. Monitoring Ongoing Runs

- **GitHub Actions runs**: [github.com/chefcai/susquehanna-river-temp/actions](https://github.com/chefcai/susquehanna-river-temp/actions)
- **USGS live station**: [waterdata.usgs.gov/monitoring-location/01576003/](https://waterdata.usgs.gov/monitoring-location/01576003/)
- **ntfy.sh topic**: [ntfy.sh/susquehanna-river-temp](https://ntfy.sh/susquehanna-river-temp)

---

## 6. Customization

### Use a different USGS station
1. Find your station at the [USGS Water Resources Mapper](https://maps.waterdata.usgs.gov/mapper/index.html)
2. In `.github/workflows/susquehanna-temp.yml`, update:

```yaml
LAT=YOUR_STATION_LAT
LON=YOUR_STATION_LON
USGS_URL="https://waterservices.usgs.gov/nwis/iv/?sites=YOUR_SITE_ID&parameterCd=00011&format=json"
```

### Use a private ntfy.sh topic
1. Choose a long/random topic name (e.g. `susky-temp-a7x92k`)
2. Replace `susquehanna-river-temp` everywhere in the workflow file
3. Subscribe using the new name

### Add email delivery
```bash
curl -H "Email: your@email.com" \
     -H "Title: River Temp" \
     -d "Water temp: ${TEMP_F}F" \
     https://ntfy.sh/susquehanna-river-temp
```
See [ntfy.sh email docs](https://docs.ntfy.sh/publish/#e-mail-notifications).

---

## 7. Troubleshooting

**Workflow not triggering on schedule**
- GitHub Actions cron can delay up to 30 minutes under load; check the Actions tab
- Ensure the workflow file is on the default branch

**USGS API returns no data**
- USGS occasionally has sensor outages; check [USGS Water Alerts](https://water.usgs.gov/)
- The failure step sends a notification to the ntfy.sh topic automatically

**Sunrise API unavailable**
- The workflow falls through immediately with a warning and still delivers the temperature

**bc not found on runner**
- bc is pre-installed on ubuntu-latest; no action needed
