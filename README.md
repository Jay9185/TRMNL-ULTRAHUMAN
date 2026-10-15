# Ultrahuman for TRMNL

A custom serverless plugin that fetches live health metrics from the **Ultrahuman Ring Air** and formats them for the **TRMNL** e-ink display.

Built on **Cloudflare Workers**, this plugin handles authentication, data processing, history tracking, and intelligent caching to provide a reliable, "set it and forget it" dashboard.

> 📖 **Read the write-up:** I built this while wearing the Ring Air through 168 days of flight training. The full review, with the sleep and HRV data from 47 flight days, is on TurbulenceGains: **[Ultrahuman Ring Air Review for Pilots](https://turbulencegains.com/imsafe-ultrahuman-ring-pilot/)**. More TRMNL builds: [TRMNL review](https://turbulencegains.com/trmnl-review-2026/) · [TRMNL flight radar](https://turbulencegains.com/trmnl-flight-radar-guide/).

![IMG20260117120301](https://github.com/user-attachments/assets/8370542d-7341-454e-ac0e-e865c313a1d8)

![IMG20260117115857](https://github.com/user-attachments/assets/25c54f21-923c-4410-b760-f8d643af370c)

## ⚡ Key Features

* **Live Metrics:** Displays Sleep Score, Duration, Recovery Index, HRV, RHR, Temperature, SpO2, Steps, and Movement Index.
* **Smart Trends:** Compares today's HRV with yesterday's to display trend arrows (▲/▼).
* **7-Day History Chart:** Automatically tracks step counts to generate a weekly bar chart at the bottom of the screen.
* **Zero-Data Fallback:** If you wake up past midnight and the ring hasn't synced yet, the plugin automatically displays yesterday's data instead of showing empty zeros.
* **Auto-Audit Engine:** Every 3 days, it performs a background "audit" to fetch the last 7 days of data, correcting any past discrepancies (e.g., edited sleep times).
* **Secure:** API tokens are stored in Cloudflare Encrypted Secrets, not in the code.
* **Efficient:** Uses `KV_STORE` for caching and history, minimizing API calls to Ultrahuman.

## 🛠️ Prerequisites

1. **Ultrahuman Ring** (Air or R1).
2. **Ultrahuman API Key**: Go to **[https://vision.ultrahuman.com/developer](https://vision.ultrahuman.com/developer)** to generate your personal API Key.
3. **TRMNL Device**: [Get one here](https://usetrmnl.com/).
4. **Cloudflare Account**: A free account works perfectly.

## 🚀 Setup Guide

### 1. Create the Cloudflare Worker

1. Log in to the [Cloudflare Dashboard](https://dash.cloudflare.com/).
2. Go to **Workers & Pages** -> **Create Application** -> **Create Worker**.
3. Name it `ultrahuman-trmnl` (or similar).
4. Click **Deploy**.

### 2. Configure the KV Namespace (Database)

This plugin needs a small database to store your history and trends.

1. In your Worker's dashboard, go to **Settings** -> **Variables**.
2. Scroll down to **KV Namespace Bindings**.
3. Click **Add Binding**.
* **Variable name:** `KV_STORE` (Must be exact).
* **KV Namespace:** Click "Create new" and name it `trmnl_db`.


4. Click **Save**.

### 3. Add Your API Token (Secret)

Never hardcode your token. Use Secrets.

1. Still in **Settings** -> **Variables**, scroll to **Environment Variables**.
2. Click **Add Variable**.
* **Variable name:** `API_TOKEN`
* **Value:** Paste the API Key you got from the Ultrahuman Developer portal.
* **Click "Encrypt"** to make it a Secret.


3. Click **Save**.

### 4. Deploy the Code

1. Click **Edit Code** to open the online editor.
2. Copy the content of `worker.js` from this repository.
3. Paste it into the editor, replacing the existing code.
4. **Important:** Check line 8: `const USER_TZ_OFFSET = 5.5;`
* Default is `5.5` (IST - India Standard Time).
* Change this to your offset if different (e.g., `-5.0` for EST, `0` for GMT).


5. Click **Deploy**.

### 5. Connect to TRMNL

1. Copy your Worker's URL (e.g., `https://ultrahuman-trmnl.yourname.workers.dev`).
2. Go to your [TRMNL Dashboard](https://usetrmnl.com/dashboard).
3. Navigate to **Plugins** -> **Private Plugin**.
4. Paste your Worker URL into the **URL** field.
5. Set the **Refresh Rate** (Recommended: 15 or 30 minutes).
6. Save and check your device!

## 🧩 How It Works

### The "Midnight Gap" Problem

Health APIs often return empty data right after midnight (00:00 - 06:00) before you wake up and sync your ring.

* **Standard Plugins:** Show "0 Steps", "0 Sleep".
* **This Plugin:** Detects the empty data and automatically fetches and displays **Yesterday's** full stats until Today's data becomes available.

### The Audit System

The plugin runs a lightweight "Audit" in the background:

* **Trigger:** Runs if the database is empty or hasn't been updated in 3 days.
* **Action:** Fetches the last 7 days of raw data in parallel.
* **Result:** Updates the Weekly Step Chart and recalculates trend baselines.

## ⚠️ Troubleshooting

**My screen shows "!!" or "Check Logs"**

* This means the worker encountered an error.
* Go to Cloudflare Dashboard -> Your Worker -> Logs -> Real-time Logs.
* Refresh the TRMNL plugin to trigger a request and see the specific error message.

**I see "API Error 401"**

* Your `API_TOKEN` is invalid or missing. Go to Settings -> Variables and re-enter it as a Secret.

**I see 0% SpO2**

* This happens if the ring hasn't recorded blood oxygen data for the night yet. The plugin tries to calculate an average from raw data if the main reading is missing, but sometimes (rarely) no data exists yet.

## 📄 License

MIT License. Feel free to modify and share!
