# Local Setup Instructions

## MeridianLink Integration PoC

This guide walks you through setting up and running the MeridianLink Integration Proof of Concept on your local machine. No prior knowledge of the codebase is needed — just follow each step in order.

---

## What You'll Need

Before starting, make sure you have these tools installed:

| Tool | What It Is | How to Install |
|------|-----------|----------------|
| **Node.js** (v18 or later) | The runtime that powers the application | Download from [nodejs.org](https://nodejs.org/) — choose the LTS version. On Mac, you can also run `brew install node` |
| **npm** | The package manager for installing dependencies | Comes automatically with Node.js — no separate install needed |
| **Git** | Version control to pull the code | On Mac: run `xcode-select --install` to get it. Or download from [git-scm.com](https://git-scm.com/) |

**Quick check** — open Terminal and run these commands to verify:

```bash
node -v      # Should show v18.x.x or higher
npm -v       # Should show 9.x.x or higher
git --version
```

If any command says "not found", install that tool first using the links above.

---

## Step 1: Get the Code

Open Terminal and run:

```bash
git clone <REPOSITORY_URL>
cd MeridianLink-Integration-PoC
```

> **Note:** Replace `<REPOSITORY_URL>` with the actual repository URL provided to you.

---

## Step 2: Install Dependencies

While in the project folder, run:

```bash
npm install
```

This downloads all the libraries the application needs (Express, Axios, etc.). It takes about 30 seconds and creates a `node_modules` folder.

---

## Step 3: Set Up Your Environment File

The application reads its settings from a file called `.env`. Create one by copying the template:

```bash
cp .env.example .env
```

Open the `.env` file in any text editor. The key setting is:

```env
# This controls whether the app uses real MeridianLink APIs or simulated data
# Keep this as "true" until live API credentials are provided
USE_MOCK=true
```

**That's it for now.** When live API credentials are available, you'll update this file with:
- `USE_MOCK=false`
- `ML_CLIENT_ID=<your client ID>`
- `ML_CLIENT_SECRET=<your client secret>`

These credentials come from the MeridianLink Vendor Portal.

---

## Step 4: Start the Application

Run:

```bash
npm start
```

You should see this in your terminal:

```
╔══════════════════════════════════════════════════════╗
║   MeridianLink Integration PoC                      ║
╠══════════════════════════════════════════════════════╣
║   Server:    http://localhost:4000                   ║
║   Mode:      SIMULATED (mock)                       ║
║   Env:       development                            ║
╚══════════════════════════════════════════════════════╝
```

The application is now running.

---

## Step 5: Open the Dashboard

Open your web browser (Chrome, Safari, Firefox) and go to:

```
http://localhost:4000
```

You'll see the **Integration Dashboard** — a visual interface showing the document processing pipeline.

---

## Step 6: Run Your First Test

1. You'll see a loan number input field pre-filled with `TEST-001`
2. Click the **"Run Integration"** button
3. Watch the 4-step pipeline execute:
   - **Authenticate** → Connects to MeridianLink (simulated)
   - **Receive** → Fetches 5 sample mortgage documents
   - **Process** → Applies processing stamps to each document
   - **Return** → Uploads processed documents back to MeridianLink
4. The whole flow takes about 15 seconds
5. You'll see a green success message with the results

Each step lights up as it runs, and stats appear showing how many documents were received, processed, and returned.

---

## Step 7: Test via Command Line (Optional)

If you prefer testing without the dashboard, you can use Terminal commands:

```bash
# Check if the server is running
curl http://localhost:4000/api/health

# Run the full integration pipeline
curl -X POST http://localhost:4000/api/integration/run \
  -H "Content-Type: application/json" \
  -d '{"loanNumber": "TEST-001"}'

# View past job results
curl http://localhost:4000/api/integration/history
```

---

## Stopping the Application

To stop the server, go back to the terminal where it's running and press `Ctrl + C`.

---

## Switching to Live Mode (When API Access Is Ready)

Once MeridianLink provides API credentials:

1. Open `.env` in a text editor
2. Change `USE_MOCK=false`
3. Fill in the OAuth credentials:
   ```env
   ML_CLIENT_ID=your_client_id
   ML_CLIENT_SECRET=your_client_secret
   ```
4. Save the file
5. Restart the server: stop it with `Ctrl + C`, then run `npm start` again

The dashboard will now show **"Live"** instead of **"Mock Mode"** in the top-right corner.

---

## Troubleshooting

| Problem | Solution |
|---------|----------|
| **"command not found: node"** | Node.js isn't installed. Download it from [nodejs.org](https://nodejs.org/) |
| **"EADDRINUSE: port 4000"** | Another app is using port 4000. Close it, or change `PORT=4001` in your `.env` file |
| **"Cannot find module"** | Run `npm install` again in the project folder |
| **Dashboard shows "Offline"** | The server isn't running. Go to Terminal and run `npm start` |
| **Pipeline fails in Live mode** | Check that your `ML_CLIENT_ID` and `ML_CLIENT_SECRET` are correct in `.env` |

---

## Summary

| Step | Command | What It Does |
|------|---------|-------------|
| Install dependencies | `npm install` | Downloads required libraries |
| Start the server | `npm start` | Launches the app on port 4000 |
| Open dashboard | Visit `http://localhost:4000` | Visual interface to run the pipeline |
| Stop the server | `Ctrl + C` | Shuts down the application |
