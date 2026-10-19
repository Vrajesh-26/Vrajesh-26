# UI Build Loop: Simple Guide

*Setup and usage for Cursor on Windows.*

## What it does

One skill, `ui-build-loop`, makes your agent work in a fixed loop: plan, build in small steps, test, fix, then give a short report.

- The core rules load for any UI task.
- 3D and motion rules load only for 3D or animation work.
- Lean-CSS rules load only for CSS work, with a before/after size check and a screenshot comparison.

## One-time setup

### 1. Install the skill

1. Download `ui-build-loop.zip` from the chat.
2. Extract it into `C:/Users/<your-name>/.cursor/skills/` (use your own Windows user name).
3. Check that you now have `.cursor/skills/ui-build-loop/SKILL.md`, with the `references` and `scripts` folders beside it.
4. Restart Cursor. Look for `ui-build-loop` in Cursor Settings (Rules / Skills) or type `/` in Agent chat.

### 2. Install helper skills

Run these in your project folder (PowerShell):

```bash
npx skills add https://github.com/anthropics/skills --skill frontend-design
npx skills add https://github.com/vercel-labs/agent-skills --skill web-design-guidelines
npx skills add https://github.com/vercel-labs/agent-skills --skill vercel-react-best-practices
npx skills add obra/superpowers
```

From `obra/superpowers` keep only `verification-before-completion`, `systematic-debugging` and `test-driven-development`.

Only if you use GSAP or 3D:

```bash
npx skills add greensock/gsap-skills
npx skills add https://github.com/emalorenzo/three-agent-skills --skill r3f-best-practices
```

Open each `SKILL.md` before installing. A skill is a set of instructions your agent will follow.

### 3. Install the browser tools

```bash
npm install -g @playwright/cli@latest
playwright-cli install --skills
npm install -D @playwright/test
npx playwright install chromium firefox webkit
```

### 4. Add a short AGENTS.md to each project

```text
# AGENTS.md
Stack: Next.js (App Router), React, TypeScript, <Tailwind or CSS Modules>
Checks: npm run lint | npx tsc --noEmit | npm test | npm run build
Design tokens live in: <path>. Reuse components before creating new ones.
Budgets (edit): CSS 40 KB gzipped, hero 3D model 2 MB, no 3D code in the initial JS.
For any UI work use the ui-build-loop skill.
```

### 5. Optional: MCP servers

File: `%USERPROFILE%/.cursor/mcp.json`

```json
{
  "mcpServers": {
    "context7": { "url": "https://mcp.context7.com/mcp" },
    "figma": { "url": "https://mcp.figma.com/mcp" }
  }
}
```

Context7 gives current library docs. Keep Figma only if you design in Figma first, and use a verified Figma Education account (the free plan allows only 6 tool calls a month). Keep at most two MCPs on.

## How to use

Start your prompt with `use ui-build-loop:`

- New section: `use ui-build-loop: build the About section, editorial style, use existing tokens`
- 3D hero: `use ui-build-loop: add a 3D hero (R3F) showing <object>, with a static fallback, reduced-motion support and a small model`
- Motion: `use ui-build-loop: add scroll motion to Projects, as an enhancement only`
- CSS cleanup: `use ui-build-loop: optimize CSS. Measure size and take screenshot baselines first, trim, then compare on Chrome, Firefox and WebKit at 375, 768 and 1440 px`
- Polish: `use ui-build-loop: polish the hero (spacing, type, hover and focus states)`

## What done means

The agent must show evidence for all of these:

- lint, type check, tests and build pass
- screenshots at 375, 768 and 1440 px look right
- tests pass on Chromium, Firefox and WebKit
- no console errors and no horizontal scroll
- 3D and motion have a static or reduced-motion fallback
- CSS size reported before and after (when CSS changed)

## Token-saving habits

- One task per chat. Mention files with `@file`, never 'look at the repo'.
- Keep at most two MCPs and about six skills enabled.
- Let tests and pixel diffs judge; open only failing screenshots.
- For big tasks ask for the plan first, then approve it.
- If 3 fix loops fail, stop and read the report instead of letting the agent keep trying.

## Troubleshooting

- **Skill not listed:** the file must be exactly `.cursor/skills/ui-build-loop/SKILL.md`. Restart Cursor.
- **`npx` MCP fails on Windows:** set the command to `cmd` and start the args with `/c`, then `npx`.
- **Blank 3D canvas in tests:** wait for the `data-ready` flag, or run `npx playwright test --headed`.
- **Screenshot tests fail on the first run:** create baselines once with `npx playwright test --update-snapshots`.
- **Scroll animations missing in Firefox:** expected. They are an enhancement and content stays visible.
- **Safari-only bug:** Playwright WebKit is close to Safari, not identical. Test on a real iPhone if you can.
