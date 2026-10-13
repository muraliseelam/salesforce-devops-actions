# Build & end-to-end test checklist

Working checklist for turning the pipeline into an open-source, zero-friction template and proving
it end to end against real Salesforce orgs (5 fresh scratch orgs from the `partner-pbo` Dev Hub) and
real GitHub Actions runs. Items are worked top to bottom; each is ticked only after it's verified.

Legend: `[x]` done and verified · `[~]` in progress · `[ ]` to do · `[!]` blocked (reason noted)

## A. Open-source architecture
- [ ] A1 Convention-over-configuration: env list is the only required input; branch, alias, secret names, status check, GitHub environment, instance URL and test policy all default from the env name; last env = production
- [ ] A2 Promotion order = `environments` order (works for 2-6 stages; 4 and 5 tested)
- [ ] A3 Two auth methods: `sfdx_url` (default, no Connected App) and `jwt` (External Client App / Connected App); per-env override
- [ ] A4 Missing-secret errors name the exact secret and the command that creates it
- [ ] A5 Approval model: `approved` label **or** N reviews → `sf-approval` status, required by branch protection
- [ ] A6 `approved` label is removed when new commits are pushed (configurable)
- [ ] A7 Approval guard on real deploys uses the same rule (label or reviews)
- [ ] A8 Two-phase release: dispatch → assemble + validate → `approved` label on the release PR → deploy → merge → tag → release → sync
- [ ] A9 `bootstrap.py init` — generate the config from env names
- [ ] A10 `bootstrap.py github` — branches, labels, merge settings, Actions permissions, environments, protection (idempotent)
- [ ] A11 `bootstrap.py secrets` — push auth secrets from locally logged-in orgs (auth URL or JWT) without printing them
- [ ] A12 `bootstrap.py doctor` — config, branches, labels, environments, protection, secret names
- [ ] A13 SF Pipeline Doctor workflow — logs in to every org from GitHub and reports
- [ ] A14 Pipeline self-test workflow — unit tests + actionlint on pipeline changes
- [ ] A15 Scratch-org definition for trying the pipeline without sandboxes

## B. Local verification
- [ ] B1 Unit tests cover config defaults, 4/5-stage chains, auth, approvals, bootstrap planning
- [ ] B2 actionlint + shellcheck clean (workflows + composite actions)
- [ ] B3 Real `sf` JSON captured from scratch orgs (success, test failure, coverage, compile error) and used as report fixtures
- [ ] B4 Local auth-URL login round trip works through the same commands sf-setup runs

## C. GitHub bootstrap
- [ ] C1 Pipeline committed and pushed to `main`
- [ ] C2 `bootstrap.py github` applied to the live repo
- [!] C3 Secrets set by the repo owner (`bootstrap.py secrets`) — needs the owner
- [ ] C4 Doctor workflow green for all five orgs

## D. End-to-end scenarios (5 stages: dev → int → qa → stage → prod)
- [ ] D1 Happy path: PR to dev with Apex + test → report → `approved` label → squash merge → deploy to dev org (verified in org)
- [ ] D2 Changed class without a test → fail fast, synthetic report, nothing deployed
- [ ] D3 Coverage below 85% → coverage gate fails
- [ ] D4 Failing Apex test → test failures table
- [ ] D5 Compile error → component failures table
- [ ] D6 Missing `-meta.xml` → preflight fails
- [ ] D7 Branch cut from an env branch → preflight fails
- [ ] D8 Feature PR straight into int → routing fails
- [ ] D9 Non-Apex metadata → NoTestRun dry-run validation
- [ ] D10 Deleted metadata → destructiveChanges deployed
- [ ] D11 Pipeline-only change → gate passes with "no Salesforce changes"
- [ ] D12 `deploy:<env>` label → pre-merge deploy, label removed
- [ ] D13 New push removes the `approved` label and blocks merge until re-approved
- [ ] D14 Promote dev → int → qa → stage via labels, validated and deployed at each step
- [ ] D15 Release dry run (PR stays unmergeable)
- [ ] D16 Release: validate → `approved` → quick deploy → merge (SHAs kept) → tag → GitHub Release
- [ ] D17 Post-release sync: env branches reset, protection restored exactly, unreleased commits listed
- [ ] D18 Quick-deploy fallback to full deploy
- [ ] D19 Concurrent merges to one env don't collide
- [ ] D20 Re-runs are idempotent (promote, release with same version)
- [ ] D21 Cleanup release removes the test metadata from production (destructive in prod)

## E. 4-stage variant (int → qa → stage → prod)
- [ ] E1 Switch config to 4 envs, re-bootstrap, PR → int → promote → release
- [ ] E2 Restore the 5-stage config

## F. Documentation & open-source polish (after testing)
- [ ] F1 README quick start (env names → bootstrap → secrets → first PR)
- [ ] F2 Docs: getting started, configuration reference, authentication, approvals, workflows, troubleshooting, FAQ
- [ ] F3 CONTRIBUTING, SECURITY, CHANGELOG, issue + PR templates
- [ ] F4 Repo marked as a template, description + topics
- [ ] F5 Publish the documentation
- [ ] F6 Final cleanup (test branches/PRs closed, scratch orgs listed for the owner)
