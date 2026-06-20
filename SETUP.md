# Setup Guide (Windows & Mac)

**No coding experience needed.** This walks you through installing a few free tools and
connecting them, so you can run the Customer Success agent and (optionally) share this
tutorial on GitHub. Plan for about **30–45 minutes**. You'll mostly **copy a command,
paste it, and press Enter**.

> 💡 If a command ever says something like *"command not found"*, the most common fix is to
> **close the Terminal window and open a new one**, then try again. (Newly installed tools
> only appear in fresh windows.)

---

## What you'll install and why

| Tool | What it's for | Plain-English description |
|------|---------------|---------------------------|
| **Claude Code** | Runs the agent | The Anthropic app that does the work and talks to your tools |
| **Node.js** | Required engine | A free runtime that the Salesforce tools need to run |
| **Salesforce CLI** | Read Salesforce | Lets the agent securely read your CRM data (read-only) |
| **Git** | Version control | Tracks files; needed to publish to GitHub |
| **GitHub CLI** | Publish/share | The easiest way to put this tutorial on GitHub |

You only need **Git** and **GitHub CLI** if you want to *share* the tutorial. For just
running the agent, you need Claude Code + Node.js + Salesforce CLI.

---

## Step 0 — Open your Terminal

The "Terminal" is a window where you type commands.

- **Windows:** Click **Start**, type **PowerShell**, press **Enter**.
- **Mac:** Press **Cmd + Space**, type **Terminal**, press **Enter**.

Keep this window open — you'll paste commands into it throughout.

---

## Step 1 — Install Claude Code

The app that runs everything. Download and install it from the official page:
- 👉 **https://claude.com/claude-code**
- Docs / help: https://docs.claude.com/en/docs/claude-code

---

## Step 2 — Install Node.js (the "engine")

Pick the **LTS** (Long-Term Support) version.

**Easiest (any skill level):** download the installer and double-click it:
- 👉 **https://nodejs.org** → click the **LTS** button → run the installer → click Next/Finish.

**Or, by command:**

- **Windows (PowerShell):**
  ```powershell
  winget install OpenJS.NodeJS.LTS
  ```
- **Mac (if you use Homebrew):**
  ```bash
  brew install node
  ```

**Check it worked** (open a NEW Terminal window first):
```bash
node --version
```
You should see something like `v20.x` or `v24.x`.

---

## Step 3 — Install the Salesforce CLI

This is what lets the agent read your Salesforce data. Same command on both Windows and Mac:
```bash
npm install --global @salesforce/cli
```
**Check it worked:**
```bash
sf --version
```
- Learn more: https://developer.salesforce.com/tools/salesforcecli

**Connect your Salesforce org** (opens your browser to log in):
```bash
sf org login web --alias your-org-alias --set-default
```

---

## Step 4 — Install Git (only needed to share on GitHub)

Git tracks file versions and is required to publish to GitHub.

**Easiest:** download the installer:
- 👉 **https://git-scm.com/downloads** → choose your OS → run installer → accept the defaults.

**Or, by command:**

- **Windows (PowerShell):**
  ```powershell
  winget install Git.Git
  ```
- **Mac:** Git often comes with Apple's developer tools. Trigger the install with:
  ```bash
  xcode-select --install
  ```
  (or `brew install git` if you use Homebrew)

**Check it worked** (new Terminal window):
```bash
git --version
```

---

## Step 5 — Install GitHub CLI (only needed to share on GitHub)

The simplest way to create and upload a GitHub repository.

- **Windows (PowerShell):**
  ```powershell
  winget install GitHub.cli
  ```
- **Mac (Homebrew):**
  ```bash
  brew install gh
  ```
- **Or** download from 👉 **https://cli.github.com**

You'll also need a free GitHub account: https://github.com/signup

**Sign in** (opens your browser — just follow the prompts):
```bash
gh auth login
```

---

## Step 6 — Connect your tools inside Claude

Inside Claude Code, connect the services the agent reads from — **Salesforce**, **Pendo**,
and **Slack** — using Claude's connectors/MCP settings. Claude will guide you through
authorizing each one in your browser.
- Learn more: https://docs.claude.com/en/docs/claude-code/mcp

---

## Step 7 — Create the weekly routine

You don't write code for this — you just tell Claude what you want. For example:

> "Every Thursday at 2pm, run the customer success digest and send it to my Slack DM."

Use the ready-made instructions in [`examples/weekly-digest-task.md`](examples/weekly-digest-task.md)
as the routine's prompt (swap in your own placeholders first). Claude saves it as a scheduled
task that runs automatically.

> ℹ️ The routine runs while the Claude app is open. If the app is closed when it's due, it
> runs the next time you open it.

---

## Helpful background (optional reading)

- What is a Terminal / command line? https://www.freecodecamp.org/news/command-line-for-beginners/
- What is Git & GitHub (beginner video & guide): https://docs.github.com/en/get-started/start-your-journey
- What is Node.js? https://nodejs.org/en/about
- Windows package manager (winget): https://learn.microsoft.com/windows/package-manager/winget/
- Homebrew for Mac (optional): https://brew.sh

---

## Troubleshooting

| Problem | Fix |
|---------|-----|
| `command not found` / `not recognized` | Close the Terminal and open a **new** window, then retry. Newly installed tools only show up in fresh windows. |
| Windows: still not found after reopening | Sign out and back in to Windows (refreshes the system PATH). |
| `npm install --global` permission error on Mac | Try again with `sudo npm install --global @salesforce/cli` and enter your Mac password. |
| Not sure which Terminal to use | Windows → **PowerShell**; Mac → **Terminal**. Both work for every command above. |
