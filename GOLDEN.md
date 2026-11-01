# 🚀 Golden Rules for Frontend Developers

What to keep in mind when building a client's project so their business grows and they stay satisfied.

**How to use this:** each rule has a *Check* line — something you can actually run or look at in five
minutes. A rule you can't check is a slogan.

---

## Part 1 — Business & Product

1️⃣ **Understand the client's business goals.**
Your job is not just to build UI, but to ensure that UI helps achieve business objectives (leads, sales, bookings).
*Check:* write down the one action you want a visitor to take. If you can't name it, the page has no job.

2️⃣ **User-friendly design.**
Simple navigation, clear buttons, readable text. Everything easy and intuitive for the end-user.
*Check:* a stranger can complete the main task without being told where to click.

3️⃣ **Mobile-first.**
Most traffic is mobile. The design must work flawlessly on all screen sizes.
*Check:* test at 360×640, not just in a resized desktop window. Use real safe-area insets on notched devices.

4️⃣ **Clear calls-to-action.**
"Book Now", "Get Quote", "Contact Us" — visible, obvious, engaging.
*Check:* the primary CTA is the highest-contrast element in the viewport, and there is only one per screen.

5️⃣ **Analytics & tracking.**
Implement GA4 or an equivalent so the client knows where users come from and where they drop off.
*Check:* open the dashboard. If it's empty, tracking isn't installed — this is the most commonly skipped rule.

6️⃣ **Regular communication.**
Progress reports, demos, feedback loops. Transparency builds trust.

7️⃣ **Post-launch support.**
Monitor after launch and suggest improvements.
*Check:* you would find out within an hour if deploys started failing. If not, you have no monitoring.

---

## Part 2 — Performance

8️⃣ **Measure the three numbers that matter, at the 75th percentile.** LCP ≤ 2.5 s, INP ≤ 200 ms, CLS ≤ 0.1.
*Check:* PageSpeed Insights on the real URL, mobile tab. Not Lighthouse on localhost.

9️⃣ **Ship less JavaScript.** Every kilobyte is parsed, compiled and executed before users can interact.
*Check:* Chrome DevTools → Coverage. Anything unused on first load should be code-split or deleted.

🔟 **Split by route and by interaction.** Load code when the user is about to need it, not at startup.
*Check:* the entry bundle contains nothing that only one screen uses.

1️⃣1️⃣ **Delete dead code and unused dependencies.** The cheapest byte is the one you don't ship.
*Check:* for each `package.json` entry, count the files that import it. Zero means delete.

1️⃣2️⃣ **Reserve space for everything that loads late.** Unsized images and late content are the top cause of CLS.
*Check:* every `<img>` and `<iframe>` has `width`/`height` (or `aspect-ratio`).

1️⃣3️⃣ **Lazy-load below the fold; never above it.** `loading="lazy"` on the hero delays the LCP request.
*Check:* the largest image in the first viewport loads eagerly, ideally with `fetchpriority="high"`.

1️⃣4️⃣ **Break up long tasks.** Anything over 50 ms blocks interaction.
*Check:* Performance panel → look for red triangles in the main thread during load.

1️⃣5️⃣ **Set a performance budget and defend it.** A common one is ≤ 170 KB of critical-path resources.
*Check:* the build output has a size limit, and CI fails when it's exceeded.

1️⃣6️⃣ **Cache aggressively, version everything.** Hashed filenames + long cache headers; HTML short-lived.
*Check:* a redeploy doesn't force re-download of unchanged vendor code.

---

## Part 3 — Assets

1️⃣7️⃣ **Images in WebP or AVIF, video in WebM.** PNG/JPEG are 4–10× larger at equivalent quality.
*Check:* `find . -name "*.png" -o -name "*.jpg"` in the served directory should return nothing you actually ship.

1️⃣8️⃣ **Keep SVG as SVG.** It's vector; rasterising it makes it bigger *and* non-scalable.
*Check:* logos and icons are `.svg`, not `.png`.

1️⃣9️⃣ **Never ship an asset nothing references.** Orphaned files are pure waste.
*Check:* grep the filename across the repo. Zero hits → delete.

2️⃣0️⃣ **Serve responsive images.** A 2000 px image in a 400 px slot wastes 90% of the bytes.
*Check:* `srcset` + `sizes` on any image wider than a phone screen.

2️⃣1️⃣ **Compress video, don't autoplay it.** `preload="none"` plus a poster frame.
*Check:* no video over ~500 KB is fetched by someone who never scrolls to it.

---

## Part 4 — Accessibility

2️⃣2️⃣ **Target WCAG 2.1 AA, and use the POUR test: Perceivable, Operable, Understandable, Robust.**
*Check:* axe DevTools in the browser; then repeat the main flow with the keyboard only.

2️⃣3️⃣ **Everything must work with the keyboard alone.** Focus visible, logical order, no traps.
*Check:* unplug the mouse. Tab through the whole page.

2️⃣4️⃣ **Never remove focus outlines without replacing them.** A custom ring is fine; nothing is not.
*Check:* `:focus-visible` styles exist and are visible against every background.

2️⃣5️⃣ **Text contrast ≥ 4.5:1** (3:1 for large text). Don't dim secondary text with opacity — pick a real colour.
*Check:* contrast checker on every muted/grey label.

2️⃣6️⃣ **Never convey meaning by colour alone.** Add an icon, label or pattern.
*Check:* screenshot it in greyscale; the meaning survives.

2️⃣7️⃣ **Use semantic HTML before ARIA.** `<button>`, `<nav>`, `<main>`, `<label>` — ARIA is a patch, not a substitute.
*Check:* a screen reader announces each control with a sensible name and role.

2️⃣8️⃣ **Every image needs `alt`** — descriptive, or `alt=""` if purely decorative.
*Check:* no `<img>` without an `alt` attribute (empty is valid, missing is not).

2️⃣9️⃣ **One `<h1>` per page, no skipped heading levels.** Headings are the screen-reader outline.
*Check:* DevTools accessibility tree — the outline reads like a table of contents.

3️⃣0️⃣ **Touch targets ≥ 44×44 px.**
*Check:* measure the smallest tappable thing on a phone with a thumb, not a mouse.

3️⃣1️⃣ **Dialogs trap focus, close on Escape, and restore focus on close.**
*Check:* open a modal with the keyboard, press Escape, and confirm focus returns where you were.

3️⃣2️⃣ **Provide a skip link** when navigation precedes content.
*Check:* press Tab once on page load — the first stop should be "Skip to content".

3️⃣3️⃣ **Respect `prefers-reduced-motion`.** Animations off, not just slower.
*Check:* enable it at OS level and confirm nothing still moves.

---

## Part 5 — Security

3️⃣4️⃣ **Treat all data as untrusted:** API responses, localStorage, URL params, hidden fields, third parties.
*Check:* grep for `innerHTML`, `dangerouslySetInnerHTML`, `eval`, `new Function`. Every hit needs justification.

3️⃣5️⃣ **Sanitise any HTML you render, with a maintained library (e.g. DOMPurify).**
*Check:* the sanitizer is a direct dependency and pinned; it gets updated.

3️⃣6️⃣ **Never put secrets in the frontend.** Anything shipped to the browser is public.
*Check:* search the bundle for `sk-`, `api_key`, `secret`, `token`.

3️⃣7️⃣ **Never rely on client-side logic for security or business rules.** Duplicate the check on the server.
*Check:* disable JavaScript and replay the request — the server still refuses.

3️⃣8️⃣ **Send a Content Security Policy.** Avoid `unsafe-inline` and `unsafe-eval`; use nonces or hashes.
*Check:* response headers include `Content-Security-Policy` (Report-Only does **not** block anything).

3️⃣9️⃣ **Pin and audit dependencies.** Lockfile committed, updates reviewed, advisories monitored.
*Check:* `npm audit` in CI; a lockfile in git.

4️⃣0️⃣ **Use HTTPS everywhere, and Subresource Integrity for third-party scripts.**
*Check:* no mixed content warnings; CDN scripts carry an `integrity` hash.

---

## Part 6 — SEO, Privacy & Trust

4️⃣1️⃣ **Semantic HTML, real meta tags, canonical URL, sitemap, robots.txt.**
*Check:* view source — title, description, `og:image`, canonical all present and pointing at URLs that exist.

4️⃣2️⃣ **Test your social preview.** A 404 `og:image` renders no card at all — and nobody notices until launch.
*Check:* paste the URL into a preview debugger; confirm the image loads.

4️⃣3️⃣ **Ask for consent before tracking, and collect the minimum.** Especially for health, finance or minors.
*Check:* with consent declined, no analytics or marketing request fires.

4️⃣4️⃣ **Show trust signals.** Contact details, privacy policy, and — for regulated products — a clear disclaimer.
*Check:* a cold visitor can find who made this and what it does not do.

---

## Part 7 — Code Health

4️⃣5️⃣ **Type-check in CI.** A bundler strips types without checking them; only `tsc` finds type errors.
*Check:* `tsc --noEmit` runs on every PR.

4️⃣6️⃣ **Test what breaks in production:** the money path, the auth flow, the API contract.
*Check:* tests run in CI and a failing test blocks merge.

4️⃣7️⃣ **Keep one source of truth for design tokens.** One border colour, one radius scale, one focus ring.
*Check:* grep for hardcoded hex values in components — should trend to zero.

4️⃣8️⃣ **Let the framework own the DOM.** Manual `appendChild`/`innerHTML` inside React/Vue causes crashes the framework can't recover from.
*Check:* no imperative DOM mutation of nodes the framework renders.

4️⃣9️⃣ **Handle empty, loading and error states for every data surface.** Blank space reads as broken.
*Check:* throttle the network and switch it off — each screen still says something useful.

5️⃣0️⃣ **Verify on a real device before calling it done.**
*Check:* one physical phone, one real network. Emulators lie about touch, fonts and safe areas.
