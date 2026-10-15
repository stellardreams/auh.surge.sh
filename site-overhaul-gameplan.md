# Site Overhaul Game-Plan — Issue #2

**Target:** <https://auh.surge.sh/>
**Issue:** [#2 — low priority: site overhaul](https://github.com/stellardreams/auh.surge.sh/issues/2)
**Status:** Plan only. No implementation has been performed.
**Baseline captured:** October 15, 2026 — 6:14 AM EDT / 10:14 UTC
**Hosting:** Surge.sh (`server: Surge`), static files, deploy via `surge` CLI on the desktop

---

## 1. Why this plan exists

The site is a single hand-maintained `index.html` (201 lines) plus one stylesheet
(`mystyles.css`, 103 lines) and 13 images totalling **3.4 MB**. It renders as an
unstyled wall of text with a flat cyan background — the "pretty bland" complaint
recorded in issue #1 — and it carries several defects that actively suppress both
search visibility and user trust.

The goal is not a redesign for its own sake. It is: **make the site legible,
credible, accessible, findable, and measurable**, then use the measurement to
decide whether to scale.

---

## 2. Verified baseline findings

Every item below was confirmed by direct inspection of the repository and the
live site on October 15, 2026. Nothing here is speculative.

### 2.1 Critical — search engines are blocked

| Check | Result |
|---|---|
| `https://auh.surge.sh/robots.txt` | `200` — Surge's **default placeholder** |
| Contents | `User-agent: *` / `Disallow: /` |
| Present in repo? | **No** — it is not tracked by git |
| `https://auh.surge.sh/sitemap.xml` | `404` |
| `BingSiteAuth.xml` | **Present and tracked** |

The live site instructs every crawler to index nothing. Meanwhile the repo carries
a `BingSiteAuth.xml` for Bing Webmaster Tools, which cannot ever be used. These
two facts contradict each other. This is the single highest-leverage fix on the
site and it costs one file.

### 2.2 Critical — no page title, description, or social metadata

- `<title>` is `Awake Universal Healthcare - Landing page` — generic, no
  keywords, no value proposition.
- No `<meta name="description">`. Search engines will invent one.
- No canonical URL, no Open Graph tags, no Twitter Card tags. Any share to
  LinkedIn, X, Slack, or iMessage renders as a bare URL.
- No `favicon.ico` (live: `404`), no `site.webmanifest` (live: `404`). The tab
  and the home-screen install show a default glyph.
- No `404.html` (live: `404`).
- No `README.md`, no `LICENSE`, no `CONTRIBUTING.md`. The repo has
  `.github/FUNDING.yml` and nothing else — the single most-visited page for a
  prospective contributor or investor does not exist.

### 2.3 High — accessibility failures

| Issue | Location | Impact |
|---|---|---|
| No `<h1>` | `index.html:28` starts at `<h2> Mission` | Screen-reader users get no document title anchor; heading hierarchy is broken |
| No `<main>`, `<nav>`, `<header>`, `<footer>` landmarks | whole file | No structural navigation for assistive tech |
| No skip-to-content link | whole file | Keyboard users tab through everything, every load |
| No visible `:focus` styling | `mystyles.css` | Browser default focus ring is the only affordance |
| Fixed heights | `mystyles.css:39` (`height: 410px`), inline `height:450px` on all 8 team images | Long role descriptions are **clipped or overflow**; text can be cut off with no scroll |
| `float: left` layout | `mystyles.css:17` | Source order ≠ visual order; breaks at zoom and on small screens |
| Justified text in narrow columns | `mystyles.css:40,98,102` | Rivers and word-gaps; WCAG 1.4.4 readability concern in 30%-wide cards |
| Heading text is a placeholder | `index.html:96,112,119,133,146,152,167` | Seven headings read **"Your name here"** or similar — signals an unfinished site to every visitor |
| Alt text includes the placeholder | `index.html:91,116,140,153` | `alt="Lead ethicist - Your name here"` |
| `<br>` inside `<ul>` | `index.html:65` | Invalid list content |
| Stray closing `</ul>` | `index.html:55` | Malformed nesting |
| `<h2>` used as card titles | `index.html:78,94,…` | Heading levels do not reflect document structure |

### 2.4 High — broken and unsafe dependencies

| Item | Finding |
|---|---|
| `js/bootstrap.min.js` | `index.html:197` references a file that **does not exist** in the repo. Live: **404**. A render-blocking request fails on every page load. |
| jQuery `1.11.2` | `index.html:196`, loaded from a CDN. **Never used** — no jQuery call exists in the file. Version is from 2013 and carries the known XSS issues CVE-2020-11022 / CVE-2020-11023. |
| Inline image styles | `style="width:100%; height:450px; object-fit: cover;"` repeated 8× on `index.html` | Duplicated, unmaintainable, and the `object-fit: cover` crop is what distorts faces |
| `.my-icon`, `.button` | `mystyles.css:72,54` | Dead CSS — no matching element in `index.html` |
| Mixed line endings | Live serves **CRLF** (12,493 B); repo holds **LF** (12,297 B) | Cosmetic, but makes every diff noisier than it needs to be |

### 2.5 Medium — performance

| Metric | Value |
|---|---|
| `index.html` | 12.5 KB (fine) |
| `Sesame_Street.jpeg` | **2,428,962 B — 2.4 MB, a single hero image** |
| `blockchain.png` | 196 KB |
| `heart-ai.png` | 185 KB |
| `AK-04-29-2021.jpeg` | 124 KB |
| **Total `img/` payload** | **3.4 MB** |
| Unused tracked images | `AK.jpg`, `RG.jpg`, `astrobiology.jpeg`, `gender-neutral-profile-icon.png`, `heart_tiny.png` |

One 2.4 MB JPEG with no `width`/`height` attributes and no `loading="lazy"`
guarantees both a slow first paint and a layout shift.

### 2.6 Medium — visual and content quality

- `background-color: lightcyan` (`mystyles.css:2`) with **no design system** — no
  type scale, no spacing scale, no colour tokens, no dark mode.
- Default browser serif throughout. No custom font, no `font-family` at all.
- Imagery language is incoherent: one portrait photo (`AK`), one candid photo
  (`Sesame_Street`), four flat clip-art icons (`ethics`, `GP`, `comms`,
  `pegasus-tribal`), one stock photo (`heart-ai`), one stock render
  (`blockchain`). No shared style.
- The "Values" list contains two GitHub sponsor `<iframe>`s (`index.html:66,68`)
  embedded **inside the list**, splitting a content list with a payment widget.
- No header/hero, no navigation, no footer, no copyright line, no contact path
  anywhere on the page.
- No call to action of any kind. The site states a mission, then ends.
- `http://asi.surge.sh/` (`index.html:49`) is plain HTTP — should be HTTPS.
- Two identical `<!-- Global site tag -->` comments (`index.html:10,11`).
- `<META>`/`<HTML>`/`<BODY>` uppercase tags (`index.html:2,3,4,25,27`) — valid
  HTML5 but inconsistent.

### 2.7 Medium — analytics are installed but unused

- Google Analytics 4 (`G-7SJ4S5BMBT`) and Microsoft Clarity (`pkp51w1lxx`) are
  both live in `<head>`. The Clarity tag dates from December 27, 2024.
- **No custom events, no conversions, no goals.** Nothing is being measured
  beyond pageviews.
- No consent banner. GA4 and Clarity both set cookies before any consent is
  requested — an EU/UK visitor has no way to refuse.
- **No baseline is recorded anywhere.** Issue #1's "9 unique users during the
  last year" is the only traffic figure in existence, and it is 10 months stale.
  No overhaul can be judged without a recorded "before".

### 2.8 Low — infrastructure

- Repo was on `master` with no `main-dev`, blocking `git-branching-workflow`.
  **Resolved October 15, 2026** — see §4.
- No CI. `.github/` contains `FUNDING.yml` only.
- No deploy automation; Surge is deployed manually from the desktop with the
  `surge` CLI (which is **not installed on the current machine** — `surge:
  command not found`).

---

## 3. Goals and success criteria

### 3.1 Goals

1. **Be findable.** Pass technical SEO audit; be indexable; render a rich
   preview on every social platform.
2. **Be credible.** Look like a healthcare organisation that takes patient
   privacy seriously — because it claims to.
3. **Be accessible.** Meet WCAG 2.2 Level AA.
4. **Be fast.** Sub-second first paint on a mid-range phone.
5. **Be measurable.** A recorded traffic baseline and a defined set of
   conversion events, live before any redesign ships.
6. **Be maintainable.** No dead dependencies, no dead CSS, no duplicated inline
   styles.

### 3.2 Quantitative success criteria

- Lighthouse: **Performance ≥ 95, Accessibility ≥ 100, SEO = 100, Best
  Practices ≥ 95** (mobile, production).
- axe DevTools: **zero critical or serious violations** on the live page.
- Total page weight **under 600 KB** (from 3.4 MB).
- Zero requests to non-existent assets.
- Every image has explicit `width`/`height`; cumulative layout shift (CLS)
  **< 0.1**.
- Valid HTML — zero errors from `html-validate`.

### 3.3 Baseline capture — do this FIRST

- [ ] **A. Record the analytics "before" before touching any code.** Export GA4
      and Clarity data for the 12 months to date — users, sessions, pages/session,
      engagement time, top entry pages, geography, device split — and attach it
      to issue #2. Without this, the overhaul is unfalsifiable.
      *(User action item — Adeel Khan (@genidma); requires GA4 and Clarity
      console access.)*
- [ ] **B. Record a Lighthouse baseline** against production and save the JSON
      alongside this plan. *(User action item — Adeel Khan (@genidma); requires
      `npx lighthouse https://auh.surge.sh/`.)*

---

## 4. Completed groundwork — October 15, 2026

- [x] **C. Branch restructure.** `master` renamed to `main`; `main` set as the
      GitHub default branch; remote `master` deleted; `main-dev` created and
      pushed. Unblocks `git-branching-workflow` for all future work. Verified via
      `git branch -vv` and `gh repo view --json defaultBranchRef`. This was a
      refs-only operation — it created no new commit, so there is no commit SHA
      to cite.

---

## 5. Phase plan

Phases are ordered by dependency, then by value-per-hour. Each phase is a
self-contained commit series. **Stop after each phase and test on `main-dev`
before merging to `main`.**

> **Standing constraint:** Surge is deployed by hand from the desktop using the
> `surge` CLI, which is not present on every machine. No phase may be considered
> done until it has been deployed and visually verified on a real browser at a
> real phone width.

---

### Phase 1 — Unblock discovery and remove the breakage
*Highest value per hour. No visual change. Safe to ship in one commit.*

- [ ] **1.1 Add `robots.txt` allowing indexing.**
      ```
      User-agent: *
      Allow: /
      Sitemap: https://auh.surge.sh/sitemap.xml
      ```
      Overrides the Surge placeholder that is currently blocking all crawlers.
- [ ] **1.2 Add `sitemap.xml`** listing `https://auh.surge.sh/` with a current
      `<lastmod>`.
- [ ] **1.3 Delete the `js/bootstrap.min.js` script tag** (`index.html:197`)
      and the unused jQuery `1.11.2` tag (`index.html:196`). Removes a
      guaranteed 404 and a 2013-era library with known XSS CVEs.
- [ ] **1.4 Retitle and describe.** `<title>` becomes
      `Awake Universal Healthcare — Global, Private, Patient-First Care`; add a
      ~155-character `<meta name="description">` drawn from the Mission text.
- [ ] **1.5 Add Open Graph and Twitter Card tags**, including an `og:image`
      (the 124 KB `AK-04-29-2021.jpeg` is a reasonable first candidate, or a new
      1200×630 asset).
- [ ] **1.6 Add a favicon.** Either a real `favicon.ico` or an SVG favicon plus
      `<link rel="icon">` and an Apple touch icon.
- [ ] **1.7 Upgrade `http://asi.surge.sh/` to `https://` (`index.html:49`).**
- [ ] **1.8 Remove the duplicate GA comment** (`index.html:10,11`) and normalise
      tag casing to lowercase.
- [ ] **1.9 Verify:** `curl -sI` on `robots.txt`, `sitemap.xml`, `favicon.ico`
      all return `200`; the browser network tab shows zero `404`s; Google's
      Rich Results Test and the LinkedIn Post Inspector both render the preview.

---

### Phase 2 — Structure and semantics
*No visual change. Makes Phase 3 safe to build on.*

- [ ] **2.1 Introduce `<header>`, `<nav>`, `<main>`, `<footer>` landmarks.**
- [ ] **2.2 Fix the heading hierarchy.** Promote the page title to `<h1>`;
      demote or restructure card headings so levels nest correctly.
- [ ] **2.3 Add a skip-to-content link** as the first focusable element.
- [ ] **2.4 Repair the malformed lists** — the stray `</ul>` (`index.html:55`)
      and the `<br>` inside `<ul>` (`index.html:65`). Move both sponsor
      `<iframe>`s out of the Values list into a proper Funding section.
- [ ] **2.5 Remove dead CSS** — `.my-icon` and `.button` in `mystyles.css`.
- [ ] **2.6 Convert every inline `style` attribute on images to a class.**
- [ ] **2.7 Normalise line endings** to LF across the repo; add `.gitattributes`
      so the CRLF/LF drift cannot return.
- [ ] **2.8 Verify:** `npx html-validate index.html` reports zero errors; heading
      order is visually sane; keyboard Tab from page load reaches the skip link
      first.

---

### Phase 3 — Design system and layout
*The phase that answers "the site is pretty bland."*

- [ ] **3.1 Define design tokens as CSS custom properties** in `:root` —
      colour, type scale (a modular scale, e.g. 1.25 ratio), spacing scale
      (4 px base), radii, shadows.
- [ ] **3.2 Pick a real palette.** Replace `lightcyan` with an intentional
      clinical palette that still meets **WCAG AA (4.5:1)** for body text.
      Recommended direction: a calm off-white surface, a deep desaturated
      teal or navy primary, one warm accent reserved for calls to action.
- [ ] **3.3 Set a type scale** with a web-safe stack or a single self-hosted
      variable font. Retire the default serif. Respect `prefers-reduced-motion`.
- [ ] **3.4 Replace the `float` grid** with CSS Grid (`grid-template-columns`
      with `repeat(auto-fit, minmax(280px, 1fr))`). This removes the 650 px
      breakpoint hack, reflows correctly at 200 % zoom, and keeps DOM order equal
      to visual order.
- [ ] **3.5 Delete the fixed `height: 410px` on `.container`** and the fixed
      `height: 450px` on images. Let cards size to their content; constrain
      images with `aspect-ratio` instead so nothing clips.
- [ ] **3.6 Stop justified text inside cards.** Keep `text-align: justify` only
      for full-width prose, if at all.
- [ ] **3.7 Add a proper header/hero** with the mission statement, a one-line
      value proposition, and **a primary call to action**.
- [ ] **3.8 Add navigation** to Mission / Focus Areas / Team / Partner / Contact.
- [ ] **3.9 Add a footer** with copyright, year, contact address, and links to
      the sister companies.
- [ ] **3.10 Add visible `:focus-visible` styling** and a dark-mode block under
      `prefers-color-scheme: dark`.
- [ ] **3.11 Verify:** Lighthouse Accessibility ≥ 100 and Performance ≥ 95 on
      mobile; visual pass at 320 px, 768 px, 1280 px, and 1920 px; keyboard-only
      walkthrough of the whole page.

---

### Phase 4 — Imagery
*Removes 3.4 MB of payload and fixes the incoherent look.*

- [ ] **4.1 Decide one imagery language** — either all photography or all
      illustration. Do not mix a candid photo with flat clip-art.
- [ ] **4.2 Compress and resize every image.** Convert `Sesame_Street.jpeg` from
      2.4 MB to a sub-100 KB WebP or AVIF at the display size actually used.
- [ ] **4.3 Replace the seven "Your name here" headings** with either a real
      name or an honest open-role label — e.g. `Open role — Lead Ethicist`. Never
      ship a placeholder to production.
- [ ] **4.4 Write descriptive alt text** that describes the role, not the stock
      image. Decorative images take `alt=""`.
- [ ] **4.5 Serve responsive `srcset`** so phones do not download desktop-sized
      images.
- [ ] **4.6 Add `width`, `height`, and `loading="lazy"`** to every non-hero
      image. The hero image gets `loading="eager"` plus `fetchpriority="high"`.
- [ ] **4.7 Audit the 5 unused tracked images** and delete the ones with no
      home, to cut repo weight.
- [ ] **4.8 Verify:** total page weight under 600 KB; CLS < 0.1 in Lighthouse;
      every card image is visually correct at 320 px and 768 px.

---

### Phase 5 — Content restructure
*The site currently reads as a list of hopes. Make it read as an organisation.*

- [ ] **5.1 Group the eleven "Primary areas of interest"** bullets into four or
      five named pillars — for example *Frontline Wellbeing*, *Mental Health at
      Scale*, *Longevity & Rare Disease*, *One Health*, *Systems & Reform*. The
      current flat list buries the strongest ideas.
- [ ] **5.2 Trim the grocery-store-clerk paragraph.** It is warm and personal,
      but it is 40 % of the first bullet and it buries the veteran and
      first-responder mission. Keep the voice, move the story to a dedicated
      "Why" section, or cut it.
- [ ] **5.3 Complete the truncated bullet** — "Leveraging robotics for geriatric
      care, limited or no mobility (including but not limited to)" ends without
      finishing its thought.
- [ ] **5.4 Rewrite the eight role descriptions** to a consistent
      length-and-structure template: title, one-line purpose, 3–5 bullets of
      responsibilities, requirements, and time commitment.
- [ ] **5.5 Replace the Team grid's clip-art with real portraits** or move to a
      consistent icon set. *(User action item — Adeel Khan (@genidma) may hold the
      real portraits; AI-generated faces must be labelled as such.)*
- [ ] **5.6 Add a Contact section** — the site currently has no way to reach
      anyone. *(User action item — Adeel Khan (@genidma): supply the address.)*
- [ ] **5.7 Proofread.** `index.html:181` reads "ideas > heirarchy" — typo for
      "hierarchy". Check for others in the same pass.
- [ ] **5.8 Verify:** every claim on the page is either supported or clearly
      framed as intent; no unfinished sentence or placeholder survives.

---

### Phase 6 — Measurement
*Must be live and recording before the spring work, so the spring work can be judged.*

- [ ] **6.1 Define the conversion events** worth tracking. Recommended set:
      `view_mission`, `view_team`, `click_partner_link`, `click_github`, and
      `sponsor_click`.
- [ ] **6.2 Implement GA4 custom events** on the corresponding elements.
- [ ] **6.3 Add Clarity custom tags** mirroring the GA4 events so the two tools
      tell the same story.
- [ ] **6.4 Add a consent banner** and gate both GA4 and Clarity behind it.
      Currently neither script can be refused.
- [ ] **6.5 Document the measurement plan** — which numbers are reviewed, how
      often, and what would count as success — and file it here.
- [ ] **6.6 Re-run the analytics export after 30 and 90 days** and post both
      comparisons to issue #2. *(User action item — Adeel Khan (@genidma).)*

---

### Phase 7 — Repository and deployment hygiene
*Low urgency, but cheap and it prevents future confusion.*

- [ ] **7.1 Add `README.md`** — what AUH is, who it serves, how to build and
      deploy, who to contact.
- [ ] **7.2 Add `LICENSE`.** *(User action item — Adeel Khan (@genidma): select
      the license; do not assume.)*
- [ ] **7.3 Add `CONTRIBUTING.md`**, especially given issue #1 carries a
      `help wanted` label.
- [ ] **7.4 Add `.github/workflows/`** — HTML/CSS validation on push, so a
      malformed tag never reaches production again.
- [ ] **7.5 Document the Surge deploy runbook** — domain, publish path, and the
      fact that `surge` must be installed locally.
- [ ] **7.6 Add a `404.html`** styled to match.
- [ ] **7.7 Add `humans.txt` and a security disclosure contact.**

---

## 6. Sequencing and dependencies

```
Phase 1 (SEO + breakage) ──┐
Phase 2 (structure) ───────┼──> Phase 3 (design) ──> Phase 4 (imagery)
Phase 6 (measurement) ─────┘                             │
        │                                                v
        └────────────> baseline recorded ────────> Phase 5 (content)
                                   │
                                   v
                        Phase 7 (repo hygiene, any time)
```

- **Phase 6 must start before Phase 3 ships**, otherwise the redesign is
  unmeasurable. Baseline capture (§3.3) gates everything.
- Phases 1 and 2 are prerequisites for Phase 3 — designing on top of broken
  semantics wastes the work.
- Phases 3, 4, and 5 are independent of each other and can be interleaved, but
  each should merge and deploy on its own.
- Phase 7 has no dependencies and can be done opportunistically.

**Suggested sizing:** Phases 1 + 2 + 7.1 together are roughly an afternoon and
deliver most of the discoverability win. Phases 3 + 4 + 5 are the real spring
project.

---

## 7. Risks

| Risk | Likelihood | Mitigation |
|---|---|---|
| Scope creep — "site overhaul" is unbounded | High | Execute phases sequentially; merge and deploy after each; do not start Phase 4 before Phase 3 is verified |
| Redesign ships without a baseline and cannot be judged | Medium | §3.3 is a hard gate — no Phase 3 work before the "before" numbers are recorded |
| Third-party embeds (GitHub sponsor iframes) break or get blocked | Medium | Test the sponsor path before relying on it for donations; consider a plain link with a GitHub Sponsors badge |
| Healthcare copy drifts into clinical claims | Medium | Every medical claim gets reviewed before it goes live; frame intent as intent |
| `robots.txt` change triggers a crawl wave mid-redesign | Low | Ship Phase 1 on its own, a few days before the visual work, so any ranking movement is attributable |
| Phone-width regressions | Medium | Fixed 450 px image heights and `float` are the known culprits; clear both in Phase 3 and test at 320 px |
| Content changes blocked on user-supplied assets | High | Mark portraits, contact address, and license as *(user action items)* and do the rest in parallel |

---

## 8. Explicitly out of scope

- JavaScript frameworks, build tooling, or a static site generator. The site is
  12.5 KB of HTML; it does not need a bundler.
- A CMS. Content changes at the pace of roughly one page per quarter.
- Server-side anything. Surge is static hosting.
- Copy that AUH's own leadership has not approved.
- The sister-company sites (`asi.surge.sh`, `shangrila-vhp`) — tracked separately.

---

## 9. Open questions for Adeel Khan (@genidma)

1. **Brand direction** — is AUH's identity clinical-trust (teal/navy, calm) or
   activist-warm? Phase 3.2 depends on the answer.
2. **Imagery** — do real portraits of the seven open roles exist, or should those
   cards use a consistent icon treatment until people are hired?
3. **Contact path** — what address should the Contact section point to?
4. **Analytics access** — can the GA4 and Clarity "before" export be produced?
   Without it there is no baseline.
5. **Trafficking ambition** — is the goal more visitors, or better-converting
   visitors? The two point to very different designs of the hero section.
6. **Scope of the spring window** — how many hours are realistically available?
   That determines whether Phase 4 (imagery) is in or out.

---

> 🤖 **Signed:** opencode, powered by **space-bunny-free** — Site Overhaul Game-Plan for issue #2 (plan only; no implementation performed)
> 📅 **Date/Time:** October 15, 2026 — 6:14 AM Eastern (ET) / 10:14 UTC