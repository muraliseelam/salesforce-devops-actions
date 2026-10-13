# salesforce-devops-actions
Salesforce DevOps Pipeline using GitHub Actions: a config-driven, single-repo CI/CD pipeline.

```
feature/* ──PR──▶ dev ──promote:int──▶ int ──promote:qa──▶ qa ──promote:stage──▶ stage ──release──▶ main (Production)
                   │                    │                   │                     │                    │
              validate+deploy      validate+deploy     validate+deploy      validate+deploy     validate → approve → quick deploy
                                                                                                      └─▶ tag · release · sync env branches · Webex
```

- **One source of truth:** [`sf-pipeline/pipeline.config.yml`](sf-pipeline/pipeline.config.yml) holds orgs, branches, secret names, test policy and coverage threshold.
- **Delta-only deploys:** `sfdx-git-delta` from `merge-base(origin/main, HEAD)` builds `package.xml` + `destructiveChanges.xml`. The same prod-relative artifact is used in every environment.
- **Smart tests:** RunSpecifiedTests from the changed classes' companion tests. Fails fast if a changed class has no test, with no silent RunLocalTests fallback. RunLocalTests (or a canary suite) in production.
- **🚀 Salesforce Pipeline Report** on every PR: component/test failures, coverage per changed class vs the gate, and the `deploy-<env>` required status check.
- **Guardrails:** branch protection, meta-pair and cut-from-main preflights, PR routing, approval guard, per-org concurrency locks, and env-branch sync with exact protection restore.

| Workflow | When |
|---|---|
| [SF PR Validation](.github/workflows/sf-pr-validation.yml) | Every PR into an environment branch |
| [SF Env Deploy](.github/workflows/sf-env-deploy.yml) | Merge into dev/int/qa/stage, or the `deploy:<env>` label |
| [SF Promote](.github/workflows/sf-promote.yml) | `promote:<env>` label on a merged PR |
| [SF Prod Release](.github/workflows/sf-prod-release.yml) | Manual, from `main` |
| [Sync Env Branches](.github/workflows/sync-env-branches.yml) | After each release (or manual) |

**Setup, flow diagrams and runbook:** [docs/PIPELINE_OVERVIEW.md](docs/PIPELINE_OVERVIEW.md)

```bash
pip install -r sf-pipeline/scripts/requirements.txt
python sf-pipeline/scripts/pipeline_config.py validate
python -m unittest discover -s sf-pipeline/tests -v
```
