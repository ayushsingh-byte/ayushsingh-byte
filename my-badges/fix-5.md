<img src="https://my-badges.github.io/my-badges/fix-5.png" alt="I did 5 sequential fixes." title="I did 5 sequential fixes." width="128">
<strong>I did 5 sequential fixes.</strong>
<br><br>

Commits:

- <a href="https://github.com/ayushsingh-byte/SystemFlow/commit/f4a3b5fd7105efac748f831a4c7ee864a53ec14b">f4a3b5f</a>: fix(ml): anchor baseline tests to a fixed instant, not wall-clock time

CI has been red on the last two commits with:

    FAILED ml/tests/test_baseline.py::test_baseline_accumulates
    AssertionError: assert 17 == 20

Nothing in either commit touched Python. This is a latent flake that finally
surfaced.

BaselineEngine keys its buckets by (node_id, signal, hour_of_day, day_of_week).
The tests start at `time.time()` and step timestamps forward by 15s for up to
50 readings — a 12.5 minute span. When a run starts near the end of a UTC hour
the readings straddle the boundary, get split across two buckets, and the final
bucket is short by however many landed in the previous hour. "17 == 20" is
exactly three readings on the far side of the line.

That made the suite fail for real on roughly the last 10 minutes of any hour;
c630c4d passed at 15:04 UTC, the two failures ran near the top of an hour.

Anchored to a fixed mid-hour instant, matching what test_time_bucket_separation
already did with its own fixed epoch. Full suite: 92 passed.

Co-Authored-By: Claude Opus 5 <noreply@anthropic.com>
- <a href="https://github.com/ayushsingh-byte/SystemFlow/commit/9253166129bcc2e730cfc75564533191790279d7">9253166</a>: fix(sw): stop intercepting Google Fonts CSS; focus the landing dialog on open

Two defects found by the production smoke test.

1. The service worker silently killed every webfont.
   fonts.googleapis.com was in CDN_HOSTS and served through cacheFirst. A <link>
   fetches that stylesheet in no-cors mode, so the worker could only ever hand
   back an OPAQUE response — and the browser cannot parse CSS out of an opaque
   response. Measured on production: document.fonts.size was 0 with the worker
   controlling and 49 with it disabled (?nosw=1), while the requests still
   looked like they had succeeded. Cache.put() rejects on opaque responses too,
   so nothing was being stored in exchange for the breakage.

   fonts.googleapis.com is no longer intercepted. fonts.gstatic.com stays —
   @font-face fetches those with anonymous CORS, so the responses are real and
   cacheable. The `res.type === 'opaque'` branches are gone from the runtime
   handlers for the same reason: they could never have cached anything.

2. The landing What's New dialog never took focus.
   Focus was requested in an effect that runs on the first render, but that
   render returns null while `mounted` is false, so every ref was still null and
   the focus() call did nothing. Keyboard users landed behind the dialog with
   only the Tab trap holding them. Focus now runs in an effect keyed on
   `mounted`, once the portal actually exists.

Co-Authored-By: Claude Opus 5 <noreply@anthropic.com>
- <a href="https://github.com/ayushsingh-byte/SystemFlow/commit/d59f509b8e09921a00b8f64cadc640cbfa4fd99e">d59f509</a>: fix: crash modal, landing dialog position, docs redirect loop, plan tint

Five defects, four of them layout/config, all reported from production.

1. Clicking a crashed node blurred the app and showed nothing.
   CrashDetailModal rendered the backdrop and the card as SIBLINGS, but
   .crash-modal-backdrop is the fixed, flex-centering parent that positions the
   card (styles.css). As a sibling the card had position:static, sat in normal
   document flow, and landed underneath the backdrop's z-index:1000. Measured
   against the real stylesheet: the card resolved to top:876 in an 862px
   viewport — off-screen, behind the blur. Nesting it restores the centering the
   CSS was always written for; the backdrop only closes on a click that actually
   hits the backdrop.

2. Landing "What's New" dialog hung off the top of the screen.
   The nav trigger lives inside SiteHeader, which has `backdrop-blur-md`, and a
   backdrop-filter makes an element the containing block for position:fixed
   descendants. `fixed inset-0` was therefore resolving against a 64px header
   instead of the viewport, so the card overflowed upward with its own header
   cut away. The dialog is now portalled to document.body.

3. /docs returned ERR_TOO_MANY_REDIRECTS.
   `{ source: '/docs/:path*' }` — `:path*` matches ZERO or more segments, so
   /docs matched the rule and redirected to itself. Changed to `:path+`, which
   requires at least one segment.

4. The whole product UI was yellow on the Team plan.
   `html[data-plan="team"]` overrode --accent with #eab308, which repainted
   primary buttons, the active tab, filter chips, the run control and focus
   rings. Yellow reads as a warning in an app whose visual language already uses
   amber for saturation and red for failure. Plans now set --plan-accent only —
   plan pills and upgrade surfaces stay tinted, the product accent stays green
   on every plan. Node status colours are a separate scale and are untouched.

5. Latest release renumbered v0.9.0 -> v2.18, continuing from v2.17 rather than
   restarting below it.

Co-Authored-By: Claude Opus 5 <noreply@anthropic.com>
- <a href="https://github.com/ayushsingh-byte/SystemFlow/commit/c630c4d0145e723ea4f0ab7d4ce5caebe212308f">c630c4d</a>: fix(ci): patch high-severity CVEs in backend deps, repair the health check

`npm audit --audit-level=high` was failing the backend job:

  brace-expansion <=5.0.7   HIGH   DoS via exponential-time expansion and
                                   unbounded expansion length (OOM)
                                   GHSA-3jxr-9vmj-r5cp, GHSA-mh99-v99m-4gvg
  body-parser <1.20.6       LOW    size enforcement silently disabled when the
                                   limit value is invalid (GHSA-v422-hmwv-36x6)

`npm audit fix` resolves both without a breaking change: brace-expansion
5.0.6 -> 5.0.8 and express's nested body-parser 1.20.5 -> 1.20.6. Verified with
a clean `npm ci` followed by `npm audit --audit-level=high`, which now exits 0.

uuid@9.0.1 (moderate, GHSA-w5hq-g745-h8pq) is deliberately left alone. The
advisory is a missing buffer bounds check in v3/v5/v6 when a `buf` argument is
supplied; this codebase only calls v4 and never passes a buffer, so it is not
reachable. The fix requires uuid@14, a major bump across four call sites, and
moderate is below the CI threshold. Worth doing on its own, not as part of a
CI-unblocking change.

Also fixes the "Check server starts cleanly" step, which could not fail. On a
refused connection curl prints "000" via -w AND the `|| echo "000"` appended a
second one, so STATUS became "000000" — not equal to "000", so the retry loop
broke on the first attempt and the final assertion passed against a server that
never started. Using `|| true` keeps curl's own "000" and lets the loop retry.
Confirmed locally: the server needs ~6s to bind, so the old loop was exiting
long before it was ready.

Co-Authored-By: Claude Opus 5 <noreply@anthropic.com>
- <a href="https://github.com/ayushsingh-byte/SystemFlow/commit/9531214fb090014582d9bd559beb9297cdbde34d">9531214</a>: fix(deploy): ship the app bundle in the image, keep Node build out of shared config

The last two deploys failed with:

    [build:app] esbuild is not installed and public/dist/app.js does not exist

Two separate mistakes, both mine.

1. .dockerignore had a blanket `**/dist`, which stripped public/dist/app.js out
   of the build context. The file is committed, but the builder never saw it —
   and even had the build passed, the running image would have 404'd on the
   bundle the app boots from. Added an explicit negation for public/dist.

2. railway.json lives at the repo root and applies to EVERY service in the
   project, including the Python ml-service — the same trap 2a64a90 fixed for
   startCommand. Putting `npm run build:app` there meant a Node toolchain build
   ran against a Python service, and depended on esbuild, which the deploy image
   does not install (root `npm ci` runs with NODE_ENV=production, so
   devDependencies are skipped). buildCommand is back to the service-agnostic
   `cd backend && npm ci --omit=dev`.

Generated artifacts stay committed and are no longer regenerated at deploy time.
scripts/verify-artifacts.mjs is the guard that makes that safe: it fails if
public/dist/app.js is older than any file in app-modules.json, if sw.js is not
stamped with the current build id, if the manifest precaches something missing,
or if the landing site's changelog copy has drifted. Plain node, no
dependencies, wired into `npm run build`.

Co-Authored-By: Claude Opus 5 <noreply@anthropic.com>


Created by <a href="https://github.com/my-badges/my-badges">My Badges</a>