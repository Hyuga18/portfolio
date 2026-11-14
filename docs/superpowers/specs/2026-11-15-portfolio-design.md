# Portfolio — design

Date: 2026-11-15 · Owner: Meinard Ga · Status: draft for approval

Sources: vault note `10 Projects/Portfolio` (decisions from 2026-11-14 and 2026-11-15), CV `~/Documents/PDF/Meinard_Ga_Resume.pdf` (2026-11-11), look mockups https://claude.ai/artifact/LKJb2TDBzWQRyg8LgvkbJ9 (direction C) and hero mockup https://claude.ai/artifact/MCaZo8VqzwXu2vZWGkRjUk.

## 1. Intent

A personal portfolio site that helps Meinard land his first full-time role. He graduates in March 2027 (BSIT, Adventist University of the Philippines) and is available full-time from April 2027. It is for everyone, but recruiters come first: in about 30 seconds a recruiter should know who he is, what he does, when he is available, see real work, and find a way to contact him or download the CV.

The site positions him as a **full-stack developer (React/Next.js + NestJS/Postgres) whose frontend craft sets him apart.** The site itself is part of the proof of that craft.

There is no deadline. It gets built properly.

## 2. Goals, non-goals, anti-goals

**Goals**
- First screen answers who, what and when: name, "Full-stack developer", "Available full-time from April 2027", and a way to see the work.
- Real work is visible without clicking: large screenshots of AniUgnay and Brewscout on the home page.
- One memorable moment: the 3D "Stack" opening in the hero, which stays alive and responds to the cursor.
- Fast, accessible, searchable: all content is real HTML; the 3D never blocks reading.
- Easy to extend: a new case study is one MDX file.

**Non-goals**
- No blog, no contact form, no booking, no CMS, no i18n (English only), no sound.
- No 3D outside the hero. The Stack does not travel through the page and there is no scroll story (dropped 2026-11-15).
- No screenshots or code of Tito Solutions products until Tito says yes.

**Anti-goals (a failure even if it works)**
- A joke or gimmick site. The tone is serious and polished.
- 3D that delays the text, drains a phone, or makes a recruiter wait before they can read.
- A design that copies oryzo.ai (Lusion) or the "portrait in front of big type" reference Meinard sent. The concepts may be used, not their look.
- Showing the phone number on the site (it stays in the PDF CV only).

## 3. Constraints

- **Stack:** Next.js 16.4 (App Router), React 19.3, TypeScript, Tailwind CSS 4, pnpm. Already scaffolded at `~/Documents/Programing Files/portfolio`. This Next.js version has breaking changes; read `node_modules/next/dist/docs/` before writing code (see `AGENTS.md`).
- **3D:** React Three Fiber + drei (to be added).
- **Hosting:** Vercel, with Vercel Web Analytics (no cookies).
- **Repo:** public, `github.com/Hyuga18/portfolio`. Meinard handles all git (commits, branches, pushes, PRs).
- **Domain:** `meinard.dev` via the GitHub Student Pack Name.com offer; `meinardga.dev` as the backup. Not claimed yet.
- **Content source of truth for dates, titles, links and numbers:** the CV PDF. Numbers from the CV may be used (they are already public).
- **Fonts:** Schibsted Grotesk via `next/font/google` (self-hosted at build time).

## 4. Approaches considered (for the 3D)

1. **R3F canvas inside the hero only (chosen).** The canvas lives in the hero section, loads client-side after the text, plays the opening, then keeps a slow turn and a cursor tilt. It stops rendering when the hero is off-screen or the tab is hidden.
2. **Pre-rendered video for the opening, then a still image.** Lightest, but nothing responds to the visitor, it needs one video per theme, and every model change means rendering again. Rejected: Meinard wants the Stack alive after the opening.
3. **One fixed full-page canvas with section anchors** (the earlier scroll-story plan; the look mockups used it). Rejected when the scroll story was dropped: it only paid off if the Stack travelled.

Plain three.js without R3F was also rejected: React would then need its own glue for lifecycle and resizing, which R3F already handles.

## 5. Look: direction C, "Poster"

Chosen 2026-11-15 from three dealt directions.

**Type:** Schibsted Grotesk only.
- Display: weight 800, tracking about -0.05em, leading about 0.86.
- Text: 400/600 at 16–18px, leading 1.5–1.55.
- No more than five sizes on a page.

**Colour tokens (mockup values; tune for contrast during the build):**

| Token | Light | Dark |
|---|---|---|
| ground `--bg` | `#dedad3` | `#0d0e0e` |
| surface `--surface` | `#d1ccc3` | `#171919` |
| ink `--ink` | `#0a0a0a` | `#e7e3dc` |
| muted `--muted` | `#4d4943` | `#a8a39b` |
| accent `--accent` (teal) | `#1f6576` | `#5fc6dc` |
| accent ink `--accent-ink` | `#ffffff` | `#061316` |

**Theme:** follows the system by default, with a light/dark toggle in the nav. The choice is remembered (localStorage), and there is no flash of the wrong theme on load. Theme changes ease rather than snap.

**House rules (from the Taste skill; hold everywhere):**
- No eyebrow labels, no 01/02/03 numbering.
- No gradients, glass panels, hairline dividers or icon cards.
- No shadows on things at rest.
- No monospace outside code.
- No tiny uppercase tracked labels.
- Buttons are flat teal with square corners.
- Every headline fits in two lines at every width.
- Equal outer margins at every width.
- Surfaces are separated by tone steps, never by lines.

## 6. The Stack (3D)

**What it is:** a sculptural object of three layers that says "full-stack" without words.
- Interface (top): a thin square plate with raised panes.
- API (middle): a thicker plate with four ports and a ring.
- Data (bottom): three stacked discs.

**Material:**
- Light mode: frosted glass/ceramic with soft shadows.
- Dark mode: dark body with teal glowing edges and a soft teal glow.

**Models:** code-built stand-ins ship first (as in the mockups). Blender models replace them one at a time later as `.glb` files. Step 1 of the Blender work: Meinard installs Blender, then a safety review of Blender MCP before connecting it. If MCP fails review, the fallback is headless Blender Python scripts that export `.glb`.

**Opening (about 2 s):**
1. The three layers drop in one after another (data, API, interface) and assemble.
2. Then the hero photo rises into place.
3. Then "Meinard" and "Ga" slide in from their corners.

It plays on the first visit of a browser session only (sessionStorage). Any click, scroll, key or touch skips straight to the finished hero. The text is in the HTML from the start and stays readable throughout; the opening only animates it into place.

**After the opening:**
- A slow turn (one revolution every 90 s).
- A gentle tilt toward the cursor (critically damped, no bounce).
- No other looping motion.

**Performance:**
- The canvas module is dynamically imported client-side; the hero text and photo render first.
- Pixel ratio is capped at 2.
- Rendering pauses when the hero is out of view (IntersectionObserver) and when the tab is hidden.

**Fallback — a still image of the assembled Stack, per theme, instead of the canvas, when any of these hold:**
- `prefers-reduced-motion: reduce` is on.
- There is no WebGL.
- `navigator.deviceMemory` is 2 GB or less.
- The frame rate stays under about 30 fps for the first 2 s.

The fallback also skips the opening and has no cursor tilt. The canvas has `role="img"` and an `aria-label`.

**Case study pages:** no live 3D. A small static Stack mark (SVG or image) shows which layers the project touched. Both current projects touched all three.

## 7. Hero

Composition (desktop):
- "Meinard" set huge in the top-left, around 15vw.
- "Ga" huge in the bottom-right.
- Meinard's photo, cut out of its background, standing right of centre in front of the type and overlapping the lower part of "Meinard".
- The Stack behind or near the photo (exact placement open, see §12).
- Bottom-left: "Full-stack developer · Available full-time from April 2027", one line on the stack ("React and Next.js in front, NestJS and Postgres behind."), a teal "See selected work" button and a "Download CV" link.
- Nav links top-right: Work, Experience, About, Contact, plus the theme toggle.

The page's `h1` is the name "Meinard Ga" (the two halves are spans of one heading).

**Photo:**
- Source: `~/Downloads/IMG_3262.JPG` (dark suit, looking to the viewer's left, so he stands right of centre and looks into the page).
- It needs a clean cut-out with a proper mask around the hair and arms (Photoroom or Photoshop). The mockup's cut-out was made locally with macOS Vision and is only a stand-in.
- A new photo shot for this would be better: plain wall, one side or back light, full resolution, framed from mid-thigh up. Not required to start.
- In dark mode a soft teal rim light on the hair and shoulders separates the black suit from the dark ground.
- Ship the cut-out as AVIF/WebP with alpha via `next/image`, with descriptive alt text.

**Phone:** "Meinard" on top, the photo with the Stack below it, "Ga" right-aligned under the photo, then the intro text and actions. The opening plays in a lighter form.

## 8. Sections below the hero (plain HTML, no 3D)

In order. Each section has its own heading, and neighbouring sections use different layouts.

1. **Selected work.**
   - One full-screen panel per project: a large screenshot (about 60% of the screen), the project name in poster type, one line on what it is plus role and stack, "Read the case study" and "Open the live site".
   - AniUgnay first: lead developer and team lead of a four-student capstone; a farm-to-buyer marketplace for Cavite; Next.js, React, TypeScript, Supabase, Tailwind; https://aniugnay.vercel.app.
   - Brewscout second: an installable web app for independent cafés in Cavite, with 50 hand-verified cafés; Next.js, Supabase (PostGIS), MapLibre GL; https://brewscout-hyuga.vercel.app (see §12.3).
   - Then a smaller third entry for CSC L.I.V.E. (Supabase-backed sports scoreboard for AUP; RLS audit closed anonymous writes on five tables), with no panel.
   - Screenshots are real captures of the live sites.
2. **Experience.**
   - Software Developer Intern, Tito Solutions, August 2026 – present. Words only: no screenshots, no code links.
   - The CV numbers set large on one row: 290+ merged PRs; 100+ employees use the products; a 4-second backlog freeze removed; about 50× fewer database calls on the CSV import.
   - One short paragraph each for Ticketo, T201 HRIS and Tito Awards, from the CV bullets.
3. **About and skills.**
   - A portrait (4:5) of Meinard beside a few sentences in his own voice. This can be a different crop or photo from the hero.
   - Skills grouped like the CV, as plain text groups (no badge cloud): languages; frontend; backend and databases; testing and tools; AI-assisted development.
   - Certifications as one line.
4. **Education.** BSIT, AUP, graduating March 2027, GPA 3.50/4.00. Leadership: Head Technical Director, Central Student Council (2025–2026); Technical Director, Silang Memoir yearbook (2025–2026). A compact band.
5. **Contact.**
   - The email (`jobga1234@gmail.com`) set as display type, as a `mailto:` link with a "Copy email" button and visible feedback ("Copied").
   - Then "Download CV" (the PDF served from `/public`), LinkedIn (`linkedin.com/in/meinard-ga-5629a3387`) and GitHub (`github.com/Hyuga18`).
   - No phone number.
6. **Footer.** "Meinard Ga" again at full width, the year, and the credit lines if any found images are used.

The leftover IT Support Ticketing System (Spring Boot) is left out.

**Motion below the hero:**
- Each section's heading, then its text, then its picture rises 16–32px and fades in once as it enters, staggered 60–90ms. Content is never hidden if JavaScript fails.
- Buttons press down slightly; screenshots scale in slightly as their panel arrives.
- Everything uses transform and opacity only, and everything respects reduced motion.

## 9. Case study pages

Route `/work/[slug]`, one MDX file per project (AniUgnay, Brewscout). Each page has:
- The project name in poster type with the small static Stack mark.
- A large screenshot.
- A fact row: role, team, stack, dates, live link, repo link if public.
- Written sections: the problem; what I built; the hardest part and how I solved it; what I'd do next.
- More screenshots between the sections.
- A "Next project" link at the bottom.

Meinard writes or approves the case study text. Drafts may be prepared from the vault notes and the repos, marked as drafts.

Adding T201 later (if Tito allows screenshots) means adding one MDX file.

## 10. Site-wide

- **Metadata:** title, description, Open Graph and Twitter images (an image of the hero), `robots.ts`, `sitemap.ts`, canonical URL on the final domain.
- **Accessibility:**
  - Visible focus styles in the accent colour, and keyboard access to everything including the theme toggle and copy button.
  - Contrast AA in both themes.
  - Alt text on every image; reduced-motion respected everywhere.
  - Themed selection colour and scrollbar.
- **Performance targets:** text visible without waiting for 3D; LCP under 2.5 s on a mid phone; CLS under 0.1 (space reserved for the photo and canvas).
- **Analytics:** Vercel Web Analytics only.

## 11. Decisions left to the builder

- File and component structure, how MDX is wired (`@next/mdx` or similar), and how the theme is stored and applied without a flash.
- How the stand-in Stack geometry is coded; how the frame-rate check is done.
- Exact easing values within the rules above (critically damped springs; one turn per 90 s; opening about 2 s).
- Exact type sizes within the five-size limit.
- How screenshots are captured and optimised.

## 12. Calls made after the builder check (Claude's calls; Meinard may override)

1. **Hero Stack placement for now:** ship one layout, the Stack as a large backdrop behind the photo, partly cropped by the top edge. Its position, scale and z-order live in one set of constants so the other option (beside him at head height) is a small change. Layer order, back to front: ground, the "Meinard"/"Ga" type, the canvas, the photo, the intro text and nav. The canvas box and the photo have fixed aspect ratios so nothing shifts on load.
2. **Missing assets ship as stand-ins, clearly replaceable:**
   - The photo is the stand-in cut-out from `IMG_3262.JPG`.
   - The fallback stills (one per theme) and the OG image are rendered from the stand-in Stack by a small script in the repo (headless browser screenshot of the canvas), so they can be regenerated when the Blender models land.
   - The AniUgnay and Brewscout screenshots are captured with a headless browser against the live URLs, by a script that can be rerun.
3. **Brewscout URL:** `https://brewscout-hyuga.vercel.app` (checked 2026-11-15: it serves "Brewscout · Cavite Coffee Gazetteer", the current product, and matches the CV). `brewscout.vercel.app` also responds but serves an older "BrewScout" title. Keep project URLs in one data file.
4. **CV and case study text:**
   - The CV PDF is copied into `public/` as `Meinard_Ga_Resume.pdf`. It is public by design, and it is the only place the phone number appears.
   - Case study MDX may be drafted from the vault and repos with `draft: true` in the front matter. Drafts render in development only and are excluded from production builds, the sitemap and the work list until Meinard approves them.
5. **Opening on phones and tablets:** the same sequence, shortened to about 1.2 s. `navigator.deviceMemory` is treated as capable when undefined (Safari, Firefox); the frame-rate check still applies there.
6. **Dark-mode glow:** emissive materials, edge lines and an additive glow sprite, as in the mockups. No postprocessing bloom, to keep the bundle and GPU cost down.
7. **Site URL:** canonical URLs, the sitemap and Open Graph read the site URL from one env var (`NEXT_PUBLIC_SITE_URL`), with the Vercel URL until the domain is claimed.

## 13. Open items (not blocking the plan)

1. **Final hero placement of the Stack** (see §12.1). Meinard deferred it; revisit with the real photo.
2. **Final photo and cut-out** (see §7).
3. **Tito Solutions screenshots:** Meinard asks Tito. If yes, T201 gets a case study.
4. **Domain:** claim `meinard.dev` (or `meinardga.dev`) and check the renewal price.
5. **Case study text** from Meinard.
