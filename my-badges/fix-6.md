<img src="https://my-badges.github.io/my-badges/fix-6.png" alt="I did 6 sequential fixes." title="I did 6 sequential fixes." width="128">
<strong>I did 6 sequential fixes.</strong>
<br><br>

Commits:

- <a href="https://github.com/ayushsingh-byte/SystemFlow/commit/f50a71ea91dfcb157937596ba8723d4f07eda9fb">f50a71e</a>: fix(auth+ui): real session wiring, same-origin CORS, server-side premium redemption

- CORS: same-origin POST/PUT/PATCH requests carry an Origin header and were
  rejected when ALLOWED_ORIGINS was empty ('CORS: origin not allowed' on the
  staff-code verify and every other mutation). The delegate now always allows
  the origin matching the serving host.
- Canvas store: hydrate the signed-in user from /api/user/me on load (was
  permanently stuck on the 'Local User' placeholder), redirect to login when
  no session, sync plan, persist name edits via PATCH /api/user/profile.
- Sign out: end the Supabase session (canvas + shared nav) — previously only
  sf_* keys were cleared so the next page silently signed back in.
- account.html: populate profile from /api/user/me instead of never-set
  localStorage keys; Save Changes persists to the backend; plan shown in
  sidebar; redirect to login when unauthenticated.
- Premium redemption: new POST /api/user/redeem validates PREMIUM_ACCESS_CODE
  (constant-time, staff-code rate limit) and upgrades the account to Pro.
  Upgrade-to-Pro modal and the user-admin promo page Validate button now call
  it — replaces the old client-side TRIAL2025 check that only set a local flag.

Co-Authored-By: Claude Fable 5 <noreply@anthropic.com>
- <a href="https://github.com/ayushsingh-byte/SystemFlow/commit/7e3ae82f820dfe4038759d08707cd3c2b68c0001">7e3ae82</a>: fix(auth): send email confirmation back to deployed origin

signUp now passes emailRedirectTo so Supabase confirmation links return
to the site the user signed up on instead of the project's default Site
URL (which pointed at localhost:3000).

Co-Authored-By: Claude Fable 5 <noreply@anthropic.com>
- <a href="https://github.com/ayushsingh-byte/SystemFlow/commit/da31e3969f6562d0d1358990215830874e0f6262">da31e39</a>: fix(landing): redirects for legacy marketing URLs

/status, /case-studies, /blog, /changelog, /roadmap, /security-review,
/legal/*, /docs/* now redirect to their nearest existing page instead
of 404ing.

Co-Authored-By: Claude Fable 5 <noreply@anthropic.com>
- <a href="https://github.com/ayushsingh-byte/SystemFlow/commit/16deff9d95542c78fd5112f2676469236f66ed4b">16deff9</a>: fix(landing): wire marketing CTAs to app, kill dead links

Register/login/Start Free buttons pointed at /register and /login, which
don't exist on the marketing site (404). Auth lives in the Railway app
(login.html / signup.html), so:

- lib/app-url.ts: central APP_URL (NEXT_PUBLIC_APP_URL override) with
  LOGIN/SIGNUP/PRIVACY/TERMS URLs
- all Start Free / Log in CTAs -> app signup/login
- /auth/login and /auth/register placeholders now redirect to the app
- next.config redirects for legacy /register, /signup, /login
- dead nav/footer routes remapped to existing pages (blog/changelog/
  roadmap->resources, case-studies->customers, status->trust-center,
  legal/*->app privacy/terms or /compliance, docs/*->/docs, partners/
  security-review->contact-sales, press-kit->about, roi-calculator->pricing)

Co-Authored-By: Claude Fable 5 <noreply@anthropic.com>
- <a href="https://github.com/ayushsingh-byte/SystemFlow/commit/59173a824fb11686ede852c6a4ba3c97d16106f2">59173a8</a>: fix(deploy): rename 'Landing Pages' to landing-pages

Vercel rejects serverless function paths containing spaces
(invalid_function_name on 'Landing Pages/___next_launcher.cjs').
Update CI workflow and README references.

Co-Authored-By: Claude Fable 5 <noreply@anthropic.com>
- <a href="https://github.com/ayushsingh-byte/SystemFlow/commit/5f14a20db4fff4a193dfdcafdb53a059c87e2174">5f14a20</a>: fix(deploy): make repo Railway + Vercel deployable

- railway.json: build/start in backend/, /health healthcheck
- fix 'Error pages' static path case (Linux case-sensitive)
- Dockerfile: run from /app/backend so CMD resolves server.js
- db.js: optional DATABASE_CA_CERT for strict Postgres TLS
- drop unused SUPABASE_JWT_SECRET requirement, require SUPABASE_URL
- Landing Pages: default exports for site-header/footer (next build fix),
  eslint flat config + deps, escape apostrophe, drop duplicate pnpm lockfile
- root package.json: engines >=20, prune dead landing script, regen stale lock
- README deployment guide, .env.example notes

Co-Authored-By: Claude Fable 5 <noreply@anthropic.com>


Created by <a href="https://github.com/my-badges/my-badges">My Badges</a>