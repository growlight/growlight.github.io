# Growlight — public site

The pages served at <https://growlight.github.io/>: a studio landing page, and privacy and
support pages for each app. Plain static HTML, one stylesheet, no build step.

This repo is public so that GitHub Pages serves it for free. The apps themselves are private.

## This repo is a submodule of the app it describes

It is vendored into the CouplesBudget project at `external/growlight`, and the pointer on that
project's `main` records which version of the policy a given app version expects.

That means **editing a file here is not publishing it.** Publishing takes three steps, and it is
the middle one that gets forgotten:

1. Commit here.
2. **Push to `growlight/growlight.github.io`.** Until this happens nobody can read the change.
3. Commit the bumped submodule pointer in CouplesBudget, so `main` pins what is live.

There used to be a second copy of these files under `_docs/site/` in the app repo. It was deleted
because two copies of a legal document drift — and within a day, they had.

## Layout

One repo serves every app, each under its own directory:

```
growlight.github.io/
├── index.html                        → https://growlight.github.io/
├── style.css                           shared by every page
├── .nojekyll                           serve as-is, no Jekyll
└── couplesbudget/
    ├── index.html                    → /couplesbudget/
    ├── privacy/index.html            → /couplesbudget/privacy/
    └── support/index.html            → /couplesbudget/support/
```

App Store Connect wants a privacy URL, a support URL and an optional marketing URL per app;
those are the three `couplesbudget/` paths. A second app is a new directory and a card on the
landing page — no new repo, no Pages setup.

Every link is relative, so nothing depends on the hostname. The site can be served from any
origin unchanged.

## Renaming things

Nothing here is locked in, but the privacy URL is compiled into shipped app binaries and handed
to the App Store, so it has to keep resolving for builds already on people's phones.

**Renaming an app directory:** never delete the old path. Leave a stub at it —

```html
<meta http-equiv="refresh" content="0; url=/newname/privacy/">
<link rel="canonical" href="https://growlight.github.io/newname/privacy/">
<p>This page has moved to <a href="/newname/privacy/">/newname/privacy/</a>.</p>
```

Old builds keep working forever, for free. This is the advantage of one repo with directories:
you own the whole path namespace and can keep any URL alive.

**Renaming the org:** `growlight.github.io` stops resolving immediately, with no redirect. The
old name is released rather than destroyed, so claim it back at once — a new free org named
`growlight` holding a `growlight.github.io` repo full of redirect stubs. Have it ready *before*
renaming. A custom domain is the only genuinely rename-proof option.

## Keeping the policy honest

The app repo's `CLAUDE.md` section "Changes to what we collect are privacy-policy changes" says
when a code change obliges a policy review. This is the checklist it refers to — every claim
below was verified against the code and the live Supabase project, and each must be rechecked
when the code moves:

- Data is hosted by Supabase in AWS `us-west-2` (Oregon, USA) — the basis for the PIPEDA
  cross-border disclosure.
- No analytics, telemetry, crash-reporting or advertising SDKs. `supabase-swift` is the only
  package the project depends on directly; the transitive pins collect nothing. CI enforces the
  direct list.
- Database backups are retained 7 days on the Supabase free tier.
- Transaction deletion is a hard delete, with no tombstone rows.
- Account deletion removes the user's transactions, incomes, profile and unreferenced per-person
  budgets, and reassigns per-person budgets that still hold others' spending.
- Sign in with Apple requests `.fullName` and `.email`; a Hide My Email user only ever yields a
  relay address.
- Everything in a household is visible to both members.

Per-app privacy policies are deliberate. Apple expects each app's privacy URL to match that
app's App Privacy disclosures, and a shared studio-wide policy would either over-disclose for a
simple app or under-disclose for a complex one. Duplicate the boilerplate between apps instead;
a shared document drifts out of sync with at least one of them.

`policy-drift.yml` in the app repo fetches these pages daily and fails when they stop matching
the pinned submodule commit — catching both a commit that was never pushed here, and a push here
that `main` never pinned.

## The account-free tier

The app runs with no Supabase account at all (app repo issue #140): no sign-in, no sync, every
row on the device. The policy's "Two ways to use the app" section draws that line, and every
account-dependent claim after it — our database, cross-border transfer, sharing with the hosting
provider, backups, partner visibility, account deletion, the California categories — is scoped to
account holders explicitly.

Keep that scoping intact. If the free tier ever starts sending anything to a server, this is the
part of the policy that becomes false first, and it is currently the strongest privacy claim on
the page.
