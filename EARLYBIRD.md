# Earlybird

`earlybird` is FeatBit's working demo branch for customer demonstrations,
capability videos, and early product feedback. Demos can include real frontend,
backend, and database changes. They are not production releases.

Sync improvements from `main` into this branch as needed. Do not merge this demo
branch wholesale into `main`. When customer demand justifies a capability, the
team should implement and review its production version separately, using the
demo as a reference for behavior and user experience.

## Repository and branches

The dedicated repository is
[`featbit/featbit-earlybird`](https://github.com/featbit/featbit-earlybird), a public
fork of [`featbit/featbit`](https://github.com/featbit/featbit).

- `main` starts from upstream main and is reserved for upstream synchronization.
- `earlybird` contains the demos, including the Release Health implementation
  and the verified September merge. Push demo changes to this branch in the fork.
- Upstream synchronization is manual; creating the fork does not schedule syncs.

In the existing FeatBit checkout, `origin` still points to `featbit/featbit` and
the additional `earlybird` remote points to `featbit/featbit-earlybird`. The local
`earlybird` branch tracks the fork's `earlybird` branch. To sync future changes
from upstream, resolve any conflicts and validate before the final push:

```sh
git fetch origin main
git switch earlybird
git merge --no-ff origin/main
# Run the relevant builds and tests before pushing.
git push earlybird earlybird
```

For a separate checkout, clone the demo branch and add the upstream remote:

```sh
git clone --branch earlybird https://github.com/featbit/featbit-earlybird.git
cd featbit-earlybird
git remote add upstream https://github.com/featbit/featbit.git
```

In that separate checkout, `origin` is the fork and `upstream` is the main
repository; fetch and merge `upstream/main` to bring in future upstream changes.

## September 2026 sync

- Demo starting point: `releasehealth-design` at `cd28a0ce9`.
- Main starting point: `origin/main` at `adb5ecb61`, fetched on 2026-09-29.
- The Release Health frontend, backend, webhooks, and additive PostgreSQL script
  are retained. The previous branch remains available for comparison.
- The old Release Decision coming-soon pages are replaced by main's implemented
  Experiments, Metrics, and Layers pages and translation resources.
- Flag confirmation keeps main's key/comment requirements and the demo's
  supplemental Release Health content.
- Resource webhooks keep main's pagination and search fixes within the demo's
  tabbed webhook page.

## PostgreSQL

Main's versioned initialization and upgrade scripts are under
`infra/postgresql/docker-entrypoint-initdb.d/`. Release Health additionally needs
[`modules/back-end/scripts/release-health/postgres.sql`](modules/back-end/scripts/release-health/postgres.sql).
It creates `release_health_documents` and is safe to apply repeatedly.

For a fresh database, apply the versioned scripts in version order through
`v6.0.0.sql`, then apply the Release Health script to the `featbit` database.
The existing local setup helper and demo instructions are in
[`integration-tests/release-health/README.md`](integration-tests/release-health/README.md).

An existing demo volume is not upgraded by a Git merge or by restarting Aspire.
In particular, do not blindly replay `v6.0.0.sql` against a database that already
ran the old `vNext.sql` or other pre-release experiment scripts. The v6 script
adds columns and renames experiment tables; those operations require checking
the current schema and backing up the database before planning its upgrade.
Keep the Release Health documents and existing encryption key together when
preserving demo connections and their credentials.

## Validation of this sync

- Frontend TypeScript check and Vite build passed.
- Frontend coverage: 158 test files / 843 tests. The initial run passed 154 files;
  after correcting a stale demo copy assertion, the remaining four files passed
  all 69 tests with `--maxWorkers=1 --testTimeout=15000`. Their initial run also
  hit UI timing limits while builds and tests were running concurrently.
- ESLint passed for the five frontend source files involved in conflict
  resolution and the transferred webhook fixes.
- Backend build and 1,127 tests passed with `Category!=Integration`.
- In a disposable PostgreSQL 15.10 container, a database initialized with the
  pre-v6 released scripts plus a saved Release Health document upgraded through
  `v6.0.0.sql` while preserving that document, its version, and protected secrets.
- A fresh database initialized through v6 accepted the Release Health script.
  Reapplying the Release Health script preserved existing documents in both
  scenarios.
- These SQL checks used generated test data. The existing Aspire database volume
  was not modified, and its pre-release schema upgrade remains a separate task.
