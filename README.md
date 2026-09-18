# kiro_superman

Setup guide for restoring this environment after a clean clone — all items in `vendor/`, `plugins/`, `skills/`, and session data are gitignored and must be reinstalled.

---

## What's gitignored (needs setup)

```
sessions/        # runtime session data
logs/            # agent logs
session-index/   # session index
vendor/          # downloaded plugin sources
plugins/         # installed plugins
skills/          # installed skills
.playwright/     # browser cache
.playwright-cli/ # browser automation cache
```

---

## 1. System prerequisites

### Node.js 18+
Required by `ponytail`, `superpowers`, and the `playwright-cli` skill.

```bash
# Check
node --version   # need 18+
npm --version

# Install via nvm (recommended)
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.7/install.sh | bash
nvm install 18
nvm use 18
```

### Python 3
Required by `ui-ux-pro-max`, `ui-styling`, and `design-system` skills.

```bash
# Check
python3 --version   # need 3.10+

# Install (Ubuntu/Debian)
sudo apt install python3

# macOS
brew install python3
```

### Playwright CLI
Required by the `playwright-cli` skill and the MCP playwright server.

```bash
# Install globally
npm install -g @playwright/mcp

# Or run via npx (already configured in settings/mcp.json)
npx @playwright/mcp@latest
```

Install browser binaries:
```bash
npx playwright install chromium
```

---

## 2. Kiro plugins (vendor/)

### ponytail
Lazy senior dev mode — minimalist coding skill.

```bash
mkdir -p ~/.kiro/vendor
cd ~/.kiro/vendor
git clone https://github.com/DietrichGebert/ponytail.git ponytail
```

Copy the `.env.example` to `.env` if using benchmark scripts:
```bash
cp ~/.kiro/vendor/ponytail/.env.example ~/.kiro/vendor/ponytail/.env
# Edit and set: ANTHROPIC_API_KEY=sk-ant-...
```

### superpowers
Full software engineering methodology (TDD, subagent-driven development, planning).

```bash
cd ~/.kiro/vendor
git clone https://github.com/PrimeRadiant/superpowers.git superpowers
```

---

## 3. Skills (skills/ and ~/.agents/skills/)

Skills live in two locations:
- `~/.kiro/skills/` — Kiro-managed skills (ponytail)
- `~/.agents/skills/` — Agent-level skills (design, UI, playwright)

### ~/.kiro/skills/ponytail/*

Bundled with ponytail. Copy from vendor after cloning:

```bash
mkdir -p ~/.kiro/skills/ponytail
cp -r ~/.kiro/vendor/ponytail/skills/* ~/.kiro/skills/ponytail/
```

Skills: `ponytail`, `ponytail-review`, `ponytail-audit`, `ponytail-debt`, `ponytail-gain`, `ponytail-help`

### ~/.agents/skills/

Custom skills tracked separately. Restore from your dotfiles/backup:

```bash
git clone <your-dotfiles-repo> /tmp/dotfiles
cp -r /tmp/dotfiles/.agents ~/.agents
```

Installed skills and their dependencies:

| Skill | Purpose | Requires |
|-------|---------|---------|
| `playwright-cli` | Browser automation, Playwright tests | `playwright-cli` binary (npm) |
| `ui-styling` | shadcn/ui + Tailwind CSS UI building | Node.js 18+, Python 3.10+ (tests) |
| `ui-ux-pro-max` | UI/UX design intelligence, font/icon/style catalog | Python 3 (stdlib only) |
| `design-system` | Design tokens, component specs, slide generation | Node.js, Python 3 |
| `design` | Logo, CIP, banners, icons, social photos | Node.js, Python 3, Gemini API |
| `brand` | Brand voice, visual identity, messaging | None |
| `slides` | HTML presentations | None |
| `banner-design` | Banner design reference | None |

#### Gemini API key (design + design-system skills)
Logo generation and icon design use Gemini AI:

```bash
export GOOGLE_API_KEY=your-key-here
# or
export GEMINI_API_KEY=your-key-here
```

Add to `~/.bashrc` or `~/.zshrc` to persist.

---

## 4. MCP servers (settings/mcp.json)

Currently configured (file is tracked in git):

```json
{
  "mcpServers": {
    "playwright": {
      "command": "npx",
      "args": ["-y", "@playwright/mcp@latest"]
    }
  }
}
```

Runs via `npx` — no separate install needed as long as Node.js is present.

---

## 5. Kiro agents (agents/)

Agent JSON files are tracked in git. No reinstall needed.

| Agent | Role |
|-------|------|
| `superpowers.json` | Full engineering workflow |
| `ponytail.json` | Minimalist implementation specialist |
| `app-verifier.json` | Acceptance verifier |
| `ui-reviewer.json` | UI/UX reviewer |
| `commit-agent.json` | Conventional commit writer |
| `mirairabo-code-reviewer.json` | Code review specialist |

---

## 6. Quick verification

```bash
# Prerequisites
node --version       # 18+
python3 --version    # 3.10+

# Playwright
npx playwright-cli --version 2>/dev/null || echo "not installed"

# Vendors
ls ~/.kiro/vendor/ponytail/package.json
ls ~/.kiro/vendor/superpowers/README.md

# Skills
ls ~/.agents/skills/
ls ~/.kiro/skills/ponytail/
```
