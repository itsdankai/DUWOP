# DUWOP (duwop)

**DU**sk**W**ood **O**asis **P**ackaged: our own shadcn registry. Every package we drop into our apps,
sites and services lives here, starting with the shared theme. Apps install items with
`npx shadcn add @duwop/<name>`. Public repo `itsdankai/DUWOP` (registry JSON served free from GitHub).

**Status:** live system (never archives). Phases 2 and 3 passed 2026-10-09: items `theme`, `appearance`,
`boot`, `boot-astro`, `design-docs`. Next: phase 4 (publish to GitHub, move MediaTrove onto it, stack
guide). **All 4 phases passed 2026-10-10:** published (itsdankai/DUWOP, PR #2 merged) and MediaTrove runs on it
(PR #40 merged). Open ideas: saved rounds 3 (Stickbird seasonal themes) and 4 (OODA Loop, Scam Baiter). See `plan.md`.
2026-10-11: phase 6 built (Glamours of the Gods + light/dark/system); apps using DUWOP: MediaTrove (full picker) and
Stick Bird's draft (Hades fixed, mode switch only). Website packs are tried in the site lab (`_private/site-lab`,
`npm run dev` there, port 5174), which runs the registry files live with the real Stick Bird products.

## Start here

1. `plan.md` is the definition of done. **Don't start phase N+1 until
   `python ICM/_system/scripts/gate-check.py ICM/Code/duwop/plan.md N` prints GATE OPEN.**
2. `origin/research-brief.md` explains why it's built this way; `origin/theme-package.md` is the original idea.
3. `saved-rounds.md` holds questions parked for the builder. Kept out of the public repo.
4. The builder is new to shadcn: explain each shadcn feature in one line before building on it.

## Layout

| Path | Holds |
|---|---|
| `registry.json` | The list of items and which files each one installs. Edit this to add an item |
| `_engine/registry/theme/` | `duwop.css` (all themes and effects), `theme.ts` (theme/effect/font lists, `applyLook`, `loadLook`, `onLookChange`, `rollPalette`), `duwop-config.ts` (the one file an app edits) |
| `_engine/registry/appearance/` | `appearance.tsx`: the Settings → Appearance picker (only the themes the config allows, the dice, "Use them" for a theme's own font and effects) |
| `_engine/registry/boot/` | `duwop-boot.js` (blocking head script, installs to `public/`; Vite apps add its script tag by hand) and `DuwopBoot.astro` (Astro wrapper that passes the config's key and default look) |
| `_engine/registry/gods/` | `gods.css` + `gods.ts`: Glamours of the Gods (`@duwop/gods`), add-on theme pack for websites. Six gods, each light and dark (`dark: "both"`), Cinzel gilt titles. Apps add `extraThemes: gods` |
| `_engine/registry/mode/` | `mode-toggle.tsx`: light / dark / system switch (`@duwop/mode`) for sites that fix their own look |
| `_engine/registry/popcorn/` | `popcorn.tsx`: opt-in popcorn rain effect (`@duwop/popcorn`), the first add-on item |
| `_engine/registry/anymouse/` | `anymouse.tsx` (`<Anymouse />` box) + `anymouse-server.ts` (`sendAnymouse`, posts to the app's Discord thread): `@duwop/anymouse`, the anonymous idea / bug / gripe box (phase 7) |
| `_engine/registry/design-docs/` | Starter `PRODUCT.md` and `DESIGN.md` for Impeccable, written against DUWOP token names |
| `CONTRIBUTING.md` | Public rules and recipes for contributors. Tyler's Claude Code reads it (this file isn't published), so keep it in step with the rules below |
| `public/r/` | Built output (`npm run build` = `shadcn build`). Committed, because GitHub serves it |
| `origin/` | Seed idea and research brief |
| `_private/` | Never published. `evidence/` holds gate screenshots and the Playwright test scripts |

## How to test an item

Serve the build (`python -m http.server 4174` inside `public/`), make a throwaway Vite or Astro + shadcn
app in the scratchpad, add `"registries": {"@duwop": "http://localhost:4174/r/{name}.json"}` to its
`components.json`, then `npx shadcn add @duwop/<item>`. (A full Windows `C:/...` path to a json fails with
"unknown scheme".) `_private/evidence/test-script.mjs` screenshots every theme and effect and measures font
widths; `picker.mjs` tests the picker and the no-flash boot (`node picker.mjs <url> [astro]`).

**Refining a theme against a real app:** use MediaTrove's theme lab (`ICM/Code/media-trove/_private/theme-lab.mjs`,
see that project's CLAUDE.md). Edit here; MediaTrove's copy updates live. PR both repos only when the round is done.

**Known catch:** `shadcn add ... --overwrite` also overwrites the app's `lib/duwop-config.ts`. After
updating an app, restore that one file from git.

## Rules for this project

- **Modularity is a key feature** (builder, 2026-10-10). Anything only some apps want (popcorn rain, themed
  extras) is its own opt-in item that installs separately; an app turns its effect on with `extraEffects` in
  `duwop-config.ts`. Only what every app needs goes in `theme`.
- Public repo: nothing secret, ever. Publish with `ICM/_system/scripts/publish.sh duwop <branch> "<message>"`,
  never `git init` here. Arm the pre-push guard once per clone: `sh .githooks/install-hooks.sh`.
- Theme selectors are two-part (`.dark[data-theme="x"]`, `[data-theme="x"]:not(.dark)`) so they beat an
  app's own `:root`/`.dark` tokens whatever order the CSS loads. Keep that when adding themes.
- A new theme = one CSS block (dark: `--nh`, `--nc`, `--primary`, `--glow2`; light: same under `:not(.dark)`)
  plus one entry in `themes` in `theme.ts`. Effects must respect `prefers-reduced-motion`.
- A new font needs a measured `adjust` (same sample-line width as Inter, within 3%). Measure, don't guess.
- App-specific changes never go in package files; they go in the app's `duwop-config.ts`.

## Shared repo: auditing Tyler's work (builder, 2026-10-11)

Tyler (GitHub JudgeBreddd) merges work on GitHub too. Nothing of his reaches this folder or `main` unaudited,
and nothing of his gets deleted by our publish.

1. **Before you approve a PR of his**, audit it: `gh pr diff <n> -R itsdankai/DUWOP`, check out the branch in
   a scratch clone, run the audit checklist below, write `_private/audits/pr-<n>.md` ending in
   `Verdict: PASS` or `Verdict: FAIL` with the fixes he needs. Tell the builder; the builder approves on
   GitHub (or tells you to). On FAIL, offer to post the fix list as a PR review comment.
2. **After it merges, bring it in:** `ICM/_system/scripts/sync-in.sh ICM/Code/duwop` lists what's new on GitHub and
   prints the diff. Check it is the audited PR and nothing else, copy the PR audit to
   `_private/audits/<commit>.md` (the commit it names), then `--apply`. It copies his files in, moves any
   file he deleted to `_private/sync-trash/`, and refuses without a passed audit. `conflict` = both sides
   changed a file: merge by hand keeping both, then `--apply --resolved`.
3. `publish.sh` refuses to run while GitHub has work this folder doesn't, so his packs can't be wiped.
   Never delete or rename his packs or theme ids without the builder's explicit OK.

**Audit checklist** (every item, report each as pass/fail):
- Follows every rule in `CONTRIBUTING.md` (modular item, no app-specific code, two-part selectors,
  reduced motion, cheap animation, generic names, licences named).
- No secrets, private hostnames, personal data, binaries or images; no new files outside
  `_engine/registry/`, `registry.json`, `public/r/` unless the PR explains why.
- Doesn't change or remove existing items, themes, ids or exports unless that's the stated point, and then
  no app breaks (MediaTrove, Stick Bird).
- `npm run build` is clean and `public/r/` matches the source; the item installs into a scratch shadcn app
  and every new theme renders in each mode it ships, at 1280 and 390, with no boot flash.
- Contrast: titles 3:1, buttons and text 4.5:1 in every mode. New fonts carry a measured `adjust`.
- Theme blocks use DUWOP's token names, so it works with Impeccable projects as well as UI/UX Pro Max.

## Decisions log

- 2026-10-09: scaffolded; plan, brief, idea and saved rounds moved in from `ICM/_research/theme-package/`.
- 2026-10-09: phase 2 gate passed. Built the `theme` item from MediaTrove's Appearance system (9 themes, AMOLED,
  4 effects, 8 fonts) plus the builder's inbox ideas: Outrun, Cyberpunk, 8-bit, ASCII, Linen (Scandi
  neutrals), a random palette roller ("Roll the dice"), Horizon and Neon effects, Pixelify and Chakra Petch
  fonts. Effect classes renamed: `poster-lift` became `fx-lift`. Licence: MIT (Claude's call, so any app can use it).
- 2026-10-09: font widths re-measured in a fresh app; several MediaTrove `adjust` values were a few percent
  off (Nunito 0.53 → 0.505). MediaTrove picks the new values up when it moves onto DUWOP in phase 4.
- 2026-10-09: phase 3 gate passed. Added `appearance`, `boot`, `boot-astro`, `design-docs`. The theme's CSS
  import is `@import "@/duwop.css"` (alias), because Astro keeps its main CSS in `src/styles/` and
  `./duwop.css` broke there. Boot is split in two items so Vite apps don't get an unused `.astro` file.
  Ponytail audit: one 3-line cut applied.
- 2026-10-09: phase 4, all but the last test. Repo itsdankai/DUWOP created (public). Moving MediaTrove onto it
  forced two package changes: duwop.css now installs into the app's lib folder (`@import "@/lib/duwop.css"`),
  because MediaTrove's app lives in `app/src/`; the picker got `onChange` (user clicks only, so a server save
  can't loop) and a `children` slot for app-specific settings. PR #1 closed in favour of #2. Took about
  8.5 minutes against a 1.5-2 hour estimate.
- 2026-10-10 — Builder: the Horizon theme felt sluggish. Cause: the grid animated background-position on a page-wide 3D-tilted, masked layer with a drop-shadow filter (full repaint every frame). Fix: still .fx-horizon child holds tilt + fade, its ::before grid slides by transform (compositor), glow drawn into the lines. Markup now needs <div class=fx-ambient><div class=fx-horizon/></div>. Headless test: 115-150 → ~228 frames per 4 s, worst stall 247 → 36 ms. DUWOP PR #3, MediaTrove PR #42.
- 2026-10-10 — Builder: scrolling still choppy, Neon invisible, page titles ignore the theme. Neon did nothing because no element had fx-neon (now page h1s + wordmark); h1s get text-primary; ambient wash layer shrunk from 140% to screen size (same look). Headless scroll tests were too noisy to prove the scroll cause (blur made no difference; removing the wash helped in stable runs). Builder to test on the phone by turning effects off one at a time. MediaTrove 43, DUWOP smaller-wash PR.
- 2026-10-10 — **Phase 5 passed (6/6): themes that look the part.** Personalities section in duwop.css (8-bit pixel frames + Press Start 2P titles, ASCII terminal, bolder titles on every theme), fx-title and fx-meter opt-in classes, theme display fonts, lookForTheme (picking a theme sets its font and effects; no Use them). Builder approved the before/after sheet. DUWOP PR #5, MediaTrove PR #44.
- 2026-10-10 — Builder: ASCII's green poster tint was too much; removed (posters keep their colours). DUWOP https://github.com/itsdankai/DUWOP/pull/6, MediaTrove https://github.com/itsdankai/media-trove/pull/45.
- 2026-10-10 (theme lab round, not yet PR'd): no theme glows permanently (Synthwave's card glow removed); Underglow now lights every card on hover in every theme, 8-bit lights its pixel frame; Outrun and ASCII ship with Underglow. New opt-in item popcorn plus extraEffects in duwop-config (add-on effects survive theme switches). Builder: modularity is a key feature. Core audit (builder asked for a full check): only MediaTrove leftover was the default theme name; Trove renamed Duskwood (renamedThemes in theme.ts plus a copy in duwop-boot.js keep old saved looks working; checked in headless Chrome), poster and cinema wording made generic. Nothing else app-specific.
- 2026-10-11: Phase 6 (Glamours of the Gods + light/dark/system). Builder: brand sites get no picker, only a mode switch; the look is ours. New items gods and mode; core: themes can be dark: "both", looks carry mode (system follows the device live, boot resolves it before paint), duwop-config gains extraThemes and loadFonts (false = app self-hosts fonts, no Google calls, for a strict CSP), Underglow now lifts image tiles itself. Builder prefers our three-button switch over shadcn's dropdown mode toggle. First site: Stick Bird draft on Hades with underglow, ambient, motion.
- 2026-10-11: Tyler joins as a contributor (theme packs and more; shadcn + UI/UX Pro Max instead of Impeccable). Added public `CONTRIBUTING.md` as his agent's brief: rules, theme-pack recipe, test steps, branch + PR flow (Kaize merges). Pre-push allowlist widened for it. DUWOP PR #9. Tyler is collaborator JudgeBreddd; main protected (PR required, no force-push, admins exempt).
- 2026-10-11: Builder: audit everything of Tyler's before it reaches main or this folder, never delete his packs. Now: CODEOWNERS (`* @itsdankai`) + 1 approval from a code owner, stale approvals dismissed (PR #11); `sync-in.sh` + publish guard (`synced=`/`published=` in `.publish`; 8/8 checks in a throwaway repo). First audit: Tyler's PR #10 (Command Center pack), FAIL with 5 fixes, see `_private/audits/pr-10.md`.
- 2026-10-15: Phase 7, `@duwop/anymouse` (builder: add the Anymouse to DUWOP so any app can have it, posting to our Discord, one thread per app). One shared webhook `ANYMOUSE_DISCORD_WEBHOOK` + per-app `ANYMOUSE_THREAD_ID` (stored as `ANYMOUSE_THREAD_<APP>`, thread "AM - <App>"). Fully anonymous, mentions off, server-side only (the webhook never reaches the browser). First app: Session Hub. Real Discord post and the PR wait on `_waiting/anymouse-discord-setup.md`.
