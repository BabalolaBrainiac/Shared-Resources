# Meanas — LLM Context Broker

**Meanas** is a high-performance, security-first CLI tool designed to act as a context broker between different LLM sessions. It runs silently in the background, captures your interactions with LLM providers, and allows you to "bridge" that context to another session or provider at the speed of a command.

---

## 🚀 Overview

In the modern development workflow, we often switch between different LLM providers (Claude, Gemini, GPT, etc.) or start new sessions within the same provider. Losing the context of previous turns is a significant friction point.

**Meanas** solves this by:
1.  **Observing:** Silently recording your CLI-based LLM interactions (e.g., `claude`, `gemini`).
2.  **Persisting:** Storing these interactions in a secure, local SQLite database.
3.  **Bridging:** Generating a `.meanas-bridge.md` file that summarizes the conversation and contains the most recent turns, ready to be pasted into a new LLM session.

---

## 🛠 Installation & Setup

### 1. Initialize Shell Hook
To automatically capture sessions, `meanas` needs to intercept your LLM command calls. Run the `init` command for your preferred providers:

```bash
# Enable interception for Claude and Gemini
meanas init --claude --gemini
```

This adds a small, zero-latency hook to your shell configuration (`.zshrc`, `.bashrc`, or `config.fish`).

**Supported Shells:** `zsh`, `bash`, `fish`
**Supported Providers:**
- `claude` (aliases: `claude-code`, `anthropic`)
- `gemini` (aliases: `aistudio`, `google`)
- `gpt` (aliases: `openai`, `chatgpt`)
- `kimi` (aliases: `moonshot`)

### 2. Start the Daemon
Meanas relies on a background daemon to handle PTY observation and data persistence.

```bash
meanas daemon start
```

---

## 📖 How It Works

### Architecture
- **Daemon:** A background process that manages active watchers and the SQLite store.
- **Watcher:** Attaches to the PTY (Pseudo-Terminal) of the LLM process in **read-only** mode. It captures output without interfering with your session.
- **Store:** A local SQLite database (`sessions.db`) that stores compressed chunks of your conversation.
- **Bridge:** A Markdown file (`.meanas-bridge.md`) generated during export/migration that acts as the "handoff" mechanism.

### The Lifecycle
1.  You run `claude` in your terminal.
2.  The shell hook detects the command and tells the `meanas` daemon to "begin session" for the current directory.
3.  The daemon starts a watcher for the `claude` process.
4.  When you finish and exit `claude`, the watcher flushes the last bits of context and stops.
5.  Later, you want to move to Gemini. You run `meanas migrate --to gemini`.
6.  Meanas creates `.meanas-bridge.md` in your current directory.
7.  You open Gemini, paste the contents of the bridge file, and continue exactly where you left off.

---

## 💻 Command Reference

### Core Commands

| Command | Description |
| :--- | :--- |
| `meanas init` | Installs/removes shell hooks for provider interception. |
| `meanas ls` | Lists all captured sessions with their token estimates and age. |
| `meanas status` | Shows the status of the session associated with the current directory. |
| `meanas migrate` | **Primary workflow:** Exports the current directory's session to a bridge file. |
| `meanas export <id>`| Exports a specific session (by ID) to a bridge file. |
| `meanas tui` | Opens a terminal user interface to browse and manage sessions. |

### Session Management

| Command | Description |
| :--- | :--- |
| `meanas rm <id>` | Deletes a specific session. |
| `meanas clear` | Deletes all sessions (requires confirmation). |
| `meanas ignore` | Adds the current directory to the ignore list. |
| `meanas compact` | Manually triggers compaction (summarization) of a session. |

### Daemon Control

| Command | Description |
| :--- | :--- |
| `meanas daemon start` | Starts the background daemon. |
| `meanas daemon stop` | Gracefully stops the daemon. |
| `meanas daemon status`| Shows daemon PID, uptime, and active watchers. |
| `meanas daemon ensure`| Starts the daemon if it's not already running (idempotent). |

### Configuration

| Command | Description |
| :--- | :--- |
| `meanas config get <key>` | Retrieves a configuration value. |
| `meanas config set <key> <val>`| Sets a configuration value. |

---

## ⚙️ Configuration

Configuration is stored in `~/.config/meanas/config.toml`.

| Key | Default | Description |
| :--- | :--- | :--- |
| `auto-compact` | `true` | Automatically summarize sessions when they grow large. |
| `compact-threshold`| `8000` | Token limit before auto-compaction triggers. |
| `max-chunks` | `50` | Number of recent raw chunks to keep before summarization. |
| `path` | (XDG Data) | Custom absolute path for the SQLite database. |

---

## 🔒 Security & Privacy

Meanas is designed with a "Security-First" philosophy:
- **Local Only:** No data ever leaves your machine. There is no "cloud sync" or telemetry.
- **Strict Permissions:** All data files (DB, Config, Bridge) are created with `0600` permissions (read/write by you only).
- **Read-Only Observation:** The watcher attaches to the PTY in read-only mode. It cannot inject commands or modify your input.
- **Zero-Secrets Policy:** Error messages and logs are sanitized to prevent leaking session content or credentials.
- **Atomic Writes:** Prevents data corruption and ensures file permissions are set before data is written.

---

## 📂 File Locations

- **Configuration:** `~/.config/meanas/config.toml`
- **Database:** `~/.local/share/meanas/sessions.db`
- **Socket/PID:** `~/.local/run/meanas/` (or `$XDG_RUNTIME_DIR/meanas/`)
- **Bridge Files:** Created as `.meanas-bridge.md` in the directory where you run `migrate` or `export`.

---

## ❓ Troubleshooting

**Daemon not starting?**
Check if a stale PID file exists in `~/.local/run/meanas/meanas.pid`. The daemon uses `flock` to ensure only one instance runs.

**Sessions not appearing?**
1. Ensure the daemon is running: `meanas daemon status`.
2. Ensure the shell hook is installed: `meanas init --<provider>`.
3. Verify the command you are running matches the provider name (e.g., the binary must be named `claude`, `gemini`, etc.).

**Bridge file empty?**
Meanas only captures output from the LLM. If you haven't received any responses in the session yet, there is no context to bridge.
