# Seasonal

Find out what is in season where you live, and whether the produce in front of you was grown nearby.

Seasonal is an open-source, offline-capable web app plus an open seasonality dataset. It starts with Switzerland.

## How it ranks origin

1. Grown in your canton
2. Grown in Switzerland
3. Grown within 400 miles (about 640 km)
4. Further away

Items that are likely air-freighted, or grown in heated greenhouses out of season, are flagged. "Local" is not automatically lower impact: how food is grown and transported matters as much as distance.

## Repository layout

- `data/` is the open seasonality dataset (YAML, one file per region). See [data/README.md](data/README.md).
- `app/` is the web app (PWA). Not started yet.
- `docs/` holds design notes and decisions.

## Licences

- Code: GNU GPL v3 (see [LICENSE](LICENSE)).
- Data in `data/`: Open Database License (ODbL 1.0), see [data/LICENSE.md](data/LICENSE.md).

## Contributing

See [CONTRIBUTING.md](CONTRIBUTING.md). Seasonality corrections for your region are the most useful contribution.
