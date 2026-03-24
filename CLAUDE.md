# CLAUDE.md — PromptPro AI Assistant Guide

## Project Overview

**PromptPro** is a client-side AI prompt optimization tool targeting the Taiwan market. It accepts user prompts, processes them through Claude or GPT-4o APIs via a Google Apps Script proxy, and returns optimized prompts following the "LYRA 4-D" methodology.

- **Architecture:** Single-file SPA (no build toolchain)
- **Language:** Vanilla JavaScript (ES6+), CSS3, HTML5
- **UI Language:** Traditional Chinese (繁體中文)
- **Target Market:** Taiwan (pricing in NT$, Newebpay payment gateway)
- **Backend:** Google Apps Script (GAS) proxy for CORS bypass

---

## Repository Structure

```
/home/user/promptpro/
├── index.html      # Entire application — HTML, CSS, and JS in one file
└── CLAUDE.md       # This file
```

The entire codebase lives in a **single self-contained `index.html`** (~1,840 lines). There are no build tools, package managers, transpilers, test frameworks, or CI pipelines.

---

## Key Architecture

### Single-File Application
All logic is embedded inline in `index.html`:
- Styles in `<style>` block
- Application logic in `<script>` block
- No external JS/CSS dependencies (no npm, no CDN imports)

### API Proxy Pattern
All AI API calls are routed through a Google Apps Script endpoint to work around browser CORS restrictions:
```javascript
const GAS_ENDPOINT = 'https://script.google.com/macros/s/AKfycbz.../exec';
```
Direct calls to Anthropic/OpenAI are not made from the browser.

### Client-Side Storage
No server-side database. All user state lives in `localStorage`:

| Key | Purpose |
|-----|---------|
| `pp_pro` | Pro subscription flag (`"1"`) |
| `pp_expiry` | Subscription expiry timestamp |
| `pp_pro_email` | Subscriber email |
| `pp_plan` | Plan type (`"monthly"` or `"annual"`) |
| `pp_usage` | Daily usage counter (JSON with date + count) |
| `pp_engine` | Selected engine (`"claude"` or `"openai"`) |
| `pp_claude_key` | User-provided Claude API key |
| `pp_openai_key` | User-provided OpenAI API key |

---

## Core Features & Code Locations

All references are line numbers within `index.html`.

### Freemium / Paywall System
- `FREE_LIMIT = 5` — daily free usage cap
- `isPro()` — checks subscription status and expiry
- `getUsage()` / `incrementUsage()` — tracks daily usage via localStorage
- `activatePro(code)` — validates activation codes using djb2 hash
- `makeCode(email, plan)` — generates expected activation code
- Activation secret: `SSQ-PROMPTPRO-2026` (hardcoded)

### AI Engine Abstraction
- `currentEngine` — `"claude"` (default) or `"openai"`
- `selectEngine(eng)` — switches engine, persists to localStorage
- `getApiKey()` — returns user-stored key or falls back to embedded defaults
- `callAI(systemPrompt, userMessage, images)` — unified AI call via GAS proxy
- Default keys for Claude and OpenAI are embedded as fallbacks

### LYRA Prompt Optimization
- 6 optimization modes: `auto`, `detail`, `basic`, `creative`, `tech`, `edu`
- Based on 4-D methodology: Decompose → Diagnose → Develop → Deliver
- `optimizePrompt()` — main entry point; assembles system prompt + user content, calls AI
- Output streamed character-by-character using `simulateStream()`

### Image Upload
- `uploadedImages[]` — stores base64-encoded images
- `handleFiles(files)` — validates and processes files (max 4 images, 5MB each)
- `buildMessageContent(text, images)` — formats content for Claude vs OpenAI API schemas
- Drag-and-drop and click-to-upload supported

### UI / Modal System
- Modals toggled by adding/removing CSS classes (`.active`, `.visible`)
- Modal IDs: `#paywallModal`, `#apiModal`, `#privacyModal`, `#termsModal`, `#contactModal`
- No JS framework; pure DOM manipulation

---

## Development Workflow

### Prerequisites
None. No Node.js, no package manager, no build step required.

### Running Locally
Open `index.html` directly in a browser, or serve it with any static file server:
```bash
python3 -m http.server 8080
# then open http://localhost:8080
```

### Making Changes
Edit `index.html` directly. The file has three logical sections:
1. **HTML markup** — structure and modal templates
2. **`<style>` block** — all CSS including custom properties (design tokens)
3. **`<script>` block** — all application logic

### Git Branches
- `master` — production branch
- `claude/add-claude-documentation-oNIIV` — documentation feature branch
- Feature branches follow the pattern `claude/<description>-<id>`

---

## Code Conventions

### CSS
- Design tokens as CSS custom properties on `:root` (e.g., `--bg-primary`, `--accent`)
- Primary accent color: `#e8691a` (orange)
- Background: `#faf7f2` (warm off-white)
- Flexbox-based layout; no CSS grid
- Responsive via media queries

### JavaScript
- ES6+ syntax (arrow functions, template literals, destructuring, async/await)
- All functions are globally scoped (no modules)
- `async/await` for API calls; `try/catch` for error handling
- `localStorage` accessed via `localStorage.getItem/setItem/removeItem`
- Keyboard shortcut: `Ctrl+Enter` / `Cmd+Enter` triggers optimization

### HTML
- Semantic elements where appropriate (`<header>`, `<main>`, `<section>`)
- Chinese-language UI text — preserve Traditional Chinese for all user-facing strings
- Icons are inline SVG or Unicode characters

---

## Security Considerations

> **Important:** The following items are known design decisions, not bugs to "fix" without discussion with the maintainer.

1. **Embedded API keys** — Default Claude and OpenAI API keys are hardcoded as JavaScript fallbacks. This is intentional to allow zero-config usage but means keys are exposed in the browser.
2. **Client-side activation validation** — The pro subscription activation uses a djb2 hash checked entirely in the browser, providing security through obscurity only.
3. **No server-side auth** — All subscription state is stored in localStorage and can be manually modified.

When making changes, do not accidentally expose secrets in logs or comments, and do not introduce server-side calls without coordinating on the GAS proxy.

---

## Subscription Plans

| Plan | Price | Duration |
|------|-------|---------|
| Monthly | NT$290 | 30 days |
| Annual | NT$2,280 | 365 days |

Payment processed via Newebpay (藍新金流). Activation codes are distributed after payment confirmation.

---

## Company Info

- **Company:** 肌研序 SSQ
- **Contact:** cklein0326@yahoo.com.tw
- **Market:** Taiwan

---

## What AI Assistants Should Know

- **Do not add a build system** unless explicitly requested — the single-file pattern is intentional.
- **Preserve Traditional Chinese** in all UI strings.
- **Do not create additional files** unless necessary; keep everything in `index.html`.
- **No test suite exists** — manually test in browser after changes.
- **The GAS proxy URL is fixed** — do not change it without updating the deployed Apps Script.
- **API keys in the code are intentional defaults** — do not remove them, but do not log or expose them further.
- When editing CSS, use the existing CSS custom properties rather than hardcoding color/spacing values.
- The `simulateStream()` function provides fake streaming; actual API responses return all at once from the GAS proxy.
