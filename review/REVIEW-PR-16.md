# Review: PR #16 — Harden WordPress.org deploy and replace EB_PAT

- PR: https://github.com/EssentialBlocks/testimonial-wp-block/pull/16
- Branch reviewed: `ci/split-controls-token` (copy: `review/testimonial-pr-16`), base `master`
- Reviewer: Abdullah Aas Suhaeel
- Verdict: **Content approved, but the PR is already redundant: recommend closing it**

## Already applied
- `ci/split-controls-token` was merged into `dev` (`c4f7e91 Merge branch 'ci/split-controls-token' into dev`) and reached `master` through #21 / #22.
- `.github/workflows/deploy.yml` on `master`, on `dev` and on this PR's head are **identical** (`git diff` is empty).
- The workflow has already run in production: run `40339974304`, "Release 1.3.1", event `release`, **success** (2026-10-26). That is the PR's one unchecked test-plan item ("First published release deploys to WordPress.org"), now satisfied.
- Merging it into `master` would change nothing.

## Content review (the workflow itself)
What it does: deploy on `release: published` instead of tag push; fail before building if the tag, plugin header `Version`, readme `Stable tag` or `TESTIMONIAL_BLOCKS_VERSION` disagree; check out `controls` and `lib/style-handler` at the pinned commits with a read-only fine-grained `CONTROLS_PAT` instead of `EB_PAT`; read-only `GITHUB_TOKEN`; no persisted credentials; serialized deploys; 30 min timeout; `actions/checkout@v7`, `actions/setup-node@v7`, `10up/action-wordpress-plugin-deploy@2.3.0`, Node 22.

Verified
- `checkout v7.0.1`, `setup-node v7.1.0` and `10up ... 2.3.0` exist as tags.
- The three version sources exist and agree on `master`/`dev` (`1.3.1`), and the `grep`/`sed` extraction matches the file formats.
- `git rev-parse HEAD:controls` works without initialised submodules, so the pinned-commit lookup is valid.
- The PR is `MERGEABLE` / `CLEAN`.

## Findings
No blocking bugs. Security direction is good (smaller token scope, no persisted credentials, pinned action).
1. (nit) `release: published` also fires for pre-releases. A pre-release tag such as `1.4.0-beta` would fail the version check (safe) but would still start a deploy run. Consider `types: [released]` if pre-releases should never deploy.
2. (nit) The 10up action is pinned to a tag. Pinning to a full commit SHA is stricter for supply-chain safety.
3. (process change, intentional) Publishing a GitHub **release** is now required to deploy; pushing only a tag no longer deploys. Make sure the release checklist says this.
4. (info) Needs the org secret `CONTROLS_PAT` to stay valid and not expire; an expired fine-grained PAT will fail the deploy at the checkout step.
