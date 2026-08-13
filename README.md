# MOSEGICCOMPANYLIMITED — GreatWorkflowRelief

Quickstart (local development)

Prerequisites
- Node.js >= 18 and npm
- PostgreSQL (local or remote)

1) Clone or open the project

2) Install dependencies

```bash
npm install
```

3) Set the database URL

Export a DATABASE_URL environment variable that points to your Postgres instance, for example:

```bash
export DATABASE_URL="postgresql://user:password@localhost:5432/greatworkflowrelief?schema=public"
```

4) Generate Prisma client

```bash
npx prisma generate
```

5) Apply migrations (development)

```bash
npx prisma migrate dev --name init
```

For production deploys, use:

```bash
npx prisma migrate deploy
```

6) Seed the database (TypeScript seed script provided)

Requirements: ts-node (installed as devDependency in package.json)

```bash
npm run seed
```

Alternate: Run the raw SQL migration directly with psql if you prefer

```bash
psql $DATABASE_URL -f prisma/migrations/20260813124003_init/migration.sql
```

Notes and tips
- If you plan to enable Row Level Security (RLS), set the session tenant using:
  SELECT set_config('app.current_tenant', '<TENANT_UUID>', true);
  before running tenant-scoped queries within that DB session.
- If npm is not available on your machine, install Node.js from https://nodejs.org/ or use Docker.
- To run everything with Docker, consider creating a docker-compose.yml that brings up a Postgres service and a Node container to run the migration and seed commands.

Helpful npm scripts (in package.json)
- npm run prisma:generate  -> prisma generate
- npm run prisma:migrate   -> prisma migrate dev --name init
- npm run prisma:deploy    -> prisma migrate deploy
- npm run seed             -> run the TypeScript seed.ts via ts-node

If you want, I can also scaffold a docker-compose.yml and a small helper script to run migrations/seeds inside a container.

## CI / Diagnostics

This repository includes a GitHub Actions workflow (/.github/workflows/ci.yml) that builds the Docker image, runs migrations, seeds the database, and performs diagnostic checks for GHCR package permissions.

Manual run (workflow_dispatch)

1. In GitHub navigate to Actions → CI → Run workflow.
2. The workflow exposes an input `skip-ghcr-cleanup` (default: `false`). Set it to `true` to skip deletion of the temporary GHCR permission-test image (useful while debugging).

Example: run with cleanup disabled

- In the web UI: set "skip-ghcr-cleanup" to "true" before clicking "Run workflow".
- Via the REST API (curl):

  curl -X POST \
    -H "Accept: application/vnd.github+json" \
    -H "Authorization: Bearer ${GITHUB_TOKEN}" \
    https://api.github.com/repos/<owner>/<repo>/actions/workflows/ci.yml/dispatches \
    -d '{"ref":"main","inputs":{"skip-ghcr-cleanup":"true"}}'

Notes
- The CI workflow performs a small GHCR push test to validate that the workflow's token can write packages. The workflow will try to delete that test image afterwards unless `skip-ghcr-cleanup` is set to true.
- PRs from forks run with a read-only GITHUB_TOKEN and the push test will fail in that scenario — this is expected. Run diagnostic-enabled workflows from branches in the main repository to validate package permissions.

Repository settings required for GITHUB_TOKEN package writes
- In the repository settings (on GitHub web): Settings → Actions → General → Workflow permissions, set "Read and write permissions" to allow the workflow's GITHUB_TOKEN to perform package pushes and deletes. Without this, the workflow's GHCR push/delete steps will fail with permission errors.
- If the repository is in an organization, also check Organization settings: Organization → Settings → Actions (or Policies) to ensure organization-level policies don't block Actions from writing packages. An organization may restrict package writes or require specific security controls.
- Note about forks and PRs: workflows triggered by pull requests from forked repositories run with a read-only GITHUB_TOKEN for security. To validate GHCR package writes, run the workflow from a branch in the main repository or use a repository secret PAT as an alternative (see README section on CI for details).

If you need, I can also add a short checklist to the README showing the exact UI path with screenshots (or the equivalent REST API calls) to verify and change these settings.

Optional: Use a PAT (Personal Access Token) for GHCR instead of GITHUB_TOKEN

If your organizational policies prevent GITHUB_TOKEN from writing packages, or you prefer an explicit credential, create a Personal Access Token (PAT) with the following scopes:
- write:packages (required)
- read:packages (recommended)
- repo (only if you need access to private repos)

Steps:
1. Create a PAT:
   - Visit https://github.com/settings/tokens (or your org's PAT settings)
   - Generate a new token with the scopes above and copy the token value.
2. Add the PAT to your repository secrets:
   - Repository → Settings → Secrets and variables → Actions → New repository secret
   - Name: DOCKER_REGISTRY_PAT
   - Value: <your PAT token>
3. The CI workflow will automatically use DOCKER_REGISTRY_PAT when present to authenticate to GHCR (it prefers the PAT over GITHUB_TOKEN). No workflow change is necessary; the workflow includes an optional login step that reads this secret.

Notes
- A PAT is more powerful than the GITHUB_TOKEN and should be stored securely as a repository secret. Rotate the token periodically and restrict its scopes to the minimum needed.


# mgn
