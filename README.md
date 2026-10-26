# Tarrant Well Explorer

A read-only, static browser app built from the Tarrant County RRC SQLite snapshot. The map, filters and well-ID search run against `dist/wells.json`; no API or real-time sensor data is implied. Address lookup uses OpenStreetMap Nominatim on explicit submission; map tiles are from OpenStreetMap and Leaflet assets use public CDNs. Internet access is required for the map and geocoder.

## Map setup and API keys

**The current app does not need a map API key.** It uses Leaflet and marker clustering from unpkg, OpenStreetMap standard basemap tiles with OpenStreetMap attribution, and the public Nominatim service for submitted address searches. No Google Cloud account, billing setup, `.env` file, or key entry is required for this version. Run it from an HTTP server, not by opening `index.html` directly:

```sh
cd tarrant-map
python3 -m http.server 8000 --directory dist
```

Open <http://localhost:8000>. The browser needs internet access to load the JavaScript libraries, map tiles and address-search service. If the map is blank, check the browser console and Network tab for blocked or failed requests to `unpkg.com` and `tile.openstreetmap.org`; also check that `wells.json` loads. If address search fails, check `nominatim.openstreetmap.org`; well-ID search still works without the geocoder. RRC data and tiles are separate, so loading the well records alone will not provide a basemap.

The public [Nominatim usage policy](https://operations.osmfoundation.org/policies/nominatim/) limits aggregate application traffic to an absolute maximum of one request per second and has additional identification and usage requirements. The current browser-only prototype does not enforce a shared rate limit across users. Before wider deployment, choose a suitable hosted geocoding provider or self-host Nominatim, implement any needed rate limiting and attribution, and review the [OpenStreetMap tile policy](https://operations.osmfoundation.org/policies/tiles/). The tile service is best-effort, so a production application should evaluate a provider with appropriate capacity and service guarantees.

### If you choose Google Maps later

Google Maps is **not integrated** in this app. Creating a key alone will not change its Leaflet/OpenStreetMap map; a developer must replace the map integration, update markers and clustering, and integrate any desired Google address search.

1. Create or select a project in [Google Cloud Console](https://console.cloud.google.com/), attach a billing account, and enable the **Maps JavaScript API**. For address search, also enable the appropriate Google Places or Geocoding API if the chosen implementation uses it. Google also offers a limited demo-key option for prototyping; see its [setup guide](https://developers.google.com/maps/documentation/javascript/get-api-key).
2. In **Google Maps Platform → Keys & Credentials**, create an API key. Set application restrictions to **Websites** and add the actual local and deployed origins/referrer patterns, following Google's instructions. Restrict the key to only the APIs the application uses.
3. Pass the browser key to the Google Maps JavaScript loader as directed by [Google's loading guide](https://developers.google.com/maps/documentation/javascript/load-maps-js-api). Browser keys are visible to visitors; use referrer and API restrictions, quotas and billing alerts rather than treating them as secrets. Keep any server-only credentials out of browser code and source control.
4. Review [usage and billing](https://developers.google.com/maps/documentation/javascript/usage-and-billing) and applicable terms before switching providers. Configure the new provider's required attribution and test local and deployed origins.

## Refresh the data

To refresh the data export after rebuilding the snapshot, run from a checkout of `texas-well-platform` with this repo at `texas-well-platform/tarrant-map` (the SQLite snapshot lives in that repo's `datasets/tarrant/`, not here):

```sh
python3 -c 'import sqlite3,json; c=sqlite3.connect("../datasets/tarrant/tarrant_wells.sqlite"); c.row_factory=sqlite3.Row; json.dump([dict(r) for r in c.execute("select * from wells")],open("dist/wells.json","w"),separators=(",",":"))'
```

The included snapshot was fetched on 2026-10-21; see `datasets/tarrant/README.md` in `texas-well-platform` for sources, identity heuristics and limitations. A production service for ongoing refreshes, access-controlled observations, a contractual geocoder/tile provider, and sensor ingestion are future work.

## Deployment

Live at **https://wells.sigilint.com**, also served at `https://sigilint-tarrant-map.netlify.app`. It is the Netlify site `sigilint-tarrant-map`, which deploys automatically on every push to `main` of `Sigilint/tarrant-map`. Build settings are in `netlify.toml`: no build step, and `dist/` is published. See `DEPLOYMENT.md` in `Sigilint/sigilint-web` for how all the Sigilint sites are hosted.
