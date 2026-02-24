# Secure OpenClaw Setup Guide — macOS (Fresh Install)

**A security-first approach to setting up OpenClaw on a wiped Mac.**

This guide prioritizes doing things right from the start rather than bolting on security after the fact. It covers disk encryption, credential management, network hardening, and ongoing hygiene — everything you need before and after installing OpenClaw.

> **Audience:** Personal use on your own Mac. If you're deploying in an enterprise environment, you'll need additional controls beyond what's covered here.

---

## Table of Contents

1. [Before You Install: Harden the Mac](#1-before-you-install-harden-the-mac)
2. [Install OpenClaw](#2-install-openclaw)
3. [Secure Your Credentials](#3-secure-your-credentials)
4. [OpenClaw Security Configuration](#4-openclaw-security-configuration)
5. [Channel Setup (Messaging)](#5-channel-setup-messaging)
6. [Network & Remote Access](#6-network--remote-access)
7. [Ongoing Security Hygiene](#7-ongoing-security-hygiene)
8. [Priority Checklist](#8-priority-checklist)
9. [Understanding the Risks](#9-understanding-the-risks)
10. [Further Reading](#10-further-reading)

---

## 1. Before You Install: Harden the Mac

Do these before installing OpenClaw or any tooling. They protect everything on the machine, not just OpenClaw.

### 1.1 Enable FileVault (Disk Encryption)

**Why:** If your laptop is lost or stolen, FileVault makes the entire disk unreadable without your password. Without it, anyone can pull the drive and read your API keys, chat history, and memory files.

```
System Settings → Privacy & Security → FileVault → Turn On
```

- Choose "Create a recovery key and do not use my iCloud account" for maximum control
- Store the recovery key somewhere safe (password manager, physical safe)
- Encryption runs in the background — takes a few hours but doesn't interrupt your work

> ⚠️ **This is the single highest-impact security step.** Everything else is secondary if the disk isn't encrypted.

### 1.2 Enable the Firewall

**Why:** Blocks unsolicited inbound connections. OpenClaw's gateway binds to localhost by default, but the firewall adds defense in depth.

```
System Settings → Network → Firewall → Turn On
```

Options to enable:
- ✅ Block all incoming connections (most restrictive — recommended for initial setup)
- Or: Allow only specific apps you trust

### 1.3 Enable Automatic Security Updates

**Why:** macOS security patches close vulnerabilities that could be used to access your files and credentials.

```
System Settings → General → Software Update → Automatic Updates
```

Enable all toggles:
- ✅ Check for updates
- ✅ Download new updates when available
- ✅ Install macOS updates
- ✅ Install Security Responses and system files

### 1.4 Use a Standard User Account (Optional but Recommended)

Create a separate admin account for installations, then use a standard (non-admin) account for daily work. OpenClaw doesn't need admin privileges to run.

```
System Settings → Users & Groups → Add User
```

- Create an admin account (for installs only)
- Downgrade your daily account to Standard

---

## 2. Install OpenClaw

With the Mac hardened, install OpenClaw:

```bash
# Install (handles Node.js detection + OpenClaw setup)
curl -fsSL https://openclaw.ai/install.sh | bash

# Run the onboarding wizard (configures auth, gateway, channels)
openclaw onboard --install-daemon

# Verify it's running
openclaw gateway status
openclaw status
```

The onboarding wizard will ask for:
- An AI provider API key (Anthropic, OpenAI, etc.)
- Optional channel setup (Telegram, Discord, etc.)
- Gateway service installation

> **Tip:** You can skip channel setup during onboarding and add channels later. The Control UI (web dashboard) works immediately without any channel config: `openclaw dashboard`

---

## 3. Secure Your Credentials

This is where most people cut corners. Don't.

### The Problem

OpenClaw and its tools need API keys (Anthropic, OpenAI, etc.). The default approach — exporting them in `~/.zshrc` or storing them in `.env` files — leaves them in plaintext on disk. Any process running as your user can read them.

### Option A: File-Based Isolation (Good)

Better than `.zshrc` exports. Keys are still plaintext but isolated with strict permissions:

```bash
# Create a locked-down secrets directory
mkdir -p ~/.config/secrets
chmod 700 ~/.config/secrets

# Store each key in its own file
echo "sk-ant-your-anthropic-key-here" > ~/.config/secrets/anthropic
chmod 600 ~/.config/secrets/anthropic

# In ~/.zshrc, load dynamically instead of hardcoding
export ANTHROPIC_API_KEY=$(cat ~/.config/secrets/anthropic)
```

**Why this is better:** Keys aren't visible in `cat ~/.zshrc`, shell history, or dotfile backups. Only your user can read them (chmod 600). Combined with FileVault, they're encrypted at rest.

**Limitation:** Still plaintext when the machine is running and you're logged in.

### Option B: macOS Keychain (Better)

Use the built-in macOS Keychain to store secrets encrypted:

```bash
# Store a key in Keychain
security add-generic-password -a "openclaw" -s "anthropic-api-key" -w "sk-ant-your-key-here"

# Retrieve it in ~/.zshrc
export ANTHROPIC_API_KEY=$(security find-generic-password -a "openclaw" -s "anthropic-api-key" -w 2>/dev/null)
```

**Why this is better:** Keys are encrypted by the Keychain, protected by your login password, and never stored in plaintext files. You'll get a one-time prompt to allow terminal access.

### Option C: `pass` with GPG (Best for CLI Power Users)

Full encrypted secret management, git-friendly, works everywhere:

```bash
# Install
brew install pass gnupg

# Generate a GPG key (one-time, follow the prompts)
gpg --full-generate-key

# Initialize pass with your GPG key ID
pass init "your-email@example.com"

# Store secrets (encrypted with GPG)
pass insert providers/anthropic
pass insert providers/openai

# In ~/.zshrc
export ANTHROPIC_API_KEY=$(pass providers/anthropic)
export OPENAI_API_KEY=$(pass providers/openai)
```

**Why this is best:** Keys encrypted at rest with GPG. You can version-control your password store (`pass git init`). Works across machines if you sync your GPG key. Integrates with `pinentry-mac` for GUI prompts.

### What NOT to Do

```bash
# ❌ Never hardcode keys directly in shell config
export ANTHROPIC_API_KEY="sk-ant-abc123..."   # Visible in dotfile backups, git, etc.

# ❌ Never commit .env files with real keys
echo "API_KEY=sk-ant-abc123" > .env && git add .env   # Keys in git history forever

# ❌ Never paste keys into chat messages or documents
```

---

## 4. OpenClaw Security Configuration

### Run the Built-In Security Audit

```bash
# Quick audit
openclaw security audit

# Deep audit (recommended for first setup)
openclaw security audit --deep

# Auto-fix safe defaults (tightens file permissions, doesn't change OS settings)
openclaw security audit --fix
```

The audit checks:
- File permissions on config and credential files
- Gateway bind address (should be localhost)
- Exposed ports and services
- Channel authentication status

### Verify Gateway Is Localhost-Only

```bash
openclaw status
```

Look for: `gateway bind = 127.0.0.1` or `ws://127.0.0.1:18789`

This means the gateway only accepts connections from the local machine. This is the default and should not be changed unless you know exactly what you're doing.

### OpenClaw Config File Permissions

The onboarding wizard should set these correctly, but verify:

```bash
# Config should be owner-only readable
ls -la ~/.openclaw/openclaw.json
# Expected: -rw------- (600)

# Fix if needed
chmod 600 ~/.openclaw/openclaw.json
```

---

## 5. Channel Setup (Messaging)

### Telegram (Recommended for Personal Use)

Telegram with pairing mode is the most straightforward secure channel:

- Create a bot via [@BotFather](https://t.me/BotFather) on Telegram
- Add the bot token during onboarding (or `openclaw onboard`)
- **Pairing mode** ensures the bot only responds to your verified Telegram account
- Messages are encrypted in transit (Telegram's MTProto protocol)

### What to Know About Channels

- Only configured and paired channels can reach your agent
- Random people messaging the bot's Telegram handle won't create a session
- Group chats only work if you explicitly add the bot and configure it
- Each channel binding is tied to a specific chat ID

---

## 6. Network & Remote Access

### Default: No Exposure (Keep It This Way)

By default, OpenClaw's gateway binds to `127.0.0.1` (localhost). Nothing is exposed to your network or the internet. **This is the correct setting for personal use.**

### If You Want Mobile/Remote Access

**DO use Tailscale** (encrypted mesh VPN):

```bash
# Install Tailscale
brew install tailscale

# Authenticate
tailscale up

# Access your OpenClaw from any device on your tailnet
# Your Mac gets a stable Tailscale IP (e.g., 100.x.y.z)
```

Tailscale creates a private encrypted network between your devices. No ports opened, no exposure to the public internet.

**DON'T do any of these:**
- ❌ Port-forward 18789 on your router
- ❌ Expose the gateway to `0.0.0.0` (all interfaces)
- ❌ Use ngrok or similar tunnels without authentication
- ❌ Put the gateway behind a reverse proxy without auth

---

## 7. Ongoing Security Hygiene

### Schedule Periodic Audits

After initial setup, schedule automated security checks:

```bash
# List existing cron jobs
openclaw cron list

# Add a weekly security audit (e.g., every Monday at 9 AM)
openclaw cron add \
  --name "healthcheck:security-audit" \
  --schedule "0 9 * * 1" \
  --task "Run openclaw security audit --deep and report any findings"
```

### Keep OpenClaw Updated

Updates include security patches. Check regularly:

```bash
# Check for updates
openclaw update status

# Update
npm update -g openclaw

# Restart the gateway after updating
openclaw gateway restart
```

> **Tip:** You can configure your agent (via HEARTBEAT.md) to check for and install updates automatically during heartbeat polls.

### Rotate API Keys Periodically

- Anthropic: [console.anthropic.com](https://console.anthropic.com) → API Keys
- OpenAI: [platform.openai.com](https://platform.openai.com) → API Keys
- Revoke old keys after rotating
- Update your secrets store (Keychain, `pass`, or config files)

### Review Workspace Files

Your workspace (`~/.openclaw/workspace/`) contains memory files, conversation context, and potentially sensitive information. Periodically review:

```bash
# Check what's in your workspace
ls -la ~/.openclaw/workspace/
ls -la ~/.openclaw/workspace/memory/

# Ensure workspace isn't world-readable
chmod -R go-rwx ~/.openclaw/workspace/
```

---

## 8. Priority Checklist

| Priority | Action | Time | Impact |
|:---:|--------|:----:|--------|
| 🔴 | **FileVault encryption** | 5 min + background | Protects everything if device is lost/stolen |
| 🔴 | **Firewall on** | 1 min | Blocks unsolicited inbound connections |
| 🔴 | **Encrypted credential storage** (Keychain or `pass`) | 15 min | API keys never sit in plaintext |
| 🟡 | **`openclaw security audit --fix`** | 2 min | Tightens file permissions |
| 🟡 | **Gateway stays on localhost** | Default | Nothing exposed to network |
| 🟡 | **Auto security updates on** | 2 min | OS stays patched |
| 🟢 | **Tailscale for remote access** | 10 min | Encrypted access without exposure |
| 🟢 | **Periodic audit cron job** | 5 min | Automated ongoing checks |
| 🟢 | **API key rotation schedule** | Varies | Limits blast radius of leaked keys |

---

## 9. Understanding the Risks

OpenClaw is a powerful agentic AI system. Being honest about the risk model helps you make informed decisions.

### What OpenClaw Can Do (by design)

- Read and write files on your machine
- Execute shell commands
- Send messages via configured channels
- Browse the web and fetch content
- Access any API keys available in its environment

### The "Lethal Trifecta" (applies to all agentic AI)

Security researcher Simon Willison identified three capabilities that, when combined, create risk:

1. **Access to private data** (files, credentials, memory)
2. **Exposure to untrusted content** (web pages, emails, external data)
3. **Ability to communicate externally** (messaging channels, API calls)

OpenClaw has all three by design — that's what makes it useful. The mitigations are:

- **Untrusted content is labeled** — OpenClaw wraps external content with security markers so the AI model knows not to treat it as instructions
- **Channel authentication** — Only paired/configured users can interact with the agent
- **Gateway isolation** — Localhost binding prevents network-level attacks
- **File permissions** — Credential files are owner-readable only
- **Disk encryption** — FileVault protects everything at rest

### What's NOT Fully Solved (industry-wide)

- **Prompt injection** — A malicious webpage could theoretically embed instructions that trick the AI into performing unintended actions. All agentic AI systems face this. OpenClaw mitigates it with content labeling, but no system has a complete solution yet.
- **Excessive agency** — The agent can run commands you didn't explicitly ask for (by design, for autonomy). Configure tool policies and review workspace files if this concerns you.

### Bottom Line

The risk profile of running OpenClaw on your personal Mac is comparable to running any CLI tool with API keys and shell access. The novel risk is prompt injection, which is an unsolved problem across the entire AI industry. FileVault + encrypted secrets + localhost gateway puts you ahead of the vast majority of installations.

---

## 10. Further Reading

- [OpenClaw Documentation](https://docs.openclaw.ai)
- [OpenClaw Security Audit CLI](https://docs.openclaw.ai) — `openclaw security audit --help`
- [Simon Willison: The Lethal Trifecta](https://simonwillison.net/) — Original analysis of agentic AI risks
- [CrowdStrike: What Security Teams Need to Know About OpenClaw](https://www.crowdstrike.com/en-us/blog/what-security-teams-need-to-know-about-openclaw-ai-super-agent/) — Enterprise perspective
- [Trend Micro: What OpenClaw Reveals About Agentic Assistants](https://www.trendmicro.com/en_us/research/26/b/what-openclaw-reveals-about-agentic-assistants.html) — Industry-wide risk analysis
- [OpenClaw GitHub](https://github.com/openclaw/openclaw) — Source code and issue tracker
- [OpenClaw Community (Discord)](https://discord.com/invite/clawd)

---

*Last updated: February 2026*
*Maintained by the community — contributions welcome via PR.*
