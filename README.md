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

# macOS (via Homebrew)
brew install python3

# Ubuntu/Debian
sudo apt install python3
```

### Playwright CLI
Required by the `playwright-cli` skill. Note: this is `@playwright/cli`, **not** `@playwright/mcp`.

```bash
# Install globally (via nvm-managed node)
npm install -g @playwright/cli
```

The binary installs to your active nvm Node path (e.g. `~/.nvm/versions/node/v20.x.x/bin/`).
If `playwright-cli --version` fails after install, nvm is not loaded in your current shell:

```bash
# Load nvm in current shell, then retry
export NVM_DIR="$HOME/.nvm"
source "$NVM_DIR/nvm.sh"
playwright-cli --version   # should print version now
```

To make it permanent, ensure your `~/.zshrc` (macOS default) loads nvm:

```bash
export NVM_DIR="$HOME/.nvm"
[ -s "$NVM_DIR/nvm.sh" ] && source "$NVM_DIR/nvm.sh"
```

The MCP playwright server is separate — it runs via npx and is already configured in `settings/mcp.json`:
```bash
# Already configured, no manual install needed
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

Add to `~/.zshrc` (macOS) or `~/.bashrc` (Linux) to persist.

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

---

## 7. Verify skills are runnable

Run these checks after restoring to confirm each skill can actually execute on this machine.

### playwright-cli

```bash
# Binary must be on PATH
playwright-cli --version
# If missing: npm install -g @playwright/cli
```

### ui-ux-pro-max

Uses Python stdlib only — no pip packages needed.

```bash
python3 ~/.agents/skills/ui-ux-pro-max/scripts/search.py "button design" --max-results 1
# Expected: JSON/text output with style results, no ImportError
```

### ui-styling

Python scripts use stdlib only. The shadcn/ui CLI is invoked per-project (not global).

```bash
# Python check (used for tests)
python3 -c "import csv, json, pathlib; print('ok')"

# Node check (shadcn/ui runs via npx in your project)
node --version   # needs 18+
```

### design-system

Mix of Python and Node scripts. No third-party Python packages.

```bash
# Python scripts
python3 ~/.agents/skills/design-system/scripts/search-slides.py "hero section" 2>&1 | head -5

# Node scripts
node ~/.agents/skills/design-system/scripts/generate-tokens.cjs --help 2>&1 | head -5
```

### design (logo / icon — Gemini AI required)

Logo and icon generation require the `google-genai` Python package and a Gemini API key.

```bash
# Check package
python3 -c "from google import genai; print('google-genai ok')" 2>/dev/null \
  || echo "MISSING: pip install google-genai"

# Check API key
python3 -c "import os; print('key set' if os.environ.get('GEMINI_API_KEY') or os.environ.get('GOOGLE_API_KEY') else 'KEY MISSING')"

# Install package if needed
pip install google-genai
```

### brand

Pure Node.js scripts, no external npm packages.

```bash
node ~/.agents/skills/brand/scripts/extract-colors.cjs --palette 2>&1 | head -5
# Expected: palette output, no "Cannot find module" error
```

### slides / banner-design

No scripts — reference-only skills. Verify the files exist:

```bash
ls ~/.agents/skills/slides/SKILL.md
ls ~/.agents/skills/banner-design/SKILL.md
```

---

### One-liner full check

Paste this to get a pass/fail summary for all skills:

```bash
echo "=== Prerequisites ===" && \
  node --version && python3 --version && \
echo "=== playwright-cli ===" && \
  (playwright-cli --version 2>/dev/null && echo "ok") || echo "MISSING" && \
echo "=== ui-ux-pro-max ===" && \
  python3 ~/.agents/skills/ui-ux-pro-max/scripts/search.py "test" --max-results 1 > /dev/null 2>&1 && echo "ok" || echo "FAIL" && \
echo "=== google-genai (design) ===" && \
  python3 -c "from google import genai" 2>/dev/null && echo "ok" || echo "MISSING: pip install google-genai" && \
echo "=== GEMINI_API_KEY ===" && \
  python3 -c "import os; sys.exit(0 if os.environ.get('GEMINI_API_KEY') or os.environ.get('GOOGLE_API_KEY') else 1)" 2>/dev/null && echo "set" || echo "NOT SET" && \
echo "=== brand scripts ===" && \
  node ~/.agents/skills/brand/scripts/extract-colors.cjs --palette > /dev/null 2>&1 && echo "ok" || echo "FAIL" && \
echo "=== skill dirs ===" && \
  ls ~/.agents/skills/ && ls ~/.kiro/skills/ponytail/
```
