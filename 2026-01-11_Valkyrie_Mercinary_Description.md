Yes, the code provided in the previous response is the **finished, corrected "Mercenary" Edition**. It strictly follows your requirement for "pure logic" by removing the volume filters for your specific assets and adding the multi-timeframe confirmation to prevent bad entries.

Here is the description of the system and the `README` documentation you requested.

### 🛡️ System Description: "The Valkyrie Mercenary"

This system is a **closed-loop, automated trading engine** running on **n8n**. It is designed to act as a cold, emotionless sniper. It does not care about your current portfolio "bags"; it only cares about the **next high-probability trade**.

**What It Does:**

1. **Forced Scans:** Every hour, it forces a scan of your target list (**XMR, SOL, LTC, XRP, BTC, ETH**) plus the top 40 highest-volume coins on Kraken. It ignores low-volume trash unless it is on your whitelist.
2. **The "Trend" Gate (4H):** It first checks the 4-Hour chart. Is the asset in a "God Mode" uptrend (Price > 200 SMA & 20 > 50 SMA)? Did it dip? Did it reclaim the trend? If no, it discards the asset immediately.
3. **The "Sniper" Gate (15M):** If the 4H trend is perfect, it zooms in to the 15-Minute chart. It checks if the RSI is "safe" (between 45 and 62). If the RSI is too high (>62), it kills the trade to prevent FOMO. If too low (<45), it waits.
4. **Self-Policing:** It watches its own performance. If it starts losing (12 fails in the last 20 trades), it activates a **"Circuit Breaker"** and goes silent (`STAND_DOWN` state) until the market improves.

---

### 📄 README.md

You can save the following text as `README.md` for your documentation.

```markdown
# 🦅 Valkyrie "Mercenary" Trading System

**Version:** 2.0 (Mercenary Edition)
**Platform:** n8n (Self-Hosted)
**Strategy:** 4H Trend Mean Reversion + 15M Momentum Confirmation

## 📋 System Overview

Valkyrie is an automated, logic-driven trading signal generator. It operates on a "State Machine" principle to ensure risk management is handled automatically.

* **Scanner (Workflow A):** Identifies high-probability entries using multi-timeframe analysis.
* **Manager (Workflow B):** Tracks the outcome of every signal (Win/Loss) and manages the system state (`ACTIVE` or `STAND_DOWN`).

## ⚙️ Prerequisites

1.  **n8n Instance:** Installed via Docker or npm.
2.  **Filesystem Access:** The n8n instance **must** have read/write access to a local directory for logging.
    * **Path:** `/data/valkyrie/logs` (Container side)
3.  **Environment Variables:**
    * `DISCORD_WEBHOOK_URL`: (Optional) URL for Discord alerts.
    * `TELEGRAM_CHAT_ID`: (Optional) Chat ID for Telegram alerts.

## 🚀 Installation

### 1. Prepare the Filesystem
Run these commands on your host machine to create the necessary log files:

```bash
sudo mkdir -p /opt/valkyrie/logs
sudo chmod -R 777 /opt/valkyrie/logs

# Initialize State File
echo '{"state":"ACTIVE","last_change_ts":0}' > /opt/valkyrie/logs/ibdl_state.json

# Initialize Event Log
echo 'event_ts,event_type,signal_id,pair,entry_price,signal,mae_pct,mfe_pct,bars_to_ma50,hit_ma50,structure_hold,fail,state' > /opt/valkyrie/logs/mr_events.csv

```

*Note: Ensure your Docker container maps `/opt/valkyrie/logs` to `/data/valkyrie/logs`.*

### 2. Import Workflows

Import the two provided JSON files into your n8n dashboard:

1. `ibdl_scanner_mercenary.json` (The Scanner)
2. `ibdl_outcome_manager.json` (The Manager)

### 3. Configuration

* Open the **Scanner** workflow.
* Check the `Build USD Universe` node. The `WHITELIST` array contains the assets that are **always scanned**:
```javascript
const WHITELIST = ['XMR/USD', 'SOL/USD', 'LTC/USD', 'XRP/USD', 'ETH/USD', 'BTC/USD'];

```


*Modify this list if you want to add/remove specific assets.*

## 🧠 Logic & Strategy

### The Entry Signal (AND Logic)

A signal is generated **only** if ALL of the following are true:

1. **4H Trend:** Price > SMA 200 **AND** SMA 20 > SMA 50.
2. **4H Structure:** Price has "dipped" (touched SMA 20 or 50) within the last 12 bars (48 hours).
3. **4H Trigger:** Price has "reclaimed" (closed above) the SMA 20.
4. **15M Confirmation:** The 15-Minute RSI is between **45 and 62**.
* *RSI > 62:* Too hot (Risk of buying the top).
* *RSI < 45:* Momentum too weak.



### The Circuit Breaker (Risk Management)

The system calculates a "Fail Rate" based on the last 20 completed trades.

* **Fail Definition:** Price drops >3% below entry BEFORE hitting the SMA 50 target.
* **STAND DOWN:** If **12 or more** of the last 20 trades failed, the system enters `STAND_DOWN` mode.
* *Effect:* Scanning continues, logs are written, but **NO ALERTS are sent**.


* **RE-ACTIVATE:** If the system is in `STAND_DOWN` and records **3 consecutive Wins**, it automatically switches back to `ACTIVE`.

## 📊 Monitoring

To see what the bot is doing, you can check the logs directly:

**Check Current State:**

```bash
cat /opt/valkyrie/logs/ibdl_state.json
# Output: {"state":"ACTIVE", ...}

```

**Check Recent Signals:**

```bash
tail -n 20 /opt/valkyrie/logs/mr_events.csv

```

## ⚠️ Disclaimer

This software is for educational purposes only. It executes trades based on strict technical logic. Past performance of a strategy does not guarantee future results. Use at your own risk.

```

```