# 5. Pin key sites to frozen minor versions, and release everything else as patches

- Status: proposed
- Date: 2026-10-15

## Context and Problem Statement

This doc outlines a simple way to release design system updates to production sites. Right now the design system has no versioning, so every change goes straight to every site with no testing.

Three considerations were addressed:

Consideration 1: We need a testing process so changes get checked before they reach key public-facing sites. We also don't want to hand-update 14 sites every time we make a change.

Consideration 2: The design system isn't one file. It's a group of related files (components, tokens, fonts, logos) that have to be versioned together.

Consideration 3: We want to build on the release workflow Tommy already created instead of starting from scratch.

## Decision Drivers

- Changes should be checked before they reach public-facing sites.
- Updating a site's URL is manual work, and there are about 14 sites. Most
  releases should need no URL changes at all.
- The release process should build on the existing Release and Publish to S3
  workflows rather than replace them.
- A version's files have to stay together.

## Considered Options

1. No versions: every site loads the latest release
2. Every site pins an exact version
3. Pre-releases: staging sites pin a beta, key sites pin a minor version
4. Key public-facing sites pin a stable minor version; everything else is unpinned

## Decision Outcome

**Chosen: Option 4.**

### Option 1: no versions — rejected

This is how CL-web-components works today. Nothing can be tested on a real
site before it reaches every site, and no site can wait.

### Option 2: exact pinning everywhere — rejected

The most control, and the most work. Every patch would mean manually editing about 14 sites' URLs.

### Option 3: pre-releases — rejected

Proposed first, and workable. Minor and major changes would go out as betas
(`1.5.0-beta.1`) to their own folder, and staging sites would pin to them. It
needs pre-release support in `tools/bump-version.js` and `release.yml`, two
more choices in the Release workflow, and a URL change on every staging site
for every beta. Option 5 gets the same protection for key sites with none of
that.

### Option 5: frozen minor versions for key sites — chosen

Sites are grouped:

| Group | Loads | Examples |
|---|---|---|
| Key sites | A minor version folder, e.g. `/design-system/v1.5/` | CaltechAUTHORS, CaltechDATA, Oral Histories, library website |
| Staging and other sites | The main folder, `/design-system/` | Staging sites, dataset, datatools, pairtree |

How a change reaches sites:

1. Every change is released as a **patch** (`1.5.1`). It is published to the
   main folder, so staging and other sites get it immediately.
2. The team checks it on staging.
3. When it passes, release a **minor** version (`1.6.0`). That publishes to the
   main folder and creates a new folder, `/design-system/v1.6/`.
4. Announce the new URL. Each key site updates to it when ready.

A minor version folder is written once, when it is created, and never changed.
That is what guarantees key sites only get what passed on staging.

A **major** version follows the same steps as a minor. None is expected unless
there is a redesign.

Versions are folders, not file names (`/v1.6/design-system.js`, not
`/design-system-1.6.js`), so all of a version's files stay together and the
files can find each other by relative path.

## Consequences

Good:

- Key sites only change when their owners update a URL, and only to something
  that was checked on staging.
- Most releases need no URL changes.
- `tools/bump-version.js` and `release.yml` need no changes. Patch, Minor and
  Major already exist.

Bad, and accepted:

- **Key sites do not get urgent fixes until the next minor release.** A minor
  can be released any time staging looks good, so the wait can be short.
- **Patches reach other sites after local testing only.** Those sites are
  lower risk, and a problem there can be fixed with another patch.
- **New releases must never require HTML changes.** New features are opt-in,
  for example a new attribute, and existing markup always keeps working. This
  is what lets most sites stay unpinned. The tag names and attributes are
  final as of 1.0.0.
- **The library website has no staging site.** It is checked on the demo pages
  instead, and updates its pinned URL last.
- **Old minor folders accumulate.** They are cleaned up monthly, after
  checking that no key site still loads them.

## More Information

- `publish-to-s3.yml` needs one change: always upload to the main folder, and
  when the patch number is 0 (`1.6.0`, `2.0.0`) also upload to the minor
  folder (`/v1.6/`).
- The first release will be `1.0.0`, made by choosing Major from `0.0.0`.
- The earlier pre-release proposal is in Confluence, "Release Workflow Option
  for Design System". This ADR supersedes it.
