# Elder Impulse System Alert — MQL4 Script

A MetaTrader 4 script that implements **Dr. Alexander Elder's Impulse System** by simultaneously monitoring the slope direction of a configurable EMA and the slope direction of a manually computed MACD histogram — derived as `MODE_MAIN − MODE_SIGNAL` — and firing buy or sell impulse alerts only when both indicators are aligned in the same direction, using `PrevEMA` and `PrevMACDHist` persistent global variables to enforce strict bar-to-bar slope comparison.

---

## Overview

The Elder Impulse System is a two-filter confluence signal framework developed by Dr. Alexander Elder. It combines a trend-following indicator (EMA slope) with a momentum oscillator (MACD histogram slope) to identify bars where both momentum and trend agree on direction — filtering out low-conviction signals that occur when the two are diverging or flat. A **buy impulse** fires only when the EMA is rising AND the MACD histogram is rising simultaneously, indicating expanding bullish momentum within an uptrend. A **sell impulse** fires only when both are falling simultaneously. This dual-condition gate significantly reduces false signals compared to single-indicator systems. The script tracks `PrevEMA` and `PrevMACDHist` across loop iterations to compute slopes on every cycle without requiring array allocation or bar-shift comparison.

---

## Features

- **Dual-condition impulse gate** — buy impulse: `currentEMA > PrevEMA && currentMACDHist > PrevMACDHist`; sell impulse: `currentEMA < PrevEMA && currentMACDHist < PrevMACDHist` — both conditions must be simultaneously true
- **Manual MACD histogram construction** — `currentMACDHist = iMACD(..., MODE_MAIN, 0) − iMACD(..., MODE_SIGNAL, 0)` computed explicitly each cycle, matching Elder's original histogram definition
- **Persistent slope state** — `PrevEMA` and `PrevMACDHist` global doubles retain prior-cycle values across loop iterations; initialized to `0.0` and skipped on the first cycle to prevent false triggers during warm-up
- **Fully configurable indicator parameters** — `EMAPeriod`, `MACDFast`, `MACDSlow`, and `MACDSignal` all independently configurable to replicate Elder's original `13/12/26/9` settings or any custom configuration
- **Three notification channels:** sound alert, email, and mobile push
- **Lightweight loop** — polls once per minute (`Sleep(60000)`)
- Logs impulse type, symbol, and timeframe to the MT4 **Experts** tab on every trigger

---

## How It Works

1. Every minute, `iMA(..., EMAPeriod, 0, MODE_EMA, PRICE_CLOSE, 0)` and both MACD `MODE_MAIN` and `MODE_SIGNAL` values are fetched; histogram computed as their difference
2. Two impulse conditions are evaluated using stored prior-cycle values:
   - `currentEMA > PrevEMA && currentMACDHist > PrevMACDHist` → **Buy Impulse Detected**
   - `currentEMA < PrevEMA && currentMACDHist < PrevMACDHist` → **Sell Impulse Detected**
3. `PrevEMA` and `PrevMACDHist` are updated at the end of every cycle for next iteration comparison
4. `AlertElderImpulse()` dispatches the message via all enabled channels with symbol and timeframe

---

## Input Parameters

| Parameter      | Type            | Default     | Description                                           |
|----------------|-----------------|-------------|-------------------------------------------------------|
| `TradeSymbol`  | string          | `EURUSD`    | Symbol for analysis                                   |
| `Timeframe`    | ENUM_TIMEFRAMES | `PERIOD_H1` | Timeframe for analysis                                |
| `EMAPeriod`    | int             | `13`        | EMA period for trend slope component                  |
| `MACDFast`     | int             | `12`        | MACD fast EMA period                                  |
| `MACDSlow`     | int             | `26`        | MACD slow EMA period                                  |
| `MACDSignal`   | int             | `9`         | MACD signal line period for histogram computation     |
| `EnableAlerts` | bool            | `true`      | Fire an on-screen/sound alert                         |
| `EnableEmail`  | bool            | `false`     | Send an email notification                            |
| `EnablePush`   | bool            | `false`     | Send a mobile push notification                       |

---

## Alert Message Format

```
Buy Impulse Detected detected on EURUSD (Timeframe: PERIOD_H1)
```

---

## Installation

1. Copy `Elder_Impulse_System_001.mq4` to `MQL4/Scripts/` in your MT4 data folder
2. Compile in MetaEditor (F7)
3. Drag onto any chart from Navigator → Scripts
4. Configure inputs and click **OK**

---

## Requirements

- MetaTrader 4 (`#property strict` compatible build)
- MQL4 compiler (MetaEditor)

---

## License

MIT License

Copyright (c) 2026

Permission is hereby granted, free of charge, to any person obtaining a copy of this software and associated documentation files (the "Software"), to deal in the Software without restriction, including without limitation the rights to use, copy, modify, merge, publish, distribute, sublicense, and/or sell copies of the Software, and to permit persons to whom the Software is furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.
