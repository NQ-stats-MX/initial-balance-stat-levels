# IB Extension Stats — Initial Balance Statistical Levels for TradingView
Highly customizable Initial Balance indicator with multiple pre sets and statistical levels to gauge how far price tends to extend.

A Pine Script v6 indicator that plots the **Initial Balance (IB)** for any intraday session and projects **percentile-based extension levels** built from that session's own history. Instead of guessing how far price "should" run after the IB breaks, you see how far it *actually* ran over the last few hundred sessions.

Built for index futures intraday trading (NQ/MNQ, ES/MES), but it works on any symbol with intraday data.

<img width="1835" height="1004" alt="image" src="https://github.com/user-attachments/assets/6b523720-761f-4cba-8511-f0480e461c23" />

---

## What it does

- **Draws the IB box.** It spans the full IB window and updates live as the range forms.
- **Marks internal IB levels**: the 50% midpoint by default, plus up to four more custom levels such as 25%, 75% or 12.5%.
- **Projects statistical extension levels** above the IB high and below the IB low once the IB closes. By default these are P25, P50 (median), P75 and P90; the percentiles are configurable.
- **Collects samples on a higher timeframe**, 30m by default, so the statistics cover hundreds of sessions even on a 1-minute chart.
- **Doesn't repaint.** Each session's levels are calculated only from *prior completed* sessions.
- **Shows a stats table** with the percentile values, sample counts, break rates for each side, both-sides break rate, and the last IB range.
- **Keeps historical IBs on the chart** (optional), so you can see visually how past sessions respected the levels.

---

## Session presets

All times are New York (America/New_York).

| Preset | IB Window | Extension Window (default) |
|---|---|---|
| **NY AM** (default) | 09:30 – 10:30 | 10:30 – 16:00 |
| NY Lunch | 12:00 – 13:00 | 13:00 – 16:00 |
| NY PM | 14:00 – 15:00 | 15:00 – 16:00 |
| Globex | 18:00 – 19:00 | 19:00 – 03:00 |
| Tokyo | 20:00 – 21:00 | 21:00 – 03:00 |
| Custom | any `HHMM-HHMM` | user defined |

The **extension window** is the period after the IB closes during which the indicator measures how far price extends beyond the IB. You can override the window's end time in settings.

---

## How the statistics work

For every completed session, the indicator records:

```
Upside extension   = (highest high in extension window − IB high) ÷ IB range
Downside extension = (IB low − lowest low in extension window)   ÷ IB range
```

Extensions are measured in **IB multiples**, so they adapt to volatility. A 0.5x level on a 100-point IB sits 50 points beyond the IB extreme; on a 300-point IB it sits 150 points beyond.

At the close of each new IB, the indicator projects the chosen percentiles of the historical distribution:

```
Upside level   = IB high + percentile × IB range
Downside level = IB low  − percentile × IB range
```

### Sample modes

| Mode | What it answers |
|---|---|
| **Break Days Only** (default) | *"Given that the IB high (or low) breaks, how far does price usually run?"* Upside stats come only from days that broke the IB high, and downside stats only from days that broke the IB low. |
| **Dominant Side** | *"How far does the day's main move usually extend?"* Each session contributes its larger extension (up or down), and the single distribution is mirrored on both sides, so the levels are symmetric. |

The **Break %** row in the table always uses *all* sessions. It tells you **how likely** a break is, while the levels tell you **how far** price runs once it breaks.

---

## Installation

1. Open any chart on [TradingView](https://www.tradingview.com).
2. Open the **Pine Editor** tab at the bottom of the screen.
3. Delete the template code and paste the full contents of [`IB_Extension_Stats.pine`](IB_Extension_Stats.pine).
4. Click **Save**, then **Add to chart**.

### Recommended chart setup

- **Timeframe:** 1m–15m for trading. Use a timeframe no longer than the IB itself (60m for the presets); the table warns you if you don't.
- **Session:** use **ETH (extended hours)** for the Globex and Tokyo presets. On an RTH-only chart those sessions don't exist.
- **Symbol:** continuous futures (e.g. `NQ1!`, `MNQ1!`, `ES1!`) give the cleanest history. CFDs work too.

---

## Settings reference

### Initial Balance
| Setting | Description |
|---|---|
| IB Preset | Choose a preset session or Custom |
| Custom IB Session | `HHMM-HHMM` NY time (used when preset = Custom) |
| Override Extension End | Replace the preset's extension end time (`HHMM`) |

### IB Box
| Setting | Description |
|---|---|
| Fill / Border / Style / Width | Box appearance |
| Extend IB High/Low & Internal Levels | Carry the IB high, low and internal levels through the extension window |
| Extended High/Low | Color, style and width for the extended IB high and low lines |

### IB Internal Levels
Five slots, each with **on/off, % from IB low, color, style, width**. Defaults: 50% on; 25%, 75%, 12.5% and 87.5% available but off.

### Extension Statistics
| Setting | Description |
|---|---|
| Show Statistical Levels | Master toggle for projected levels |
| Upside / Downside | Show either side independently |
| Stats Timeframe | Timeframe used to collect samples (default 30m). It must divide evenly into the IB start/end and extension end; the table warns if it doesn't |
| Sample | Break Days Only or Dominant Side (see above) |
| Lookback (sessions) | Number of recent sessions in the distribution (default 250) |
| Min Samples Before Plotting | Levels stay hidden until this many samples exist (default 10) |

### Statistical Levels (Percentiles)
Five slots, each with **on/off, percentile (1–99), color, style, width**. Defaults: P25, P50, P75 and P90 on; P95 available but off. Want P30 instead of P25? Just change the number.

### Labels
Toggle labels for the statistical and internal levels, include or hide prices, and set the label size.

### History & Table
| Setting | Description |
|---|---|
| Show Historical IBs & Levels | Keep past sessions on the chart or show only the current one |
| Max Historical Sessions | Up to 25 (TradingView's line limit) |
| Show Stats Table | Toggle the table, set its position, text size and colors |

---

## Reading the stats table

| Row | Meaning |
|---|---|
| P25 … P90 | Extension percentiles in IB multiples, upside and downside |
| Samples | Number of sessions in each side's distribution |
| Break % | % of all sessions where the IB high (Up) or low (Down) was taken |
| Both Sides % | % of sessions that broke **both** sides (neutral-day frequency) |
| Last IB Range | Current or most recent IB range in points |
| Stats TF | Timeframe the samples were collected on |

---

## Trading ideas

These are ways to *use* the levels, not a strategy. Test everything on your own instrument and timeframe.

- **Targets for breakouts:** after a clean break of the IB high, P50 is a "typical" run and P75–P90 are stretched moves.
- **Fade zones for mean reversion:** P75–P90 mark where extensions usually exhaust. A failed push beyond P90 back toward P75 or the IB edge is a classic fade setup.
- **Day-type context:** a low *Both Sides %* means one side usually holds, so a break of the first side can act as a bias filter for the rest of the session.
- **IB width context:** compare *Last IB Range* to recent IBs. Narrow IBs tend to extend further in IB multiples; wide IBs tend to stay balanced.
- **Midpoint rotation:** once price is back inside the IB after a failed extension, the 50% line is a common rotation target.

---

## Notes and limitations

- Statistics depend on the data feed. Futures, CFDs and different brokers can produce slightly different distributions.
- Holidays, half-days and data gaps are counted as regular sessions.
- Session times are fixed to New York time and respect daylight saving automatically.
- TradingView caps drawings at 500 lines, boxes and labels per indicator, which is why historical display is limited to 25 sessions.

---

## Disclaimer

This indicator is for educational and informational purposes only. It is **not financial advice**. Historical statistics do not guarantee future results. Trading futures and leveraged products involves substantial risk of loss. Use at your own risk.

---

## Contributing

Issues, suggestions and pull requests are welcome. If you find a bug, please include your symbol, timeframe, preset and a screenshot.
