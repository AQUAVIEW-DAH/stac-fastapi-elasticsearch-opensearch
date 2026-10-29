# AQUAVIEW Fork Maintenance

This is AQUAVIEW's fork of [stac-fastapi-elasticsearch-opensearch](https://github.com/stac-utils/stac-fastapi-elasticsearch-opensearch) (SFEOS). It carries patches that haven't been merged upstream yet.

## How it works

- **Upstream**: `stac-utils/stac-fastapi-elasticsearch-opensearch`
- **Fork**: `AQUAVIEW-DAH/stac-fastapi-elasticsearch-opensearch`
- **Cloud Build trigger** (defined in `aquaview-monorepo/infra/shared/sfeos.tf`) watches this repo for tag pushes, builds the Docker image, and deploys to Cloud Run.

## Tag convention

```
v<upstream-version>-aquaview.<patch-number>
```

Examples: `v8.0.0-aquaview.1`, `v8.0.0-aquaview.2`

## Deploy a new version

Push a tag. Cloud Build does the rest.

```bash
git tag v8.0.0-aquaview.2
git push origin v8.0.0-aquaview.2
```

Monitor: https://console.cloud.google.com/cloud-build/builds?project=aquaview-461315

## Add a new fix

```bash
# 1. Create a fix branch from main
git checkout main
git checkout -b fix/describe-the-fix

# 2. Make changes, commit

# 3. Merge into main (this is our release branch)
git checkout main
git merge fix/describe-the-fix

# 4. Tag and push
git tag v8.0.0-aquaview.N
git push origin main
git push origin v8.0.0-aquaview.N

# 5. (Optional) Submit the fix as an upstream PR from the fix branch
#    Keep the fix branch around until upstream merges it
```

## Sync with upstream

When upstream releases a new version (e.g. v6.11.0):

```bash
# Add upstream remote if not already set
git remote add upstream https://github.com/stac-utils/stac-fastapi-elasticsearch-opensearch.git

# Fetch upstream
git fetch upstream

# Check if our patches are included in the new release
git log --oneline upstream/main | head -20

# If all our fixes are merged upstream:
git checkout main
git reset --hard upstream/main
git push origin main --force
git tag v6.11.0-aquaview.1
git push origin v6.11.0-aquaview.1

# If some fixes are NOT yet merged upstream:
git checkout main
git rebase upstream/main
# Resolve conflicts if any — our unmerged patches will be on top
git push origin main --force
git tag v6.11.0-aquaview.1
git push origin v6.11.0-aquaview.1
```

## Revert to upstream (no more patches needed)

When all fixes are merged upstream and we no longer need the fork:

1. Delete the Cloud Build trigger and fork repo connection from `infra/shared/sfeos.tf`
2. Restore `infra/shared/copy-sfeos-image.sh` to pull from `ghcr.io/stac-utils/stac-fastapi-es:<version>`
3. Run `terraform apply`

## Current patches

Synced on top of upstream `v8.0.0`.

| Commit | Description | Upstream PR |
|--------|-------------|-------------|
| `cloudbuild.yaml` | Cloud Build config for AQUAVIEW deployment | N/A (fork-only) |
| `AQUAVIEW.md` | This file | N/A (fork-only) |
| client.py | Add `centroid_geohex_grid_frequency` to `DEFAULT_AGGREGATIONS`, so the root `/aggregate` accepts it (the frontend explore map calls it with no `collections`) | To send: make the root `/aggregate` accept any aggregation the catalog can serve. Drop this patch when it lands |

### Previously carried patches (now upstream or dropped)

| Description | Upstream PR |
|-------------|-------------|
| Fix ES_API_KEY to use native api_key param | [#598](https://github.com/stac-utils/stac-fastapi-elasticsearch-opensearch/pull/598) (merged in v6.10.2) |
| Catalogs-extension v0.1.3 / v0.2.0 cherry-picks for v6.16.0 boot crash | merged upstream, in v6.17.0+ (dropped on v6.19.0 sync) |
| Stop double URL-decoding CQL2 filter on GET endpoints | [#783](https://github.com/stac-utils/stac-fastapi-elasticsearch-opensearch/pull/783) (merged in v6.17.2; dropped on v6.19.0 sync) |
| `[catalogs]` extra in the deployment Dockerfile | upstream `Dockerfile.ci.es` installs it since v7 (dropped on v8.0.0 sync) |
| `geometry_geohex_grid_frequency` aggregation (three commits) | [#705](https://github.com/stac-utils/stac-fastapi-elasticsearch-opensearch/pull/705) closed: needs a commercial ES license, no OpenSearch support, and nothing in AQUAVIEW calls it (dropped on v8.0.0 sync) |

## Files specific to this fork

- `AQUAVIEW.md` — this file
- `cloudbuild.yaml` — Cloud Build config for AQUAVIEW deployment
