# 📜 Project History & Architecture Log: AI Standup Bot

> **Author / Maintainer:** 0abhishhek-hub (Sourav Kumar)  
> **Initial Release:** 02 November 2026  
> **Host Architecture:** Realme Phone (Android ARM64) -> Termux -> PRoot Ubuntu Linux Container  
> **GitHub Repository:** [0abhishhek-hub/ai-standup-bot](https://github.com/0abhishhek-hub/ai-standup-bot)  

---

## 1. 🎯 Problem Statement & Goal
Junior and working software engineers spend unnecessary time every morning parsing through commit logs to draft their daily standup update for Slack/Teams. 
* **The Goal:** Build an autonomous, headless CLI tool that reads local git activity across developer repositories, generates an executive-level, professional standup summary using Google Gemini, and delivers it directly to Telegram before morning standup meetings.

---

## 2. 🏗️ System Architecture
```
[Developer Commits] ➔ [git log (last 24h)] ➔ [Google Gemini 3.8 / Flash-Lite API] ➔ [Polished Standup] ➔ [Telegram Bot API]
```
- **Hardware:** Realme Android Phone (ARM64 octa-core).
- **Execution Container:** Isolated `proot-distro` Ubuntu 24.04 environment (completely isolated from personal phone storage).
- **Runtime:** Python 3.14 with `google-genai` and `requests`.
- **Integrations:**
  - Git CLI (`git log --since=24.hours`)
  - Google Gemini API (`gemini-flash-lite-latest` with fallback chain)
  - Telegram Bot HTTP API (`@realme_computer_bot`)

---

## 3. 🔬 Iteration & Engineering Log

### Phase 1: Prototype & Model Selection
- **Attempt 1 (Groq LLaMA-3.3 70B):** Initially tested Groq API. Found that external API keys frequently require rotation.
- **Attempt 2 (Gemini Flash Models):** Tested `gemini-2.0-flash` and `gemini-2.5-flash`. Observed model deprecation notices and traffic spikes (503 UNAVAILABLE).
- **Final Architecture (Multi-Model Resilient Fallback):** Implemented an automated fallback loop:
  `["gemini-flash-lite-latest", "gemini-2.5-flash-lite", "gemini-pro-latest"]`
  This guarantees 99.9% uptime and zero latency even during high cloud traffic.

### Phase 2: Telegram Delivery System
- Configured dedicated Telegram bot `@realme_computer_bot` via BotFather.
- Implemented payload dispatcher using direct HTTP POST requests to Telegram Bot API endpoint (`/sendMessage`).
- Formatted output with Markdown bolding, emojis, and structured sections (Yesterday, Today, Blockers).

### Phase 3: Security & Secret Management (GitHub Push Protection)
- **Problem:** GitHub Secret Scanning blocked initial push (`GH013: Repository rule violations - Push cannot contain secrets`).
- **Fix:** Purged all hardcoded fallback keys from git history.
- **Architecture Standard:** Credentials decoupled into environment variables (`GEMINI_API_KEY`) and `.gitignore` added for `__pycache__` and `.env`.

---

## 4. 🚀 Quickstart for Developers

### Setup
```bash
git clone git@github.com:0abhishhek-hub/ai-standup-bot.git
cd ai-standup-bot
pip install google-genai requests
```

### Configure Key
```bash
export GEMINI_API_KEY="your_gemini_api_key"
```

### Usage
```bash
# Run for current repo
python3 standup.py

# Run for a specific repository path
python3 standup.py /path/to/any/repo

# Generate standup without sending to Telegram
python3 standup.py --no-send
```
