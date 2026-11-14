# Portfolio Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Build Meinard Ga's portfolio: a direction C ("Poster") one-page site with a live 3D Stack in the hero only, plain-HTML sections below, and MDX case study pages.

**Architecture:**
- Next.js App Router, statically rendered. All content is server-rendered HTML.
- The hero's 3D is a client-only React Three Fiber canvas, loaded after the text. It falls back to a still image on reduced motion, no WebGL, low memory or a low frame rate.
- Project data lives in one TypeScript file. Case study bodies are MDX, loaded by dynamic import (no frontmatter plugin).

**Tech Stack:** Next.js 16.4, React 19.3, TypeScript, Tailwind CSS 4, pnpm, three + @react-three/fiber + @react-three/drei, @next/mdx, Vitest + Testing Library (jsdom), Playwright, @vercel/analytics.

**Spec:** `docs/superpowers/specs/2026-11-15-portfolio-design.md` (approved 2026-11-15). Read it before any task.

## Global Constraints

- **Git belongs to Meinard.** Never run `git commit`, `git push`, branch or PR commands. Each task ends with a checkpoint: list the changed files and a suggested commit message for Meinard.
- **Next.js 16 differs from your training data.** Read the matching guide in `node_modules/next/dist/docs/` before writing Next.js code (see `AGENTS.md`). MDX guide: `01-app/02-guides/mdx.md`.
- **Skills to load before UI work, in order:** `design-taste-frontend`, `ui-ux-pro-max`, `apple-design`, then `impeccable` (read its craft floor before editing). Load `ponytail` (full) for all coding.
- **Graphify:** the repo has no `graphify-out/` yet, so read files directly.
- **Font:** Schibsted Grotesk only, via `next/font/google`, weights 400, 600 and 800. Display: 800, tracking about -0.05em, leading about 0.86. Text: 16–18px, leading 1.5–1.55. At most five sizes per page.
- **Colour tokens:** exactly the spec §5 table, as CSS custom properties `--bg --surface --ink --muted --accent --accent-ink`, light on `:root` and dark under `[data-theme="dark"]` and `prefers-color-scheme: dark` (when no explicit choice is stored).
- **House rules (spec §5):**
  - No eyebrows, index numbers, gradients, glass panels, hairline rules, icon cards, resting shadows, monospace outside code, or tiny tracked uppercase labels.
  - Buttons are flat teal with square corners.
  - Headlines fit in two lines or fewer at every width.
  - Equal outer margins at every width.
- **Copy:**
  - Every fact comes from the spec or the CV (`~/Documents/PDF/Meinard_Ga_Resume.pdf`).
  - Never show the phone number on the site.
  - Email: `jobga1234@gmail.com`. LinkedIn: `https://www.linkedin.com/in/meinard-ga-5629a3387`. GitHub: `https://github.com/Hyuga18`.
  - AniUgnay: `https://aniugnay.vercel.app`. Brewscout: `https://brewscout-hyuga.vercel.app`.
- **Motion:**
  - Transform and opacity only. Critically damped easing with no bounce.
  - Every animation has a `prefers-reduced-motion` fallback.
  - Content is never hidden when JavaScript fails or is off.
- **Site URL:** from `process.env.NEXT_PUBLIC_SITE_URL`, falling back to `https://portfolio-hyuga18.vercel.app`. Set it in Vercel once the domain exists.
- **Drafts:** a project with `draft: true` appears only when `NODE_ENV !== "production"`.
  - This replaces the spec's "front matter" wording: the flag lives in `content/projects.ts`, because `@next/mdx` has no frontmatter.

## Review Focus

1. **JavaScript off, or the 3D chunk fails to load.** All text, the photo and the still Stack still show, and nothing stays at opacity 0 (Task 4 and Task 8 e2e tests).
2. **Clipboard refused** (older Safari, embedded browsers). "Copy email" selects the address text and says "Press Ctrl+C / ⌘C to copy" instead of failing silently (Task 3 unit test).
3. **localStorage throws** (private mode) or the system theme changes mid-visit. The theme still resolves, with no flash on load, and the stored choice wins over the system (Task 1 unit and e2e tests).
4. **Someone opens a draft case study URL on production.** It returns 404, and the draft is missing from the work list and the sitemap (Task 9 and Task 10 tests).
5. **A 320px-wide phone.** No horizontal scroll, every headline at two lines or fewer, and the "Meinard"/"Ga" type stays inside the frame (Task 11 e2e test).

---

### Task 1: Tooling, tokens, font and theme

**Files:**
- Modify: `package.json`, `app/layout.tsx`, `app/globals.css`, `app/page.tsx` (clear the starter content)
- Create: `vitest.config.ts`, `playwright.config.ts`, `lib/theme.ts`, `components/ThemeToggle.tsx`, `lib/theme.test.ts`, `e2e/theme.spec.ts`
- Delete: the starter SVGs in `public/` that nothing uses

**Interfaces:**
- Produces:
  - `type ThemeChoice = "light" | "dark" | "system"`
  - `resolveTheme(choice: ThemeChoice, systemDark: boolean): "light" | "dark"`
  - `readStoredTheme(storage: Pick<Storage,"getItem"> | null): ThemeChoice` (returns `"system"` on a missing value, a bad value, or a throw)
  - `THEME_KEY = "theme"`
  - `themeInitScript: string`, an inline script that sets `document.documentElement.dataset.theme` before first paint
  - `<ThemeToggle />` cycles system → light → dark and sets `aria-label` to "Theme: System", "Theme: Light" or "Theme: Dark"
- Produces scripts: `test` (vitest run), `test:e2e` (playwright test), `typecheck` (tsc --noEmit)

- [ ] **Step 1:** `pnpm add -D vitest @vitejs/plugin-react jsdom @testing-library/react @testing-library/user-event @playwright/test` and `pnpm exec playwright install chromium`. Vitest: jsdom environment, include `**/*.test.ts(x)`. Playwright: `webServer` runs `pnpm build && pnpm start -p 3100`, base URL `http://localhost:3100`, projects for desktop 1440×900 and mobile 390×844.
- [ ] **Step 2: Write the failing tests** in `lib/theme.test.ts`:
  - `resolveTheme("system", true) === "dark"`
  - `resolveTheme("light", true) === "light"`
  - `readStoredTheme({ getItem: () => { throw new Error() } }) === "system"`
  - `readStoredTheme({ getItem: () => "purple" }) === "system"`
  - `readStoredTheme(null) === "system"`
- [ ] **Step 3:** Run `pnpm test lib/theme.test.ts`. Expected: FAIL (module missing).
- [ ] **Step 4:** Implement `lib/theme.ts`. Add the tokens to `app/globals.css`, using Tailwind 4 `@theme inline` to map the tokens to `bg-bg`, `text-ink`, `bg-accent` and so on. Load Schibsted Grotesk in `app/layout.tsx` as the `--font-sans` variable and inject `themeInitScript` in `<head>`. Add the themed `::selection`, `:focus-visible` (2px accent outline, 3px offset) and `scrollbar-color`. Theme changes ease `background-color` and `color` over 400ms.
- [ ] **Step 5:** Run `pnpm test`. Expected: PASS.
- [ ] **Step 6: Write `e2e/theme.spec.ts`:**
  - with `colorScheme: "dark"` and nothing stored, `html[data-theme]` is `"dark"` on the first `domcontentloaded`
  - after clicking the toggle to Light and reloading, it is `"light"` even with a dark system
  - the toggle is reachable by Tab and changes on Enter
- [ ] **Step 7:** Run `pnpm test:e2e e2e/theme.spec.ts`. Expected: PASS. Run `pnpm lint && pnpm typecheck`. Expected: clean.
- [ ] **Step 8: Checkpoint for Meinard.** Suggested message: `chore: tooling, tokens, Schibsted Grotesk and theme toggle`.

### Task 2: Site and project data

**Files:**
- Create: `lib/site.ts`, `content/projects.ts`, `content/projects.test.ts`

**Interfaces:**
- Produces from `lib/site.ts`:
  - `SITE_URL: string` (see Global Constraints)
  - `PERSON = { name: "Meinard Ga", role: "Full-stack developer", availability: "Available full-time from April 2027", email, linkedin, github, cvPath: "/Meinard_Ga_Resume.pdf" }`
- Produces from `content/projects.ts`:
  - `type Project = { slug: string; name: string; summary: string; role: string; stack: string[]; live: string; repo?: string; dates: string; layers: ("interface" | "api" | "data")[]; screenshot: { src: string; alt: string }; draft: boolean; caseStudy: boolean }`
  - `PROJECTS: Project[]`, in this order: `aniugnay` (caseStudy true, draft true), `brewscout` (caseStudy true, draft true), `csc-live` (caseStudy false, no live link needed, so `live` is `""`)
  - `listProjects(opts?: { includeDrafts?: boolean }): Project[]` (`includeDrafts` defaults to `process.env.NODE_ENV !== "production"`; non-case-study entries are always listed)
  - `getProject(slug: string, opts?): Project | undefined`, same draft rule
  - `nextProject(slug: string, opts?): Project | undefined`, the next case-study project, wrapping around

- [ ] **Step 1: Write the failing tests:**
  - `listProjects({ includeDrafts: false })` contains no project with `draft && caseStudy`
  - `getProject("aniugnay", { includeDrafts: false })` is `undefined`
  - `getProject("aniugnay", { includeDrafts: true })?.live === "https://aniugnay.vercel.app"`
  - `nextProject("brewscout", { includeDrafts: true })?.slug === "aniugnay"`
  - no string anywhere in `PROJECTS` or `PERSON` contains `"976"` (the phone number)
- [ ] **Step 2:** Run `pnpm test content`. Expected: FAIL.
- [ ] **Step 3:** Implement. Copy summaries, roles and stacks from the CV and spec §8. `screenshot.src` is `/work/aniugnay.jpg`, `/work/brewscout.jpg`, `/work/csc-live.jpg` (the last one is unused for now).
- [ ] **Step 4:** Run `pnpm test`. Expected: PASS.
- [ ] **Step 5: Checkpoint.** Suggested message: `feat: site and project data`.

### Task 3: Home sections below the hero (no motion yet)

**Files:**
- Create:
  - `components/sections/Work.tsx`, `Experience.tsx`, `About.tsx`, `Education.tsx`, `Contact.tsx`, `Footer.tsx`, `components/Nav.tsx`
  - `components/CopyEmail.tsx`, `components/CopyEmail.test.tsx`
  - `e2e/home.spec.ts`
- Modify: `app/page.tsx`

**Interfaces:**
- Consumes: `PERSON`, `listProjects()`, `<ThemeToggle />`
- Produces:
  - `<CopyEmail email={string} />`. It calls `navigator.clipboard.writeText` inside the click handler. On success it shows "Copied" for 2 s through an `aria-live="polite"` region. On rejection or a missing API it selects the email text node and shows "Press Ctrl+C / ⌘C to copy".
  - Section ids `work`, `experience`, `about`, `education`, `contact`. Nav links point to the first four plus `contact`.

- [ ] **Step 1: Write the failing tests** in `CopyEmail.test.tsx`:
  - with `writeText` resolving, clicking shows "Copied"
  - with `writeText` rejecting, the text "Press Ctrl+C / ⌘C to copy" appears and `window.getSelection()?.toString() === "jobga1234@gmail.com"`
  - with `navigator.clipboard` undefined, it behaves the same as rejecting
- [ ] **Step 2:** Run. Expected: FAIL.
- [ ] **Step 3:** Implement `CopyEmail`, then the sections, per spec §8:
  - **Work:** a full-screen panel per case-study project (screenshot about 60% of the width, `next/image`, `sizes` set), the name in poster type, a one-line summary with role and stack, and "Read the case study" plus "Open the live site" links. CSC L.I.V.E. is a smaller entry with no panel.
  - **Experience:** the four CV numbers on one row ("290+", "100+", "4s → 0", "~50×", each with its label), then one paragraph each for Ticketo, T201 HRIS and Tito Awards.
  - **About:** a portrait slot (4:5, `/me-portrait.jpg`; until the real photo exists use the hero stand-in) beside a short placeholder paragraph marked `{/* DRAFT: Meinard writes this */}`, plus skills as plain text groups and certifications on one line.
  - **Education:** a compact band.
  - **Contact:** the email as display type, a `mailto:` link, `<CopyEmail />`, CV, LinkedIn and GitHub links.
  - **Footer:** "Meinard Ga" at full width and the year.
  - Neighbouring sections use different layouts; the surface tone separates bands.
- [ ] **Step 4:** Run `pnpm test`. Expected: PASS.
- [ ] **Step 5: Write `e2e/home.spec.ts`:**
  - the page has exactly one `h1` with accessible name "Meinard Ga"
  - each section id has an `h2`
  - the page text does not contain "976"
  - the CV link `href` is `/Meinard_Ga_Resume.pdf`
  - every external link has `rel` containing `noopener`
- [ ] **Step 6:** Run `pnpm test:e2e e2e/home.spec.ts`. Expected: PASS (the `h1` exists once Task 5 lands; until then, mark that one test `test.fixme`).
- [ ] **Step 7: Checkpoint.** Suggested message: `feat: home sections and copy email`.

### Task 4: Section entrance motion

**Files:**
- Create: `components/Reveal.tsx`, `e2e/no-js.spec.ts`
- Modify: `app/globals.css`, the section components from Task 3

**Interfaces:**
- Produces: `<Reveal as?: keyof JSX.IntrinsicElements; delay?: 0 | 1 | 2>`, a client component.
  - It adds `data-reveal` on mount and `data-revealed` when 15% of it enters the viewport (one IntersectionObserver per element, disconnected after the first reveal).
  - CSS hides it only under `html.js [data-reveal]:not([data-revealed])`.
  - `html.js` is set by the theme init script.
  - Stagger: 0, 75 and 150ms. Rise: 24px. Duration: 600ms, `cubic-bezier(.16,1,.3,1)`.
  - Under reduced motion it fades only, with no rise.

- [ ] **Step 1: Write `e2e/no-js.spec.ts`** with `javaScriptEnabled: false`: every section `h2` and the contact email are visible with computed opacity 1.
- [ ] **Step 2:** Run. Expected: PASS already (it guards against regressions); if it fails, fix before continuing.
- [ ] **Step 3:** Implement `Reveal` and wrap each section's heading, text and picture in order with delays 0, 1, 2. Add press feedback: buttons get `active:scale-[.97]` with a 100ms transition. Work screenshots scale from 1.03 to 1 as they reveal.
- [ ] **Step 4:** Run `pnpm test:e2e`. Expected: PASS, with the no-JS test still green.
- [ ] **Step 5: Checkpoint.** Suggested message: `feat: section entrance motion`.

### Task 5: Static hero (type, photo, intro), no 3D yet

**Files:**
- Create: `components/hero/Hero.tsx`, `components/hero/placement.ts`, `scripts/lift-subject.swift`, `public/me.png` (generated), `public/Meinard_Ga_Resume.pdf` (copied)
- Modify: `app/page.tsx`

**Interfaces:**
- Produces:
  - `STACK_PLACEMENT = { desktop: { left: "57%", top: "-6%", width: "62vw", aspectRatio: "4 / 3" }, mobile: { left: "50%", top: "0%", width: "120vw", aspectRatio: "1 / 1" } }`, the backdrop layout from spec §12.1, cropped by the top edge
  - `<Hero stack?: React.ReactNode />` renders `stack` into the placement box
  - z-order, back to front: type (`z-0`), stack box (`z-10`), photo (`z-20`), intro and nav (`z-30`)

- [ ] **Step 1:** Generate the stand-in cut-out: `swiftc -O scripts/lift-subject.swift -o /tmp/lift && /tmp/lift ~/Downloads/IMG_3262.JPG public/me.png`. The script uses `VNGenerateForegroundInstanceMaskRequest` with `generateMaskedImage(..., croppedToInstancesExtent: true)` and writes an RGBA PNG. Then soften the edge: `ffmpeg -i public/me.png -filter_complex "[0]format=rgba,split[c][a];[a]alphaextract,erosion,erosion,gblur=sigma=1.1[m];[c]eq=contrast=1.12:saturation=1.06:gamma=0.96[g];[g][m]alphamerge" -y public/me.png`. Check it by eye on putty and near-black.
- [ ] **Step 2:** Copy the CV: `cp ~/Documents/PDF/Meinard_Ga_Resume.pdf public/`.
- [ ] **Step 3:** Implement `Hero` per spec §7 and the hero mockup:
  - **Type:** one `h1` with accessible name "Meinard Ga". "Meinard" top-left at `clamp(64px, 15.5vw, 236px)`; "Ga" bottom-right.
  - **Photo:** `next/image` with `priority`, right of centre (`left: 57%`), `height: 92%` of the hero, bottom-aligned, alt "Meinard Ga in a dark suit, looking to the side".
  - **Dark-mode rim light:** a `<span>` masked by `public/me-rim.png`, accent fill, opacity 0.55, `mix-blend-mode: screen`. Generate the rim mask with the ffmpeg recipe in this task's notes below.
  - **Intro, bottom-left:** "Full-stack developer · Available full-time from April 2027", "React and Next.js in front, NestJS and Postgres behind.", "See selected work" (to `#work`) and "Download CV".
  - **Nav:** top-right.
  - **Height:** `calc(100svh - nav)` on desktop. On phones: "Meinard", then the photo and stack (64svh), then "Ga" right-aligned, then the intro.
- [ ] **Step 4:** Un-fixme the `h1` test from Task 3 and run `pnpm test:e2e e2e/home.spec.ts`. Expected: PASS.
- [ ] **Step 5:** Run `node ~/.claude/skills/design-taste-frontend/scripts/inspect.mjs http://localhost:3100 --widths 1440,390` against `pnpm start`. Fix every FAIL that isn't the intended photo-over-type overlap; pass `--allow` for that rule only, and say so in the checkpoint.
- [ ] **Step 6: Checkpoint.** Suggested message: `feat: poster hero with stand-in photo`.

Notes, rim mask recipe:

```
ffmpeg -i public/me.png -filter_complex "[0]format=rgba,alphaextract,split[a1][a2];[a2]erosion,erosion,erosion,erosion,erosion,gblur=sigma=2[e];[a1]erosion,erosion,gblur=sigma=1.1[x];[x][e]blend=all_mode=subtract,gblur=sigma=1.5,geq=lum='lum(X\,Y)*(1-0.85*Y/H)'[r];color=c=white:s=WxH,format=rgba[w];[w][r]alphamerge" -frames:v 1 -y public/me-rim.png
```

Replace `WxH` with `public/me.png`'s size.

### Task 6: The Stack scene (geometry, materials, motion maths)

**Files:**
- Create: `components/stack/geometry.ts`, `components/stack/StackScene.tsx`, `components/stack/motion.ts`, `components/stack/motion.test.ts`

**Interfaces:**
- Consumes: the colour tokens (read with `getComputedStyle(document.documentElement)`)
- Produces from `motion.ts` (pure functions):
  - `OPENING = { desktop: 2.0, mobile: 1.2 }` (seconds)
  - `layerDrop(t: number, layer: 0 | 1 | 2, total: number): number`, the 0–1 progress of a layer. Layer 2 (data) starts first, then 1, then 0; each starts `total * 0.11` after the previous one and lasts `total * 0.35`, eased out cubic.
  - `photoProgress(t, total)` starts at `total * 0.58`.
  - `typeProgress(t, total)` starts at `total * 0.45`.
  - `SPIN_RAD_PER_S = (2 * Math.PI) / 90`
  - `damp(current: number, target: number, lambda: number, dt: number): number`, framerate-independent exponential damping (`current + (target - current) * (1 - Math.exp(-lambda * dt))`)
- Produces from `StackScene.tsx`:
  - `<StackScene theme: "light" | "dark"; openingT: number; opening: number; pointer: { x: number; y: number }; reducedMotion: boolean />`, to be rendered inside an R3F `<Canvas>`
  - Geometry as spec §6:
    - interface: a 2.6×2.6×0.16 rounded plate with three raised panes
    - API: a 2.3×2.3×0.3 plate, four port cylinders and a ring
    - data: three discs of radius 1.15
  - Materials:
    - light: `MeshPhysicalMaterial` glass, transmission about 0.45, roughness 0.28, clearcoat 0.8, colour `#d6e6e8`
    - dark: `MeshStandardMaterial`, colour `#1a1d1e`, low emissive in the accent colour, accent edge lines (`EdgesGeometry`), plus an additive glow sprite
  - No postprocessing.

- [ ] **Step 1: Write the failing tests** in `motion.test.ts`:
  - `layerDrop(0, 2, 2) === 0`
  - `layerDrop(2, 0, 2) === 1`
  - `layerDrop(0.3, 2, 2) > layerDrop(0.3, 0, 2)` (data drops first)
  - `photoProgress(1.0, 2) === 0`
  - `photoProgress(2, 2) === 1`
  - `damp(0, 1, 7, 1/60)` is between 0 and 1
  - `damp(0, 1, 7, 10)` is greater than 0.999
- [ ] **Step 2:** Run. Expected: FAIL.
- [ ] **Step 3:** `pnpm add three @react-three/fiber @react-three/drei && pnpm add -D @types/three`. Implement `motion.ts`, `geometry.ts` (rounded slabs with `ExtrudeGeometry` from a rounded-rect `Shape`, as in the mockup) and `StackScene`:
  - layers drop from +3.2 units according to `layerDrop`
  - after the opening the scene turns at `SPIN_RAD_PER_S`
  - it tilts toward the pointer with `damp` (lambda 4): y-rotation ±0.3 rad, x-rotation ±0.1 rad
  - with `reducedMotion` there is no spin or tilt and the drop is complete
- [ ] **Step 4:** Run `pnpm test`. Expected: PASS.
- [ ] **Step 5: Checkpoint.** Suggested message: `feat: stack scene and motion maths`.

### Task 7: Fallback decision, frame-rate probe and opening session rule

**Files:**
- Create: `components/stack/capability.ts`, `components/stack/capability.test.ts`

**Interfaces:**
- Produces:
  - `shouldUseStill(env: { reducedMotion: boolean; webgl: boolean; deviceMemory?: number }): boolean`. It is true when `reducedMotion`, when `!webgl`, or when `deviceMemory !== undefined && deviceMemory <= 2`.
  - `createFpsProbe(durationMs = 2000, minFps = 30): { tick(now: number): void; verdict(): "pending" | "ok" | "slow" }`
  - `shouldPlayOpening(storage: Pick<Storage, "getItem" | "setItem"> | null): boolean`. It returns true and stores `"opening" = "1"` the first time in a session, and false afterwards. A throwing or null storage returns true and must not throw.
  - `OPENING_KEY = "opening"`

- [ ] **Step 1: Write the failing tests:**
  - `shouldUseStill({ reducedMotion: false, webgl: true })` is false (undefined memory counts as capable)
  - `shouldUseStill({ reducedMotion: false, webgl: true, deviceMemory: 2 })` is true
  - an fps probe ticked at 16ms for 2 s gives `"ok"`, and at 50ms gives `"slow"`
  - `shouldPlayOpening` on a fake storage gives true, then false
  - `shouldPlayOpening` on a throwing storage gives true without throwing
- [ ] **Step 2:** Run. Expected: FAIL.
- [ ] **Step 3:** Implement.
- [ ] **Step 4:** Run `pnpm test`. Expected: PASS.
- [ ] **Step 5: Checkpoint.** Suggested message: `feat: stack capability checks and opening rule`.

### Task 8: Hero 3D integration, opening and still fallback

**Files:**
- Create: `components/hero/HeroStack.tsx` (client), `components/hero/HeroStackCanvas.tsx` (client, the default export loaded with `next/dynamic`), `components/hero/useOpening.ts`, `e2e/hero.spec.ts`
- Modify: `components/hero/Hero.tsx`, `app/page.tsx`

**Interfaces:**
- Consumes: `StackScene`, `OPENING`, `typeProgress`, `photoProgress`, `shouldUseStill`, `createFpsProbe`, `shouldPlayOpening`, `STACK_PLACEMENT`
- Produces:
  - `<HeroStack />`. It renders the still `<Image src="/stack-still-{theme}.png">` (from Task 9) first, then swaps in the canvas once `shouldUseStill` is false and the dynamic chunk has loaded.
  - The canvas has `role="img"` and `aria-label="The Stack: three layers, interface on top, API in the middle, data at the base."`, `dpr={[1, 2]}`, and `frameloop` set to `"never"` while the hero is off-screen (IntersectionObserver) or `document.hidden`.
  - If the fps probe says `"slow"`, it swaps back to the still.
  - `useOpening(): { playing: boolean; t: number; total: number; skip(): void }`. The total is the mobile or desktop length depending on `matchMedia("(max-width: 800px)")`. It skips on the first `pointerdown`, `keydown`, `wheel` or `touchstart`, and it isn't playing when `shouldPlayOpening` is false or motion is reduced.
  - The hero's type and photo use `typeProgress` and `photoProgress` through inline `transform` and `opacity` driven by `t`, or CSS classes keyed to the same timings. Their resting state (not playing) is fully visible.

- [ ] **Step 1: Write `e2e/hero.spec.ts`:**
  - with `reducedMotion: "reduce"`, no `canvas` exists and an `img` with `src` containing `stack-still` is visible
  - with default settings, a `canvas[role=img]` appears within 5 s
  - when the request for the 3D chunk is aborted (`page.route("**/*HeroStackCanvas*", r => r.abort())`, or match the chunk by content), the `h1`, the photo and the still image are visible
  - on first load, pressing a key ends the opening, so the `h1` has opacity 1 within 300ms
  - after a reload in the same context, there's no opening, so the `h1` has opacity 1 right after load
- [ ] **Step 2:** Run. Expected: FAIL.
- [ ] **Step 3:** Implement. The pointer is normalised to −1..1 over the window. Stop the spin when the tab is hidden. Pass `theme` from `data-theme` through a MutationObserver on `<html>` so the materials follow the toggle.
- [ ] **Step 4:** Run `pnpm test:e2e e2e/hero.spec.ts`. Expected: PASS.
- [ ] **Step 5:** Measure: `pnpm build` and check that the initial JS for `/` doesn't include three (the three chunk loads lazily). Note the sizes in the checkpoint.
- [ ] **Step 6: Checkpoint.** Suggested message: `feat: live hero stack with opening and still fallback`.

### Task 9: Generated images (stills, OG, work screenshots)

**Files:**
- Create:
  - `app/dev/stack-still/page.tsx`: calls `notFound()` when `NODE_ENV === "production"`; renders the Stack fully assembled in a fixed pose with a transparent background, theme from `?theme=`
  - `scripts/capture-images.ts`: Playwright, run with `pnpm tsx`
  - Outputs: `public/stack-still-light.png`, `public/stack-still-dark.png`, `app/opengraph-image.png`, `public/work/aniugnay.jpg`, `public/work/brewscout.jpg`
- Modify: `package.json` (script `images`, plus `tsx` as a dev dependency)

**Interfaces:**
- Produces: the file paths above. Task 8's still and Task 3's Work section use them.

- [ ] **Step 1:** Implement the dev route and the script. The script:
  1. starts from a running `pnpm dev` on port 3100
  2. screenshots `/dev/stack-still?theme=light|dark` at 1200×900 with `omitBackground: true`
  3. screenshots `/` at 1200×630 in the light theme with the opening skipped (sessionStorage `opening=1`) for the OG image
  4. screenshots the two live sites at 1440×900 after `networkidle`, saved as JPEG quality 80
- [ ] **Step 2:** Run `pnpm dev -p 3100 &` then `pnpm images`. Expected: all five files exist and are non-empty. Look at each one.
- [ ] **Step 3:** Add a unit test to `content/projects.test.ts`: each case-study `screenshot.src` exists under `public/` (use `fs.existsSync`).
- [ ] **Step 4:** Check that `/dev/stack-still` returns 404 under `pnpm build && pnpm start` (add it to `e2e/home.spec.ts`).
- [ ] **Step 5: Checkpoint.** Suggested message: `feat: generated stills, OG image and work screenshots`.

### Task 10: Case study pages (MDX)

**Files:**
- Modify: `next.config.ts` (`@next/mdx`, `pageExtensions`)
- Create:
  - `mdx-components.tsx` (required by `@next/mdx`): poster-type `h2`, body text at 18px, `img` mapped to `next/image`
  - `app/work/[slug]/page.tsx`
  - `components/StackMark.tsx`
  - `content/work/aniugnay.mdx`, `content/work/brewscout.mdx`
  - `e2e/work.spec.ts`

**Interfaces:**
- Consumes: `listProjects`, `getProject`, `nextProject`
- Produces:
  - `generateStaticParams()` returns the case-study slugs from `listProjects()`, with `dynamicParams = false`
  - the page loads `await import(\`@/content/work/${slug}.mdx\`)` and calls `notFound()` when `getProject(slug)` is undefined
  - `generateMetadata` sets the title "{name} · Meinard Ga" and the summary as the description
  - `<StackMark layers={Project["layers"]} size?: number />`, an inline SVG of three stacked shapes (plate, plate, disc); used layers in accent, unused in muted; `aria-hidden`
- Each MDX file has four `##` sections: "The problem", "What I built", "The hardest part", "What I'd do next".
  - Draft text comes from the vault notes (`10 Projects/AniUgnay Capstone`, `10 Projects/Brewscout`) and the repos' READMEs.
  - The first line is `{/* DRAFT — Meinard to rewrite */}`.
  - The projects stay `draft: true` in `content/projects.ts`.

- [ ] **Step 1: Write `e2e/work.spec.ts`:**
  - under `pnpm dev`, `/work/aniugnay` shows `h1` "AniUgnay" and the four `h2`s
  - under the production build, `/work/aniugnay` returns 404 while it is a draft, and the home page has no "Read the case study" link to it
  - `/work/does-not-exist` returns 404
- [ ] **Step 2:** Run. Expected: FAIL.
- [ ] **Step 3:** `pnpm add @next/mdx @mdx-js/loader @mdx-js/react @types/mdx`. Implement per the Next MDX guide's "Using dynamic imports" section. The page layout, top to bottom:
  1. poster name with the `StackMark`
  2. a large screenshot
  3. a fact row (role, team, stack, dates, live link)
  4. the MDX body
  5. "Next project"
- [ ] **Step 4:** Run `pnpm test:e2e e2e/work.spec.ts`. Expected: PASS.
- [ ] **Step 5: Checkpoint.** Suggested message: `feat: case study pages`.

### Task 11: SEO, analytics and the final gate

**Files:**
- Create: `app/robots.ts`, `app/sitemap.ts`, `app/sitemap.test.ts`, `e2e/narrow.spec.ts`
- Modify: `app/layout.tsx` (metadata and `<Analytics />`)

**Interfaces:**
- Consumes: `SITE_URL`, `listProjects`
- Produces:
  - `sitemap()` returns `/`, plus `/work/{slug}` for each listed case study
  - `robots()` allows all and points to `${SITE_URL}/sitemap.xml`
  - root `metadata`: `metadataBase: new URL(SITE_URL)`, the title "Meinard Ga · Full-stack developer", a description from the hero lines, and Open Graph plus Twitter (`summary_large_image`)

- [ ] **Step 1: Write the failing test** `app/sitemap.test.ts`: with `NODE_ENV` set to `"production"`, `sitemap()` has no `/work/aniugnay` entry while it is a draft, and every URL starts with `SITE_URL`.
- [ ] **Step 2:** Run. Expected: FAIL.
- [ ] **Step 3:** `pnpm add @vercel/analytics`. Implement.
- [ ] **Step 4: Write `e2e/narrow.spec.ts`** at 320×640 and 390×844, in both themes:
  - `document.documentElement.scrollWidth <= innerWidth`
  - for every `h1`, `h2` and `h3`, the rendered height divided by the line height is at most 2.05
- [ ] **Step 5: Final gate.** Run each of these and fix anything this plan caused:
  - `pnpm lint`
  - `pnpm typecheck`
  - `pnpm test`
  - `pnpm build`
  - `pnpm test:e2e` (all specs)
  - the Taste inspector on the production build at 1440, 768 and 390 in both themes
  - `impeccable detect` on the changed UI files

  Look at every screenshot. Expected: all green; inspector FAILs only for the allowed photo-over-type rule.
- [ ] **Step 6: Reviews.** In a separate subagent (read-only), run `code-reviewer` and `code-review-and-quality` on the full diff. Fix the Critical and Required findings and rerun the affected checks.
- [ ] **Step 7: Checkpoint.** Suggested message: `feat: seo, analytics and final checks`. Suggested branch for Meinard: `feat/portfolio-v1`.

---

## Not in this plan (later, separate work)

- **Blender models replacing the stand-in geometry:** install Blender, run a safety review of Blender MCP, export `.glb`, and swap one layer at a time inside `StackScene`.
- **The final photo and its cut-out, and the final hero Stack placement** (spec §13.1–2).
- **A T201 case study,** if Tito allows screenshots.
- **The domain:** set `NEXT_PUBLIC_SITE_URL` in Vercel once it's claimed.
