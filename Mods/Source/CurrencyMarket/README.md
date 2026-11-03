# Currency Market

Local synthetic liquidity for Faustus, priced from poe.ninja's public PoE 1 exchange feed.
The mod creates finite local stock. It requires this checkout's Currency Exchange API and NpcDialogue module.

Build from the server root: `dotnet build Mods/Source/CurrencyMarket -c Debug`.
Enable/disable: `/mod_enable currencymarket` / `/mod_disable currencymarket`.
Status: `/cx_market`. Re-enter the hideout after enabling if Faustus was absent.

Edit `data/mods/currencymarket/settings.json`, then run `/mod_reload currencymarket`.
Builds never overwrite settings. The mod generates this file on first load:

```json
{
  "FetchPrices": true,
  "PriceLeague": "Standard",
  "LocalLeagues": ["Standard"],
  "RefreshMinutes": 60,
  "GapPercent": 10,
  "LevelGapPercent": 2,
  "VolumeFactor": 0.05,
  "LevelVolumeMultiplier": 1.25
}
```

| Setting | Meaning |
| --- | --- |
| FetchPrices | Fetch prices online; false uses only the saved cache |
| PriceLeague | poe.ninja source league |
| LocalLeagues | Local leagues receiving stock; [] serves all non-SSF leagues |
| RefreshMinutes | Price fetch interval and UTC-aligned restock interval; minimum 15 minutes |
| GapPercent | Total gap around reference: 10 means buy at -5% and sell at +5% |
| LevelGapPercent | Additional percentage points away from reference per deeper level |
| VolumeFactor | First level's units = observed traded units × this factor, on both sides; 0 disables stock |
| LevelVolumeMultiplier | Each deeper level's target volume multiplier; greater than 0, at most 2 |

Six levels per side are fixed. With a 100-Chaos reference, default bids are
95, 93, 91, 89, 87, 85; asks are 105, 107, 109, 111, 113, 115.
An observation of 1,000 traded units produces volumes 50, 62, 78, 97, 122, 152.
Observed units are `volumePrimaryValue / primaryValue`.
Rates round outward into distinct integer lots; this can widen gaps further.
Each level has at least one whole lot and at most 10,000 currency units, subject to protocol limits.
Unrepresentable levels and unsupported client currencies are skipped.

Faustus placement, disk caching and Currency/Fragment categories are automatic.
The cache lives at `data/mods/currencymarket/prices.json`.
Failed fetches keep the last valid cache and its original timestamp; no cache means no liquidity.
Network work runs in the background. No accounts, tokens or player data are sent.

Published stock enters the book on the existing player tick cycle. Unchanged ticks skip database work.
Trades reduce stock persistently and push depth updates; exhausted levels disappear.
Requests never replenish stock. Reloading the same snapshot within an interval preserves fills.
Changed prices or settings can replace stock before the next interval.
Disabling withdraws mod liquidity while preserving player orders and earned proceeds.
Gold fees and normal collection apply. SSF stays disabled.

Source: https://poe.ninja/docs/api
