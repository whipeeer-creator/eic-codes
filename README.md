# European electricity bidding zones — EIC codes

A plain, machine-readable list of the **EIC codes** you need to query European
electricity market data: 45 bidding zones, the 13 ENTSO-E document types worth
knowing, and the 11 process types that go with them.

No dependencies, no API, no account. CSV and JSON, ~8 kB in total.

## Why this exists

EIC codes are published by ENTSO-E as a set of spreadsheets that mix bidding
zones, control areas, market participants and metering points into one file of
several thousand rows. If all you want is *"what do I put in `in_Domain` to get
German day-ahead prices"*, that file is the wrong shape.

This repository is the subset that answers that question, kept in the form that
a script can read.

## The data

| file | rows | what it is |
|---|---|---|
| [`data/bidding-zones.csv`](data/bidding-zones.csv) · [`.json`](data/bidding-zones.json) | 45 | bidding zones, with the traps noted |
| [`data/document-types.csv`](data/document-types.csv) · [`.json`](data/document-types.json) | 13 | `documentType` values (A44 day-ahead prices, A65 load, …) |
| [`data/process-types.csv`](data/process-types.csv) · [`.json`](data/process-types.json) | 11 | `processType` values (A01 day-ahead, A18 intraday, …) |

```csv
name,short,eic,note
Germany–Luxembourg,DE-LU,10Y1001A1001A82H,"One zone, four control areas"
Germany–Austria–Lux (historic),DE-AT-LU,10Y1001A1001A63L,Valid until 30 Sep 2018; needed for older series
```

## Use it

```python
import csv, urllib.request

url = ('https://raw.githubusercontent.com/whipeeer-creator/eic-codes/'
       'main/data/bidding-zones.csv')
rows = list(csv.DictReader(
    urllib.request.urlopen(url).read().decode().splitlines()))

zones = {r['short']: r['eic'] for r in rows}
print(zones['DE-LU'])      # 10Y1001A1001A82H
```

```bash
curl -s https://raw.githubusercontent.com/whipeeer-creator/eic-codes/main/data/bidding-zones.json \
  | jq -r '.[] | select(.short | startswith("IT")) | "\(.short)\t\(.eic)"'
```

Then, against the ENTSO-E Transparency Platform:

```
https://web-api.tp.entsoe.eu/api
  ?securityToken=YOUR_TOKEN
  &documentType=A44
  &in_Domain=10Y1001A1001A82H
  &out_Domain=10Y1001A1001A82H
  &periodStart=202601010000
  &periodEnd=202601020000
```

## The traps, in one place

These are the ones that actually cost people time:

- **DE-AT-LU vs DE-LU.** Germany, Austria and Luxembourg were one bidding zone
  until 30 September 2018. Any series that crosses that date needs *both*
  codes — `10Y1001A1001A63L` before, `10Y1001A1001A82H` after. A query using
  only the modern code returns an empty document for the older period, not an
  error.
- **Italy is six zones, not one.** North, Centre-North, Centre-South, South,
  Sicily and Sardinia. Calabria was split off from South in 2021, so a
  long series changes shape mid-way. The islands decouple often, and the PUN
  is a consumption-weighted average — not a price anything is settled at.
- **There is no single Nordic price.** Norway has five zones, Sweden four,
  Denmark two (DK1 is synchronous with continental Europe, DK2 with the Nordic
  system). Averaging them into a "country price" hides spreads of several
  hundred percent.
- **Ireland is one all-island market** (SEM) covering the Republic and Northern
  Ireland, so it does not line up with either country's grid operator.
- **Resolution is not constant.** Great Britain settles in half-hours. Poland
  moved to 15 minutes with the 2024 reform, and the continental day-ahead
  auction moved to 15-minute products in 2025. Code that assumes 24 hourly
  points per day will silently mis-align.

## Accuracy

Compiled by hand from the ENTSO-E Transparency Platform and checked against
live API responses. If you find a code that no longer resolves, or a zone that
is missing, open an issue — corrections are the point of publishing this.

Zone definitions change when markets split or merge. This is a snapshot, not a
feed.

## Licence

[CC0 1.0](LICENSE) — public domain. Use it in anything, no attribution
required. A link back is welcome but not asked for.

## Related

- [A practical guide to the ENTSO-E API](https://progrunners.com/entso-e-api/) —
  tokens, document types, resolutions and the errors that are not documented
- [Live European prices](https://progrunners.com/european-electricity-prices/) —
  today's numbers for 38 zones, with a national page per market:
  [Spain](https://progrunners.com/es/precio-luz-hoy/) ·
  [Germany](https://progrunners.com/de/strompreis-boerse/) ·
  [Austria](https://progrunners.com/at/strompreis-oesterreich/) ·
  [Estonia](https://progrunners.com/et/elektri-hind/) ·
  [Finland](https://progrunners.com/fi/sahkon-hinta/) ·
  [Sweden](https://progrunners.com/sv/elpriser-idag/) ·
  [Norway](https://progrunners.com/no/strompriser-i-dag/) ·
  [Denmark](https://progrunners.com/da/elpriser-i-dag/) ·
  [Lithuania](https://progrunners.com/lt/elektros-kaina/) ·
  [Latvia](https://progrunners.com/lv/elektribas-cena/) ·
  [Netherlands](https://progrunners.com/nl/stroomprijs/) ·
  [Poland](https://progrunners.com/pl/ceny-pradu/) ·
  [France](https://progrunners.com/fr/prix-electricite/) ·
  [Italy](https://progrunners.com/it/prezzi-zonali/) ·
  [Slovenia](https://progrunners.com/sl/cena-elektrike/) ·
  [Czechia](https://progrunners.com/cs/spotova-cena-elektriny/)
- Market data guides per country:
  [Germany](https://progrunners.com/market-data/germany/) ·
  [UK](https://progrunners.com/market-data/united-kingdom/) ·
  [France](https://progrunners.com/market-data/france/) ·
  [Spain](https://progrunners.com/market-data/spain/) ·
  [Italy](https://progrunners.com/market-data/italy/) ·
  [Poland](https://progrunners.com/market-data/poland/) ·
  [Nordics](https://progrunners.com/market-data/nordics/)

Maintained by [progrunners](https://progrunners.com/) — we build trading
dashboards and market data pipelines for European power markets.
