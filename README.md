# 🦅 Valkyrie "Mercenary" Trading System

**Version:** 2.0 (Mercenary Edition)

> Automated n8n workflows for multi-timeframe, logic-driven trading signals.

## Overview

Valkyrie is a pair of n8n workflows that act as a signal scanner and an outcome manager. The system implements a conservative multi-timeframe strategy (4H trend + 15M confirmation) and includes a simple state machine for risk control (ACTIVE / STAND_DOWN).

**Key features**

- Scanner workflow (`ibdl_scanner_mercenary`) — scans Kraken USD pairs with a forced watchlist and multi-timeframe checks.
- Outcome manager workflow (`ibdl_outcome_manager`) — evaluates signals and updates system state based on outcomes.
- File-based logging for reproducible behaviour and easy audit.
- Alerts via Discord (optional) via `DISCORD_WEBHOOK_URL` environment variable.

## Quickstart ⚡

1. Ensure you have a running n8n instance (Docker or npm install).
2. Clone this repository to your n8n host.
3. Create the log directory and initialize state/event files:

```bash
sudo mkdir -p /opt/valkyrie/logs
sudo chmod -R 700 /opt/valkyrie/logs

echo '{"state":"ACTIVE","last_change_ts":0}' > /opt/valkyrie/logs/ibdl_state.json

echo 'event_ts,event_type,signal_id,pair,entry_price,signal,mae_pct,mfe_pct,bars_to_ma50,hit_ma50,structure_hold,fail,state' > /opt/valkyrie/logs/mr_events.csv
```

4. (Optional) Set environment variables for alerts:

- `DISCORD_WEBHOOK_URL` — Discord webhook for signal notifications.
- `TELEGRAM_CHAT_ID` — Telegram alerts (if implemented).

5. Import the workflows from `workflows/` into n8n (Use the n8n UI → Import → JSON).
6. Activate both workflows and monitor logs at `/opt/valkyrie/logs/mr_events.csv`.

## Files

- `workflows/ibdl_scanner_mercenary.json` — Scanner (4H + 15M confirmation)
- `workflows/ibdl_outcome_manager.json` — Outcome manager and state machine
- `2026-01-11_Mercinary_Edition.md` — Notes & original embedded JSON

## Testing & Validation ✅

I validated the workflow JSON files in `2026-01-11_Mercinary_Edition.md` and both JSON blocks parse successfully (no syntax errors).

> Note: This validation checks only JSON syntax and structural validity for import; it does not run the workflows against live market data or simulate trade outcomes.

## Safety & Disclaimer ⚠️

- This project is provided for educational purposes. Trading involves significant risk; past performance is not indicative of future results. Use at your own risk.

## Contributing

Contributions are welcome. For small fixes, open a pull request with a short explanation. For larger changes, open an issue first to discuss scope.

## License

This repository will use the MIT License by default. If you prefer another license, tell me and I will switch it.

---

If you'd like, I can replace the empty `README.md` with this generated README; say "Replace" and I'll perform the replacement.