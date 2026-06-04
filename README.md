# Global Tracking

**Deja Vu** is your AI counterpart, running natively on your Mac. It learns how you write, work, and think — then acts autonomously on your behalf. **Forge** (EnvBuilder-MCP) is the developer infrastructure layer that powers remote compute provisioning through a custom MCP server.

---

## Deja Vu

Your personal AI that mirrors your digital identity and operates as an autonomous desktop agent.

### How It Works

Deja Vu builds a multi-layered profile of who you are from your emails, calendar, and public footprint:

- **Voice Profile** — Analyzes your sent emails to capture tone, vocabulary, writing style, emoji usage, and code-switching across contexts (formal vs. casual, internal vs. external)
- **Topic Fingerprint** — Extracts your top recurring interests and domains
- **Behavior Patterns** — Maps reply timing, active hours, initiation ratio, and response tendencies
- **Relationship Graph** — Weighted contact map (inner circle → acquaintances) built from communication frequency and depth
- **Episodic Memory** — Timestamped life events (job changes, travel, milestones) that grow over time

### What It Can Do

- **Computer Use** — Full desktop control via PyAutoGUI (click, type, scroll, screenshot) running in a dedicated macOS user session
- **Browser Control** — Automated browsing via Vercel's agent-browser (Chromium/Playwright), with cookie sync from your real Chrome sessions
- **MCP Connectors** — Gmail (send, reply, draft, summarize), Google Calendar (CRUD), Google Docs & Slides (create, share), Tavily (web search)
- **Native App Control** — AppleScript-driven interaction with any macOS application
- **Voice-Matched Communication** — Drafts and replies match your actual writing style, not generic AI output

### Architecture

Runs as a second macOS user session with two orchestration servers:

| Component | Port | Role |
|-----------|------|------|
| Orchestrator | 8420 | Claude API bridge, tool routing, SSE streaming |
| Agent Server | 8421 | Desktop/browser tools, MJPEG screen capture |
| SecondSelf App | — | SwiftUI notch-resident UI with chat, VNC feed, status |
| Web Dashboard | 3000 | Next.js + React frontend |

### Tech Stack

**Backend:** Python 3.11+, Anthropic SDK (Claude), FastAPI, Firebase Admin, Google APIs, Tavily SDK, Beautiful Soup
**Frontend:** Next.js 16, React 19, TypeScript, Tailwind CSS
**Native:** SwiftUI (macOS menubar app)
**Desktop:** PyAutoGUI, AppleScript, agent-browser (Rust CLI)

### Data & Privacy

No cloud database. Everything stays local:

- Identity profiles written to `~/.secondself/` as Markdown files
- Email cache stored as local JSON with 24h TTL
- File-locked append API for runtime memory
- Optional cloud sync for cross-device continuity, or use the hosted service to skip integration setup

---

## Forge (EnvBuilder-MCP)

A custom-built MCP server that eliminates the pain of provisioning remote servers — SSH into RunPod, Azure, or any remote pod and get a working ML environment without the usual headache.

### Tools

| Tool | What It Does |
|------|-------------|
| `analyze_plan_for_environment` | Deterministic planner with 17+ framework detectors (PyTorch, TensorFlow, JAX, HuggingFace, diffusers, vLLM, etc.) — no LLM needed, fully offline. Azure-aware: surfaces managed alternatives (ML, ACI, AKS) alongside raw VM |
| `connect_machine` | SSH session establishment with classified error codes (`auth_failed`, `connection_refused`, `key_not_found`, etc.) for smart agent retry |
| `execute_setup_step` | Run commands remotely with timeout control (up to 1h). Auto-triggers health check after every step |
| `get_environment_status` | 5 parallel probes in ~7s — SSH liveness, Python version, GPU/CUDA (nvidia-smi), disk space, OS/kernel info |
| `attempt_recovery` | 8 automated recovery strategies (missing modules, pip, CUDA, disk cleanup, git failures) with a hardcoded safety filter blocking 15+ destructive patterns |

### Architecture

```
Copilot / Agent  →  MCP Server (stdio JSON-RPC)  →  ssh2  →  Target Host
                          ↓
                    Next.js Dashboard (mock-ready, WebSocket bridge planned)
```

- **Session lifecycle:** `created` → `connecting` → `ready` → `running` → `closed`
- **Never-throw design:** Health probes catch their own errors; the server never crashes on a bad host
- **Safety-first recovery:** Rejects `rm -rf /`, `reboot`, `mkfs`, fork bombs, `curl | sudo bash`, and more before execution

### Tech Stack

**MCP Server:** TypeScript, Node.js, `@modelcontextprotocol/sdk`, `ssh2`
**Dashboard:** Next.js 16, React 19, Framer Motion, Tailwind CSS, Lucide icons

---

## Getting Started

### Deja Vu

```bash
cd "deja vu"
pip install -r requirements.txt
npm install
python main.py              # Run full identity pipeline
```

**CLI Flags:** `--dry-run` (preview), `--no-cache` (fresh fetch), `--tavily-only` (skip Gmail), `--memory-only` (refresh Layer 2+4)

### Forge

```bash
cd EnvBuilder-Mcp/mcp-server
npm install
npm run dev                 # MCP server (stdio)
```

```bash
cd EnvBuilder-Mcp/ui
npm install
npm run dev                 # Dashboard at localhost:3000
```

---

## Why This Exists

Every minute counts — whether you're at a hackathon racing to provision GPU pods or just tired of context-switching between twelve tabs to get work done. Deja Vu gives you an autonomous counterpart that already knows how you communicate, and Forge gives that counterpart (or you) the ability to spin up and manage remote compute the same way you would, just faster.
