[README.md](https://github.com/user-attachments/files/32960051/README.md)
# Heating oil tracker

A single-file web app for tracking domestic heating oil: gauge readings, refills, prices, consumption, temperatures and forecasts. It runs in any modern browser and has no server, account or build step.

File: `heating-oil-tracker.html`

---

## Contents

1. [Quick start](#quick-start)
2. [Running it on your devices](#running-it-on-your-devices)
3. [Where your data is stored and how to back it up](#where-your-data-is-stored-and-how-to-back-it-up)
4. [Entering data](#entering-data)
5. [What the app shows](#what-the-app-shows)
6. [Settings](#settings)
7. [How the calculations work](#how-the-calculations-work)
8. [Temperatures and degree days](#temperatures-and-degree-days)
9. [Data checks](#data-checks)
10. [Import, export and the data format](#import-export-and-the-data-format)
11. [Initial data from the spreadsheet](#initial-data-from-the-spreadsheet)
12. [Limitations](#limitations)
13. [Troubleshooting](#troubleshooting)
14. [Technical notes](#technical-notes)

---

## Quick start

1. Open `heating-oil-tracker.html` in a browser (Chrome, Edge, Firefox or Safari). It opens with all readings and refills from `Petrelaio.xlsx` already loaded.
2. Open **Settings**, search for your town under **Location for temperatures**, choose it and click **Save settings**. Temperatures download automatically.
3. After each gauge reading or delivery, add it under **Add a gauge reading** or **Add a refill**.
4. Click **Export data** regularly and keep the file on Google Drive. This is your only real backup.

An internet connection is needed the first time the page loads (for the chart library and fonts) and whenever temperatures update. Entering data, the tables and the calculations work offline.

---

## Running it on your devices

| Where | How |
|---|---|
| Computer | Download the HTML file and open it in a browser. Everything works, including saving. |
| Phone or tablet | Phones are unreliable at opening local HTML files. The simplest option is to host the file for free: drag it onto <https://app.netlify.com/drop>, then open the link it gives you and optionally add it to the home screen. Upload the file again whenever it changes. |
| Google Drive or Google Docs | Neither can run the app. Drive shows a preview or the source code. Use Drive only to store the HTML file and your exported backups. |
| Claude's preview panel | It works, but nothing is saved there; a red banner says so. Use it only to try the app. |

Each browser on each device keeps its own data. To move data between devices, export on one and import on the other (see [Import, export](#import-export-and-the-data-format)).

---

## Where your data is stored and how to back it up

The app stores data in the browser's **local storage**, under these keys:

| Key | Contents | Exported? |
|---|---|---|
| `heatingOilTracker.v1` | Readings, refills and settings | Yes, this is the export |
| `heatingOilTracker.v1.backup` | Backup reminder: last export time and number of changes since | No |
| `heatingOilTracker.v1.weather` | Downloaded temperatures, tied to your location | No, they can be downloaded again |
| `heatingOilTracker.v1.unreadable.<time>` | A copy of saved data that failed the validation check on load (see below) | No |

**Local storage is not a backup.** It can be lost in several ways:

- clearing browsing data, cookies or site data, or running a cleanup tool;
- using private or incognito windows, where it's deleted when the window closes;
- Safari (iPhone, iPad, Mac), which can delete a site's data after 7 days of using Safari without visiting that site;
- low disk space, when the browser may evict site data;
- a different browser, device or profile, or moving or renaming the HTML file or changing where it's hosted;
- a lost or broken device.

**Backup routine:** after each session where you add or change data, click **Export data** and save the JSON file to Google Drive. If data is lost, open the app and **Import data** with the latest file.

The **backup reminder** under the header shows the state at all times:

- "Not exported yet": no export has been made in this browser.
- "*N* changes since the last export …": highlighted, and the Export button turns dark.
- "All changes exported. Last export …"

Adding, editing or deleting a reading or refill counts as a change, as do saving settings with different values and restoring the spreadsheet data. Importing a file resets the count; "last export" then shows the time stored in that file. The browser can't report whether a download was actually saved, so the count resets as soon as Export is clicked. Make sure the file reaches Drive.

**Unreadable saved data:** if the data saved in the browser fails the validation check on load, the app never discards it silently. It keeps a copy under an `.unreadable.<time>` key, shows the spreadsheet data instead, and displays a banner with a **Download the unreadable data** button, so the file can be fixed and imported.

---

## Entering data

### Gauge reading

| Field | Notes |
|---|---|
| Date | Defaults to today. Future dates are rejected. |
| Outside gauge (cm) | The value as read on the outside gauge, before the offset. 0 to 500. A decimal comma or point both work: `37,5` or `37.5`. |
| Note | Optional, up to 140 characters in the form. |

### Refill

| Field | Notes |
|---|---|
| Date | Delivery date. Future dates are rejected. |
| Litres | Litres delivered. Required in the form. |
| Price €/L | Required. |
| Total € | Optional. Empty means litres × price (the placeholder shows the value). Enter the invoice total if it differs, for example because of rounding or fees. |
| Gauge after fill (cm) | Optional. If filled in, a reading with the same date is saved too. Recommended: it lets the app check the delivery. |
| Note | Optional. |

Press **Enter** in any field to save. Every save or delete shows a confirmation message, including how many changes haven't been exported yet. **Edit** and **Delete** are in the Readings and Refills tables; deleting asks for confirmation.

**Good practice:** take a reading just before and just after each refill, and read the gauge every few days in the heating season. Frequent readings make the rates, the temperature model and the delivery check much more accurate.

---

## What the app shows

### Tank panel

- **Current level**: litres, cm on the gauge and % full. It's measured if a reading exists for today, otherwise estimated from the last reading (see [Current level](#current-level-and-reorder-forecast)).
- **Reorder level reached**: the date the tank is expected to reach your reorder level, days remaining, the expected empty date, the litres needed to fill up, and the estimated cost at the trend price. It turns red within 14 days. It shows **Reorder now** if the estimate is already at or below the reorder level.
- **Stats**: recent use (L/day over the last measured gap), last price paid, and the litres and cost forecast for the current season.

### Charts

| Chart | Shows |
|---|---|
| Tank level and forecast | Real litres from readings (solid), estimate since the last reading and forecast (dashed), refill markers, reorder line |
| Litres per season | Recorded and projected litres, July to June |
| Cost per season | Money spent on refills, and consumption cost including the forecast |
| Price history and trend | Prices paid, linear trend 12 months ahead, ±1 standard deviation band |

### Consumption and temperature tabs

| Tab | Shows |
|---|---|
| **Cumulative** | Litres used since 1 July per season. Dashed lines are the forecast to season end. |
| **Daily rate** | Measured L/day between readings as steps, one line per season. Dotted parts are refill gaps, where use is estimated. Pick a season to add its temperature (7-day average). **Smoothed (14-day average)** shows a centred 14-day average instead; it's off by default. |
| **By month** | Average L/day per calendar month per season, with monthly mean temperature. Pick a season to compare it with the all-season average. Months with fewer than 7 recorded days are left out. |
| **Season average** | Average L/day from 1 October to 30 April per season, with that period's average temperature. |
| **Per degree day** | Litres per degree day per season (bars) and how cold each heating period was in degree days (line). See [Degree days](#degree-days). |
| **Use vs temperature** | Each gap between readings as a bubble: average temperature against L/day, with bigger bubbles for longer gaps. Includes the fitted model line. |

A `*` after a season label means it's only partly recorded. The note under the chart lists the number of recorded days.

### Season summary table

For each season (July to June): recorded litres, projected litres, total, litres bought, money spent, average price paid, consumption cost, and for 1 October to 30 April the average L/day, average °C, degree days and litres per degree day.

### Readings and Refills tables

Readings show date, gauge cm, real litres, days to the next reading and L/day. Estimated refill-gap rates are in italics and refill rows are shaded. Refills show date, litres, €/L, total (**set** marks a manually entered total), measured litres and the difference from the ordered amount.

---

## Settings

| Setting | Default | Meaning |
|---|---|---|
| Litres per cm | 9.33 | Tank litres per cm of gauge |
| Gauge reads high by (cm) | 6 | The outside gauge shows this many cm more than the real level |
| Full tank on gauge (cm) | 140 | Gauge reading when full (about 1,250 L real with the defaults) |
| Reorder at gauge (cm) | 30 | Level used for the reorder forecast |
| Location | none | Town used to download temperatures (search by name). Only the coordinates are sent. |
| Degree-day base (°C) | 18 | Outdoor daily mean temperature. See [Degree days](#degree-days). |
| Heating threshold (°C) | 15 | Outdoor daily mean at or below which a day counts degree days |
| Forecast method | Automatic | **Automatic** uses the temperature model when it fits well, otherwise monthly averages. **Monthly averages only** never uses temperatures for forecasts. |

The dialog also has the **Fit to my readings** button (see [Fitting](#fitting-the-base-and-threshold-to-your-house)), the **Data format** downloads, and **Restore spreadsheet data**. Restoring replaces all current data, so export first.

---

## How the calculations work

### Real litres

```
real litres = max(0, (gauge cm − gauge offset) × litres per cm)
            = max(0, (cm − 6) × 9.33)   with the defaults
```

A reading below the offset (for example 0 cm) counts as 0 L and is flagged.

### Gaps between readings

Consecutive readings form gaps. If the gauge rises by **more than 2 cm**, the gap is treated as a refill gap; smaller rises are treated as gauge noise, meaning no consumption.

- **Normal gap:** use = start litres − end litres. L/day = use ÷ days. This is the same as the spreadsheet's Dcm/Dt × 9.33.
- **Refill gap:** the refill hides consumption, so use is estimated. For gaps of up to 10 days it uses the average L/day of the nearest normal gaps before and after; for longer gaps it uses the monthly profile.

Use is spread evenly over the days of each gap, giving daily use for every day between the first and last reading. All season, monthly, heating-period and cumulative figures are sums of this daily use, so the totals always balance.

### Matching refills to readings

Each refill is assigned to the refill gap whose dates contain the refill date (start and end inclusive). This works whether the refill is logged on the day of the reading before or after the fill. A refill dated after the last reading is pending: it's added to the current-level estimate.

### Delivery check

For refill gaps of up to 10 days:

```
measured litres = end litres − start litres + estimated use during the gap
difference      = measured − ordered
```

A difference larger than **30 L or 5% of the order**, whichever is bigger, is flagged. Longer gaps aren't checked, because the estimated use would dominate the result.

### Monthly profile

The average L/day for each calendar month, from normal gaps only. It's the fallback forecast and is shown under the **By month** tab ("Average, all seasons").

### Current level and reorder forecast

1. Start from the last reading's real litres.
2. For each day since then, subtract the expected daily use and add any pending refills (capped at the full level).
3. Continue day by day from today, for up to 730 days, until the level reaches the reorder level (**reorder date**) and 0 (**empty date**).

Expected daily use comes from the temperature model when it's in use, otherwise from the monthly profile. The note under the tank chart says which method is used and why. The suggested order is full minus reorder level, rounded down to 10 L, priced at the trend price on the reorder date.

### Seasons and costs

- A **season** runs from 1 July to 30 June (for example 2025-26).
- **Recorded litres:** daily use within the season, between the first and last reading.
- **Projected litres:** expected use from the last reading to the end of the season.
- **Spent:** totals of the refills dated in that season.
- **Average paid €/L:** litre-weighted average price of that season's refills.
- **Consumption cost:** recorded litres × average paid price (or the trend price if the season has no refills), plus projected litres × trend price at the middle of the projected period.

### Price trend

An ordinary least-squares straight line through all prices paid against date, projected 12 months ahead. The shaded band is ±1 standard deviation of the residuals. The note shows the yearly change and how much of the price movement the trend explains. With heating-oil prices this is usually little, so treat it as a rough guide.

---

## Temperatures and degree days

### Source

Temperatures come from **Open-Meteo** (<https://open-meteo.com>), which is free for non-commercial use and needs no API key:

- **Historical archive:** daily mean temperature from about 10 years before today (or 31 days before your first reading, whichever is earlier) up to yesterday.
- **Forecast:** the last 92 days, to fill the archive's delay, plus the next 16 days.
- **Typical temperature** for each day of the year: the average of the observed years over a ±7-day window. It's used beyond the 16-day forecast.

Temperatures are cached in the browser and refreshed at most every 6 hours; after the first download only new days are fetched. **Update temperatures** under the tabs forces a refresh. Without a connection, the saved temperatures are used and a message explains it.

### Degree days

A degree day measures how much heating the weather demands, combining how cold it was and for how long. With base *B* and threshold *T*:

```
degree days for a day = B − daily mean   if daily mean ≤ T
                      = 0                otherwise
```

These are **outdoor** temperatures, not thermostat settings. More degree days means colder weather and more heating needed. For example, 10 days at 8 °C and 5 days at −2 °C both give 100 degree days with base 18.

**Litres per degree day** = litres used ÷ degree days over the same recorded days (1 October to 30 April). Lower means less oil for the same cold. It includes any baseline use that doesn't depend on temperature.

### Temperature model

Fitted over all normal gaps that have temperatures:

```
L/day = baseline + litres per degree day × (degree days per day)
```

It's fitted by weighted least squares, with each gap weighted by its number of days; the baseline is kept at 0 or above. The model is used for forecasts only when **Forecast method** is Automatic, at least 5 gaps are available, litres per degree day is positive, and the model explains **at least 50%** of the variation in your usage (R²).

### Fitting the base and threshold to your house

The defaults (18 °C base, 15 °C threshold, Eurostat method) suit a thermostat around 20 °C. A warmer setting and a large house with a lot of marble usually mean heating starts on milder days, so both values are likely higher.

**Settings → Fit to my readings** tries every base from 15 to 24 °C and every threshold from 12 °C up to the base, in 0.5 °C steps (304 combinations). It reports the pair that best explains your consumption, compared with the current values. **Use these values** fills them in; **Save settings** applies them.

- **"Several combinations fit almost equally well"** appears when other pairs come within 10% of the best pair's unexplained variation, and gives the range. Treat the exact values with caution then; more frequent readings, especially in October–November and March–April, narrow it down.
- **"The improvement is small"** appears when the best pair explains less than 2 percentage points more than the current values.

---

## Data checks

The **Data checks** panel lists everything worth a look. Warnings come first.

| Check | Meaning |
|---|---|
| Last reading is *N* days old | More than 14 days since the last reading; the current level is an estimate |
| Reading below the gauge offset | Real level counted as 0 L |
| Gauge rose but no refill is recorded | A rise of more than 2 cm with no refill in that gap |
| Refill has no matching rise | A refill not within any rising gap |
| Ordered vs measured | Delivery check outside ±30 L or 5% |
| Refill has no litres | It counts for the price trend only |
| Total differs from litres × price | A manually entered total that differs by more than €0.50 |
| Two readings on the same day | Used in the order entered |
| Dated in the future | Possible only via import; check the date |
| Notes | Every note on a reading or refill, including the corrections made to the spreadsheet data |

---

## Import, export and the data format

- **Export data** downloads `heating-oil-YYYY-MM-DD.json` with all readings, refills and settings.
- **Import data** replaces the current data with a file, after confirmation. The whole file is validated first; if anything is wrong, nothing is imported and every problem is listed (up to 12), with the record number and date.
- **Settings → Data format** has two downloads:
  - **Download schema**: `heating-oil-schema-v1.json`, a JSON Schema (draft-07) with descriptions, limits and defaults for every field. Editors such as VS Code can use it to check files and suggest fields.
  - **Download example file**: `heating-oil-example.json`, a small valid file (7 readings, 2 refills) that shows every field. It can be imported directly, which replaces your data, so export first.

### Structure

```json
{
  "version": 1,
  "app": "heating-oil-tracker",
  "exportedAt": "2026-09-30T10:00:00.000Z",
  "settings": {
    "cmToL": 9.33, "gaugeOffsetCm": 6, "fullGaugeCm": 140, "reorderGaugeCm": 30,
    "hddBase": 18, "hddThreshold": 15, "forecastMethod": "auto",
    "location": { "name": "Town, Region, Country", "latitude": 40.64, "longitude": 22.93 }
  },
  "readings": [
    { "id": "r1", "date": "2026-01-13", "cm": 26, "note": "" },
    { "id": "r2", "date": "2026-01-14", "cm": 112, "note": "" }
  ],
  "refills": [
    { "id": "f1", "date": "2026-01-13", "litres": 800, "pricePerL": 1.09, "totalEur": 879, "note": "Invoice total" }
  ]
}
```

### Rules checked on import

| Field | Rule |
|---|---|
| `version` | Optional; if present it must be `1` |
| `readings`, `refills` | Required lists |
| `settings` | Optional; each missing field takes its default |
| `cmToL` | Above 0 |
| `gaugeOffsetCm` | 0 or more |
| `fullGaugeCm` | Above `gaugeOffsetCm` |
| `reorderGaugeCm` | At least `gaugeOffsetCm` and below `fullGaugeCm` |
| `hddBase` | 10 to 25 |
| `hddThreshold` | 5 to `hddBase` |
| `forecastMethod` | `"auto"` or `"profile"` |
| `location` | `null` or `{name, latitude (−90 to 90), longitude (−180 to 180)}` |
| `id` | Optional non-empty text, unique across readings and refills; generated if missing |
| `date` | `YYYY-MM-DD`, a real calendar date between 1990 and 2100 |
| `cm` | Required number, 0 to 500 |
| `litres`, `pricePerL` | Number above 0, or `null`; each refill needs at least one of them |
| `totalEur` | Number 0 or more, or `null` (meaning litres × price) |
| `note` | Optional text, up to 500 characters |

Text instead of numbers (for example `"37.5"`), empty strings and numeric IDs are rejected. Unknown fields are ignored. Four rules can't be expressed in JSON Schema and are checked only by the app: real calendar dates, unique IDs, the reorder level between the offset and full, and the threshold not above the base. They're listed in the schema's `$comment`.

Temperatures, the backup reminder and all calculated figures are not exported; they're recalculated or downloaded again.

---

## Initial data from the spreadsheet

The app ships with 65 readings and 16 refills from `Petrelaio.xlsx`, seasons 2021-22 to 2025-26. These corrections and interpretations were applied. Each one is recorded as a note on its record, listed under Data checks, and can be edited:

| Record | Spreadsheet | In the app | Reason |
|---|---|---|---|
| Reading, sheet 2023-24, first row | 22/10/2022, 36 cm | 22/10/**2023** | Assumed typo (it gave a 376-day gap) |
| Refill 900 L at €1.30 | Fill date 04/11/**2024** | 04/11/**2023** | Assumed typo; the reading row is 2023 |
| Refills in 2024-25 and 2025-26 | No fill date | Date of the row the refill is on | No fill date given |
| 25/10/2021 at €1.05/L | Price without litres | Refill with unknown litres | Counts for the price trend only |
| Reading 01/03/2025 | 0 cm | Kept, counted as 0 L | Possibly the tank ran dry; please confirm |
| Refill 13/01/2026 | Total €879 typed in | Manual total kept | 800 × 1.09 would be €872 |

Season spend totals match the spreadsheet: 2021-22 €2,891.00, 2022-23 €3,390.00, 2023-24 €3,214.00, 2024-25 €3,306.43, 2025-26 €1,969.00. Settings → **Restore spreadsheet data** reloads this data set.

---

## Limitations

- **Accuracy depends on reading frequency.** Rates between readings are averages; long gaps, especially in summer, show as one flat rate.
- **Gauge precision:** readings to 0.5 cm (about 4.7 L) make rates over gaps of a few days jumpy. Use the smoothed view for trends.
- **Forecasts assume normal behaviour:** typical temperatures beyond 16 days and unchanged heating habits. The price trend is statistical only.
- **Thermal mass:** a marble house reacts to cold spells a day or two late. With readings several days apart this has little effect, but the model doesn't account for it.
- **Storage:** data lives in one browser on one device until exported (see [backup](#where-your-data-is-stored-and-how-to-back-it-up)).
- **Testing:** automated tests ran in Chromium with the chart library and weather service replaced by stand-ins. Chart appearance, live Open-Meteo responses, and Safari or Firefox should be checked by hand.

---

## Troubleshooting

| Problem | What to do |
|---|---|
| Red banner: "This browser view can't save data" | You're in a preview or a restricted view. Download the file and open it directly in a browser. |
| Red banner: "Charts need an internet connection" | Connect and reload. Tables and numbers still work. |
| "Reorder now" and 0 L although the tank is full | No reading since the last refill. Add a current reading (and any missing refills). |
| Temperatures couldn't be updated | Check the connection, then click **Update temperatures**. Saved temperatures are still used. |
| No places found | Try the name in English or Greek, or a nearby larger town. |
| Import stopped | Fix the listed problems in the file and import again. Nothing was changed. |
| Data disappeared | Import your latest export from Google Drive. If the app showed the "unreadable data" banner, download that data first. |
| A gauge rise is flagged with no refill | Add the missing refill with the right date, or correct the reading. |
| Large ordered vs measured difference | Check the reading dates around the refill and the delivery note. |

---

## Technical notes

- **One file:** HTML, CSS and JavaScript, with no build step and no server.
- **External resources:**
  - Chart.js 4.4.1 from cdnjs: `https://cdnjs.cloudflare.com/ajax/libs/Chart.js/4.4.1/chart.umd.min.js`
  - Barlow fonts from Google Fonts, with a system font fallback
  - Open-Meteo APIs: `geocoding-api.open-meteo.com`, `archive-api.open-meteo.com`, `api.open-meteo.com`
- **No `<form>` elements and no `alert` or `confirm`:** the app works inside sandboxed previews.
- **Calculations** run in the browser in one function (`compute`) between the `CALC-START` and `CALC-END` markers, so they can be tested on their own with Node.js.
- **Dates** are calendar dates (`YYYY-MM-DD`) handled as UTC day numbers, so daylight saving time doesn't affect day counts.
- **Version:** data format version 1.
