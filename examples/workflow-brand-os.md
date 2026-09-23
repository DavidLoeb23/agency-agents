# Multi-Agent Workflow: Brand OS Build

> Build and extend a brand-centric, design-forward platform (e.g. Hello Brand OS) with Agency agents plus a small stack of Claude Code skills — without losing design consistency along the way.

## The Scenario

You're building a brand operating system: a web app where teams manage brand assets, design tokens, templates, and on-brand content. Three things tend to go wrong on this kind of project:

1. **Brand drift** — every new component invents its own spacing, color, and type choices.
2. **Architecture sprawl** — features get bolted on without a spec, and the component tree fragments.
3. **Unverified UI** — uploads, template editors, and previews "work" in the diff but break in the browser.

The fix is to pair **agents** (who own a role and a point of view) with **skills** (reusable, task-specific instructions Claude loads on demand), and to put a hard gate between each phase.

## Agents vs. Skills

| | Agents (this repo) | Skills |
|---|---|---|
| What it is | A persona with a mission, rules, and deliverables | A packaged set of instructions/resources for one kind of task |
| Where it lives | `~/.claude/agents/` | `~/.claude/skills/` or a plugin |
| Answers | *Who* is doing the work and what "good" looks like | *How* to do a specific task the same way every time |

Use them together: the agent decides, the skill standardizes.

## Recommended Skill Stack

These are commonly used third-party or community skills. Check each skill's source, license, and what it instructs Claude to do before installing it. Names and availability change.

| Need | Skill | Pairs with agent | Notes |
|---|---|---|---|
| High-end UI, no generic "AI slop" | `frontend-design` or `ui-ux-pro-max` | UI Designer, Frontend Developer | Pick one as the default so you don't get two competing style systems |
| Brand adherence | **Your own** brand skill, or `theme-factory` as a starting point | Brand Guardian | The Anthropic `brand-guidelines` skill applies *Anthropic's* brand. Don't use it for your brand. Write a skill that encodes your tokens (see below) |
| Development methodology | `superpowers` (brainstorm → spec → plan → TDD → review) | Software Architect, Code Reviewer | Stops features from shipping without a spec |
| React / Next.js patterns | Vercel React best-practices / composition-patterns skills | Frontend Developer | Only if the app is on React/Next.js |
| Architecture diagrams | `excalidraw-diagram` | Software Architect, Technical Writer | Asset flows, permission models, service maps |
| Browser verification | `webapp-testing` (Playwright) | Evidence Collector, Accessibility Auditor | Screenshots and interaction checks against a running app |

### Write your brand skill first

A generic design skill makes UI look *good*. It doesn't make it look like *your brand*. Give Claude a small skill that is the single source of truth:

```
~/.claude/skills/hello-brand/SKILL.md
---
name: hello-brand
description: Hello Brand OS design tokens and rules. Use for any UI, component,
  template, or content work in the Hello Brand OS codebase.
---
- Colors: only tokens from `tokens/color.json`; never raw hex in components
- Type scale: tokens from `tokens/typography.json`; max 2 families
- Spacing: 4px base grid via Tailwind theme; no arbitrary values (`p-[13px]`)
- Components: extend `ui/` primitives; never fork a primitive for a one-off
- Voice: see `docs/voice.md` for copy in empty states, errors, and CTAs
```

Keep the tokens in the repo (Tailwind theme / CSS variables) and have the skill *point to* them rather than duplicating values.

## Agent Team

| Agent | Role in this workflow |
|-------|---------------------|
| Brand Guardian | Owns the brand contract: tokens, voice, do/don't rules |
| UX Architect | Turns the contract into a CSS/Tailwind foundation and layout system |
| Software Architect | Specs the feature and data model before code |
| UI Designer | Component specs that consume tokens, never raw values |
| Frontend Developer | Builds it |
| Code Reviewer | Reviews against the spec and the brand skill |
| Evidence Collector | Proves it renders correctly, with screenshots |
| Accessibility Auditor | WCAG pass on new UI |
| UI Finish-Gate Reviewer | Final "is this generic?" gate before merge |

## The Workflow

### Phase 0: Brand Contract (once per project)

**Step 0 — Activate Brand Guardian**

```
Activate Brand Guardian.

We're building Hello Brand OS. Produce the brand contract the codebase will enforce:
1. Color tokens (semantic names: surface, text, accent, danger — not "blue-500")
2. Typography scale and font pairing
3. Spacing and radius scale
4. Voice rules for UI copy (empty states, errors, confirmations)
5. Five "never do this" rules a code reviewer can check mechanically

Output tokens as JSON I can drop into tokens/.
```

Then use the output to write the `hello-brand` skill above, and have **UX Architect** wire the tokens into `tailwind.config` / CSS variables.

### Phase 1: Spec (per feature)

**Step 1 — Activate Software Architect** (with the `superpowers` brainstorm/spec flow)

```
Activate Software Architect.

Feature: Brand asset library — upload, tag, version, and approve logos/images.
Roles: Admin, Editor, Viewer.

Deliver a spec: data model, API surface, permission matrix, upload flow
(including failure cases), and an excalidraw diagram of the asset lifecycle.
No code yet.
```

### Phase 2: Design + Build

**Step 2a — Activate UI Designer**

```
Activate UI Designer.

Using the hello-brand skill, spec the components for the asset library:
grid, asset card, upload dropzone, version history drawer, approval badge.
Reference tokens by name only. Reuse existing ui/ primitives wherever possible.
```

**Step 2b — Activate Frontend Developer**

```
Activate Frontend Developer.

Build the asset library from [spec] and [component specs].
Use the hello-brand skill and the React best-practices skill.
Write tests first for the upload and approval flows.
No raw hex, no arbitrary Tailwind values, no new primitives without asking.
```

### Phase 3: Verify

**Step 3a — Activate Code Reviewer**

```
Activate Code Reviewer.

Review this diff against the asset-library spec and the hello-brand rules.
Flag any raw colors, arbitrary spacing, forked primitives, prop drilling,
or request waterfalls.
```

**Step 3b — Activate Evidence Collector** (with `webapp-testing`)

```
Activate Evidence Collector.

Run the app locally. Using Playwright, screenshot the asset library at
375px, 768px, and 1440px, in light and dark mode. Exercise: upload a PNG,
upload an oversized file, approve an asset, open version history.
Report layout shifts, broken states, and anything off-brand, with screenshots.
```

**Step 3c — Activate Accessibility Auditor** in parallel with 3b, then **UI Finish-Gate Reviewer** as the last gate before merge.

## Timeline (per feature)

| Phase | Activity | Agent(s) | Skill(s) |
|------|----------|-------|------|
| 0 | Brand contract (once) | Brand Guardian → UX Architect | your brand skill |
| 1 | Spec + diagram | Software Architect | superpowers, excalidraw-diagram |
| 2 | Component specs | UI Designer | brand skill, frontend-design |
| 2 | Build (TDD) | Frontend Developer | brand skill, React best practices |
| 3 | Review | Code Reviewer | brand skill |
| 3 | Browser proof + a11y (parallel) | Evidence Collector, Accessibility Auditor | webapp-testing |
| 3 | Finish gate | UI Finish-Gate Reviewer | — |

## Key Patterns

1. **Brand contract before components**: tokens live in the repo, and the brand skill points to them. It's the one source of truth every agent reads.
2. **One design skill, not three**: stacking `frontend-design`, `ui-ux-pro-max`, and a theme skill gives you conflicting instructions. Your brand skill wins, and one general design skill fills the gaps.
3. **Spec gate**: nothing gets built without a Software Architect spec. That's what stops the component tree from fragmenting.
4. **Proof gate**: "it works" means screenshots from a real browser, not a passing diff.
5. **Mechanical brand rules**: write rules a reviewer (or a lint rule) can check: no raw hex, no arbitrary values. Taste-based rules drift.
