# Growlight — public site (staged, not yet published)

Landing, privacy and support pages for Growlight's apps, written for issue #121. They are **not
live anywhere yet**: publishing needs the GitHub org to exist.

These files live under `_docs/` so they don't trigger a TestFlight build
(`.github/workflows/beta.yml` has `paths-ignore: ['_docs/**', '**/*.md']`). They are staged here
so the work isn't lost; the intended home is a separate public repository.

Note `.gitignore` has a blanket `*.html` rule under "Local artifacts". A companion
`!_docs/site/**/*.html` negation keeps these pages tracked. If you move this directory, move that
negation with it, or the pages will vanish from commits without any warning.

## Layout

One repo serves every app, each under its own directory:

```
growlight.github.io/
├── index.html                        → https://growlight.github.io/
├── style.css                           shared by every page
├── .nojekyll
└── couplesbudget/
    ├── index.html                    → /couplesbudget/
    ├── privacy/index.html            → /couplesbudget/privacy/
    └── support/index.html            → /couplesbudget/support/
```

App Store Connect wants a privacy URL, a support URL and an optional marketing URL per app;
those are the three `couplesbudget/` paths. A second app is a new directory and a card on the
landing page — no new repo and no Pages setup.

Every link is relative, so nothing depends on the hostname. The site can be served from any
origin unchanged.

## Publishing

1. Create a free org named `growlight` at <https://github.com/organizations/new>.
2. Inside it, create a **public** repo named exactly `growlight.github.io`.
3. Copy the *contents* of this directory to that repo's default branch — `index.html` must land
   at the repo root, not inside a `site/` folder.
4. Org sites publish from the default branch automatically; check Settings → Pages for the build.
5. Bump the `Last updated` date in `couplesbudget/privacy/index.html` if publication has
   slipped well past it.

## Renaming things later

Nothing here is locked in. The org can be renamed whenever you like — but `growlight.github.io`
stops resolving the moment you do, with no redirect, and that URL will be compiled into shipped
builds. Two rules keep that cheap:

**Renaming an app directory:** never delete the old path. Leave a stub at it:

```html
<meta http-equiv="refresh" content="0; url=/newname/privacy/">
<link rel="canonical" href="https://growlight.github.io/newname/privacy/">
<p>This page has moved to <a href="/newname/privacy/">/newname/privacy/</a>.</p>
```

Old builds keep working forever, for free.

**Renaming the org:** the old name is released rather than destroyed, so claim it back
immediately — a new free org named `growlight` holding a `growlight.github.io` repo full of
redirect stubs. Have it ready *before* renaming, so the window where someone else could take the
name is measured in seconds.

## Drift between the two copies

`.github/workflows/policy-drift.yml` fetches the live pages daily and fails if they do
not match this directory. It is scheduled rather than a PR check: a PR that edits the
policy makes live and source differ by definition, and the only way to satisfy a PR check
would be publishing unreviewed policy text. What it catches is a merged change nobody
republished. Run it on demand from the Actions tab after publishing.

## Keeping the policy honest

The root `CLAUDE.md` section "Changes to what we collect are privacy-policy changes"
describes when a code change obliges you to revisit the policy. This list is the
checklist it refers to — keep the two in step.

Every factual claim in the privacy policy was verified against the codebase and the live Supabase
project when written. If any of these change, the policy has to change with it:

- Data is hosted by Supabase in AWS `us-west-2` (Oregon, USA) — the basis for the PIPEDA
  cross-border disclosure.
- No analytics, telemetry, crash-reporting or advertising SDKs. `supabase-swift` is the only
  package the project depends on **directly**; the six transitive pins in `Package.resolved`
  (Apple's swift-crypto/asn1/http-types, Point-Free's clocks/concurrency-extras/xctest-overlay)
  are ordinary libraries that collect nothing. CI enforces the direct list — see
  `.github/workflows/ci.yml`.
- Database backups are retained 7 days on the Supabase free tier.
- Transaction deletion is a hard delete, with no tombstone rows.
- Sign in with Apple requests `.fullName` and `.email`; a Hide My Email user only ever yields a
  relay address.
- Everything in a household is visible to both members.

Per-app privacy policies are deliberate. Apple expects each app's privacy URL to match that app's
App Privacy disclosures, and a shared studio-wide policy would either over-disclose for a simple
app or under-disclose for a complex one. Duplicate the boilerplate between apps instead; a shared
document drifts out of sync with at least one of them.

## Known gap

The policy promises access, export and deletion. Export is real — Settings → Export Data (issue
#120) produces transactions and income as CSV, which is also what satisfies GDPR Art. 20, and the
policy points at it rather than at an email request.

Deletion is **not currently possible to honour**: `transactions.spender_id`, `incomes.earner_id`
and `user_profiles.user_id` reference `auth.users` with no `ON DELETE` behaviour, so removing a
Supabase auth user fails outright. That is issue #119, and it needs to ship before the policy's
deletion promise is true.
