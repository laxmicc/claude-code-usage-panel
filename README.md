# Claude Code Usage Panel

A customized Claude Code statusline — a compact two-line panel showing your model,
reasoning effort, project, context-window usage, token count, session time, and a
live **5-hour / 7-day rate-limit quota** with pace tracking. Pure Bash + `jq`,
single file, no Node.js.

> **Customized fork of [Astro-Han/claude-pace](https://github.com/Astro-Han/claude-pace).**
> See [`LICENSE`](LICENSE) for the original license. This fork adds: per-model and
> per-effort colors, a used/total **token counter**, a **session wall-clock**, a
> bold-**amber** highlight for the project path and token count, theme-aware
> coloring (ANSI palette where it adapts, hand-picked truecolor where the palette
> is too dim on dark), and a Windows Git-Bash CRLF/alignment fix.

## Preview

```
Opus 4.8 (1M)  xhigh  |  D:\…\Trading-Codebase (feat/sqx-bridge) 19f +1543 -1805 · 15h4m
▓▓░░░░░░░░  4%  35.4k/1M  |  5h 56% ↓22% 1h   7d 37% ↓31% 2d
```

---

## Reading the panel

### Line 1 — Identity & Project

| Element | Meaning |
|---|---|
| `Opus 4.8 (1M)` | Active **model** + its **context-window size** (`1M` = 1,000,000 tokens). Color = model family (see legend). |
| `xhigh` | Reasoning **effort level** (`low` / `medium` / `high` / `xhigh` / `max`). Color = heat gradient. |
| `\|` | Separator (dim) between the identity (left) and project (right) sections. |
| `D:\…\Trading-Codebase` | **Project folder** (your working dir), shown in **bold amber** as the focal point. Full path appears on Windows because backslash paths have no `/` to trim. |
| `(feat/sqx-bridge)` | Current **git branch**. |
| `19f` | **Files changed** in the working tree (uncommitted, vs `HEAD`). |
| `+1543` / `-1805` | Lines **added** (green) / **removed** (red) in that uncommitted diff. |
| `· 15h4m` | **Session wall-clock** — how long this Claude Code session has run (cyan = time info). |

### Line 2 — Budget & Usage

> **Left of the `\|` = this conversation's context. Right of the `\|` = your account's rate limits.** They measure different things — don't conflate them.

| Element | Meaning |
|---|---|
| `▓▓░░░░░░░░` | **Context-window fill bar** (10 cells, eighth-block precision). Green < 70%, yellow 70–90%, red 90%+. |
| `4%` | Percent of the **context window** used by the current chat. |
| `35.4k/1M` | **Token counter** — input tokens used / window size (bold amber). How close this chat is to filling up / compacting. |
| `\|` | Separator (dim). |
| `5h` / `7d` | The two **rate-limit windows** — rolling 5-hour and 7-day account quotas. |
| `56%` / `37%` | Percent of that quota **already used**. Green = healthy, yellow ≥ 70%, red ≥ 90%. |
| `↓22%` / `↓31%` | **Pace delta.** **↓ green = under pace** (spending slower than the clock → surplus). **↑ red = over pace** (spending faster than the window elapses → you'll hit the cap before reset). |
| `1h` / `2d` | **Time until that window resets** (cyan). |

---

## Color legend

| Color | Used for |
|---|---|
| **Model name** | Opus = purple-magenta · Sonnet = azure · Haiku = green · Fable = yellow · other = cyan |
| **Effort** | low = green · medium = cyan · high = yellow · xhigh = red · max = **bold** red |
| **Health** (bar, quota %) | green < 70% · yellow 70–90% · red 90%+ |
| **Pace arrow** | green ↓ = under pace (good) · red ↑ = over pace (slow down) |
| **Bold amber** | project path + token counter (the two "find-it-fast" numbers) |
| **Cyan** | time info — session clock, reset countdowns |
| **Dim** | separators |

Colors use the terminal's ANSI palette (which your theme remaps for light/dark)
wherever it stays readable; a few (model magenta/blue, the amber highlight) are
fixed mid-tone truecolor because those palette slots wash out or go dim on one of
the two backgrounds.

---

## How to interpret it

- **The pace arrows are the steering signal.** Green ↓ means your current burn
  rate is sustainable until the window resets; red ↑ means you'll be throttled
  before reset unless you slow down. Example: `5h 56% ↓22% 1h` = you've used 56%
  of the 5-hour quota with ~80% of the window already elapsed, so you're ~22%
  *under* pace — plenty of headroom for the last hour.
- **Context (left) and quota (right) are independent.** A near-full *context bar*
  means *this chat* is getting long (may compact soon); it has nothing to do with
  your *account* 5h/7d limits.
- **Amber = the numbers you most often want to find fast** (where you are, how
  full the chat is). **Green→yellow→red anywhere = health** (good → attention).

---

## Install

Needs **`jq`** and Claude Code **2.1.80+** (for live rate-limit data).

**1. Download the script**

```bash
# macOS / Linux
curl -sSL -o ~/.claude/statusline.sh \
  https://raw.githubusercontent.com/laxmicc/claude-code-usage-panel/main/usage-panel.sh
chmod +x ~/.claude/statusline.sh

# Windows (Git Bash)
curl -sSL -o "$HOME/.claude/statusline.sh" \
  https://raw.githubusercontent.com/laxmicc/claude-code-usage-panel/main/usage-panel.sh
```

**2. Install `jq`**

```bash
brew install jq          # macOS
sudo apt install jq      # Debian/Ubuntu
winget install jqlang.jq # Windows
```

**3. Point Claude Code at it** — in `~/.claude/settings.json`:

```jsonc
// macOS / Linux
"statusLine": { "type": "command", "command": "~/.claude/statusline.sh" }

// Windows
"statusLine": { "type": "command", "command": "bash C:/Users/<you>/.claude/statusline.sh" }
```

Restart Claude Code (or wait for the next status refresh).

---

## Credits

Built on **[claude-pace](https://github.com/Astro-Han/claude-pace)** by Astro-Han —
all the rate-limit / pace machinery is theirs. This repo is a personal customized
fork; see [`LICENSE`](LICENSE).
