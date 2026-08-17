# fem-cicd-service — POC stage

This branch holds the end-of-segment-6 state of the Frontend Masters workshop "Cloud CI/CD with GitHub Actions". The pipeline builds the sample app and ships `dist/` to S3 on every push to `main`. It is the first of three progressive end-states — the other two live on the `stable` and `enterprise` branches.

## What this branch shows

The pipeline is a single workflow file at `.github/workflows/deploy.yml` with two jobs. The `build` job runs on `ubuntu-latest`, checks out the repo, sets up Node, runs `npm ci` and `npm run build`, then uploads `dist/` as a build artifact. The `deploy` job needs `build`, downloads the artifact, configures AWS credentials from the IAM-user keys stored in repository secrets, and runs `aws s3 sync` to push the artifact to the bucket. Triggers are `push` to `main` and `workflow_dispatch`.

## AWS prerequisites

Create these by hand in the AWS console before pushing. The README uses `<example-bucket>` as a placeholder — substitute your real bucket name where indicated.

- One S3 bucket with static-website hosting enabled.
- A bucket policy on `<example-bucket>` granting public read (`s3:GetObject`) on every object. The website endpoint serves traffic directly from S3 — there is no CDN at this stage.
- One IAM user with programmatic access. Attach an inline policy granting `s3:PutObject`, `s3:DeleteObject`, and `s3:ListBucket` on `<example-bucket>` and its contents.
- The IAM user's access key ID and secret access key. You will paste these into GitHub repository secrets in the next section.

## GitHub prerequisites

Configure these on the repository hosting this branch before the first push.

- The repository must be public. Environment protection rules and required reviewers are free only on public repos, and the Stable / Enterprise stages depend on those features.
- Two repository secrets: `AWS_ACCESS_KEY_ID` and `AWS_SECRET_ACCESS_KEY`, set to the IAM user's credentials. Treat these as demo-only and rotate them before the workshop — never reuse production credentials in a teaching repo.
- Replace the `<example-bucket>` placeholder in `.github/workflows/deploy.yml` with the real bucket name before pushing. The workflow will fail otherwise.
- The `us-east-1` region in the workflow must match the bucket's region. If the bucket lives elsewhere, edit the workflow accordingly.
- Branch protection, environments, and OIDC are intentionally not configured at this stage. Push to `main` deploys directly.

## What was deliberately left as the wrong choice

These choices are wrong on purpose. Each one is fixed in a later stage — they are the lead-in for the Stable and Enterprise segments.

- Long-lived AWS credentials in repository secrets. Anyone with write access to the repo can exfiltrate them. The `enterprise` branch replaces this with OIDC and a short-lived role assumption.
- No PR gate. Every push to `main` deploys directly with no review. The `stable` branch adds a `pull_request`-triggered CI workflow plus branch protection that blocks merges until checks pass.
- No dependency caching. Every CI run reinstalls npm from scratch. The `stable` branch enables `actions/setup-node`'s built-in cache.
- No concurrency control. Two simultaneous pushes to `main` race each other to deploy. The `enterprise` branch adds `concurrency:` blocks to serialize deploys.
- Public-read S3 bucket and no CDN. The `enterprise` branch fronts the bucket with CloudFront and locks origin access down with an Origin Access Control policy.

## Linkouts

Workshop and reference documentation lives on `main`; the branches below hold the matching code end-states.

- Master outline: `docs/instructor/OUTLINE.md`.
- POC stage step-by-step guide: `docs/instructor/POC.md`.
- Architecture TDD: `docs/tdd/fem-cicd-workshop-architecture.md`.
- Stable stage reference: run `git checkout stable`.
- Enterprise stage reference: run `git checkout enterprise`.
- Workshop URL: https://frontendmasters.com/workshops/github-actions/.
