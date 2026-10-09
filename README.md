# 🛒 IntelFlow Lite: Automated Shopify Price Monitor

**IntelFlow Lite** is a lightweight, fully automated workflow built in **n8n** that monitors Shopify product pages for price changes. It leverages Shopify's undocumented public JSON endpoints to scrape product data reliably without requiring expensive proxies, headless browsers, or paid scraping APIs.

When a price drop or increase is detected, the system updates a central database and instantly pushes a formatted alert to Discord/Slack.

## ✨ Key Features
- **Zero-Proxy Scraping:** Bypasses standard anti-bot measures by utilizing the native `.json` endpoint structure of Shopify stores.
- **Stateful Change Detection:** Compares live API data against historical database records to ensure alerts only fire on actual price changes (prevents spam).
- **Resilient Architecture:** Implements automatic retries and `Continue on Fail` error handling so a single broken URL never crashes the entire monitoring batch.
- **Automated Scheduling:** Runs hands-free on a configurable cron schedule (e.g., hourly).

## 🏗️ Workflow Architecture
```text
[Schedule Trigger] (Runs hourly)
       ↓
[Google Sheets: Read Products] (Fetches target URLs)
       ↓
[HTTP Request] (Fetches URL + .json) → Retries on failure
       ↓
[Code Node] (Parses JSON, compares new vs old price)
       ↓
   ┌───┴───┐
[IF: Price Changed?]
   ↓ (True)                ↓ (False)
[Update Sheet]         [Do Nothing]
   ↓
[Discord Alert]
       ↓
[Append to Logs] (Always runs for historical tracking)
```

## 🛠️ Tech Stack
Automation Engine: n8n (Self-hosted / Desktop)
Data Source: Shopify Public JSON API
Database: Google Sheets (via OAuth2)
Alerting: Discord Webhooks / Slack
Languages: JavaScript (Node.js inside n8n Code Nodes)
## 🚀 Setup & Installation
Import Workflow: Download workflow.json and import it into your n8n instance.
Credentials: Connect your own Google Sheets OAuth2 credential and Discord Webhook credential when prompted.
Database Setup: Create a Google Sheet with two tabs:
Products: Columns for Product Name, Product URL, Last Price, Last Checked.
Logs: Columns for Product Title, Price, Product URL, Checked At.
Activate: Turn on the workflow and let the Schedule Trigger run.
## 🧠 Lessons Learned & Debugging Journey
Building this project surfaced several real-world automation challenges:
The "Missing Data" Trap: Initially, the workflow flagged every item as "Out of Stock." I discovered that many Shopify themes hide inventory_quantity and available booleans from the public JSON. I refactored the code to handle missing fields gracefully rather than assuming a default state.
Node Context Blindness: When wiring the Discord alert after the Google Sheets "Update" node, the expressions returned undefined. I learned that n8n nodes only have context of their direct input. Moving the Discord node to run in parallel with the Update node (both fed by the IF node) solved the data-drift issue.
Infinite Alert Loops: If a database cell is blank, null !== undefined logic can cause a system to think the price "changed" on every single run. I implemented strict type-checking in the JavaScript node to seed the database cleanly on the first run.


---

### 3. What to put in the `/docs` folder

Create a folder named `docs` in your project directory. Recruiters love visual proof. Take screenshots of your working project and save them in this folder:

1. **`workflow-canvas.png`**
   - *What it is:* A screenshot of your n8n canvas showing all the nodes connected.
   - *Why:* Proves you actually built it and understand workflow logic.
2. **`discord-alert.png`**
   - *What it is:* A screenshot of the Discord message showing a successful "PRICE CHANGE DETECTED" alert.
   - *Why:* Proves the end-to-end integration works.
3. **`google-sheet-schema.png`**
   - *What it is:* A screenshot of your Google Sheet showing the `Products` tab and the `Logs` tab with a few rows of dummy data.
   - *Why:* Shows you understand database structuring and state management.

---

### 4. How to Push to GitHub

Open your terminal (or GitHub Desktop) in your project folder and run these exact commands:

```bash
# Initialize the repository (if you haven't already)
git init

# Add all your files (workflow.json, README.md, and the docs folder)
git add .

# Commit with a professional message
git commit -m "feat: Initial release of IntelFlow Lite Shopify Price Monitor"

# Rename branch to main (standard practice)
git branch -M main

# Connect to your GitHub repository (Replace YOUR_USERNAME and YOUR_REPO)
git remote add origin https://github.com/i252610-max/intel-flow-lite.git

# Push the code to GitHub
git push -u origin main
```