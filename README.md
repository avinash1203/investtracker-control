# InvestTracker remote control

`control.json` is a kill switch for the InvestTracker desktop app.
The app reads it (public, read-only); only this repository's owner can change it.

```json
{
  "app": "InvestTracker",
  "enabled": true,
  "message": "",
  "checkEveryMinutes": 10
}
```

| Field | Meaning |
|---|---|
| `app` | Must be exactly `InvestTracker`. Any other value → the file is ignored (treated as "no answer"). |
| `enabled` | `true` = the app runs. `false` = the app closes itself (data is **never** deleted). |
| `message` | Optional text shown to the user when the app closes (e.g. "Maintenance — call me"). |
| `checkEveryMinutes` | How often the running app re-reads this file (1–1440, default 10). |

## To stop the app

Edit `control.json` → set `"enabled": false` → commit to `main`.
Every running copy closes within `checkEveryMinutes` (GitHub's CDN may add up to ~5 min).
Copies started later close immediately.

## To allow it again

Set `"enabled": true` and commit. Copies that were closed can be started again;
they check this file first and run.

## Rules the app follows

1. **A confirmed change is required to stop.** The app stops only after it has
   actually read this file with `"enabled": false` (HTTP 200, valid JSON, `app` matches).
2. **No answer ≠ disabled.** Network down, GitHub down, 404, malformed JSON — the app
   treats all of these as "no answer" and keeps using the **last value it stored locally**.
3. **The last read value is cached** in the app's local database. Offline behaviour follows
   that cache: last known `enabled: true` → runs offline; last known `false` → does not run
   offline (it will try GitHub first, and only a fresh `true` lets it run again).
4. **Nothing is ever deleted.** Disabling only closes the window; all accounts, ledgers,
   allotments and backups stay on disk.
