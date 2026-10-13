# Seasonality dataset

## Files

- `crops.yaml` lists canonical produce items, with names in en/de/fr/it and PLU codes where known.
- `regions/<country>.yaml` holds seasonality for a country, with optional per-canton/state overrides.

## Seasonality format

Months are 1 to 12. Each item in a region has:

- `field`: months it is harvested outdoors (the "in season" signal).
- `storage`: months it is commonly sold from local storage (apples, potatoes, carrots in winter).
- `heated_greenhouse`: months when local supply typically comes from heated greenhouses (flagged in the app).
- `sources`: keys into the region's `sources` list. Every entry needs at least one.
- `status`: `draft` until checked against a cited source, then `verified`.

Seasonality calendars we cite are generally not openly licensed. Record the facts (which months) in your own words and cite the source; don't copy tables or text.
