# Propstreet GitHub Workflows

Reusable GitHub Actions workflows for the Propstreet organization.

## Available Workflows

### Devin Integration

Automated issue investigation using [Devin](https://devin.ai).

| Workflow | Description |
|----------|-------------|
| `devin-trigger.yml` | Triggers Devin investigation on issues via label or `/devin` command |
| `devin-monitor.yml` | Polls active sessions every 10 minutes and updates issue labels/comments |

## Quick Setup

### 1. Add secrets to your repository

Go to **Settings → Secrets and variables → Actions** and add:

| Secret | Description |
|--------|-------------|
| `DEVIN_API_KEY` | Devin **v3 service-user token** (starts with `cog_`). Create one in Devin → Enterprise Settings → Service Users (role: Member). |
| `DEVIN_ORG_ID`  | Your Devin organization ID. Found in Devin → Settings → Organization. Used in every v3 URL path. |

Optional variables (Settings → Secrets and variables → Actions → Variables):

| Variable | Description |
|----------|-------------|
| `DEVIN_ISSUE_PLAYBOOK_ID` | Devin playbook ID for consistent behavior |

### 2. Create the `devin` label

Go to **Issues → Labels → New label** and create a label named `devin`.

The following state labels will be created automatically:
- `devin-investigating` - Devin is actively working
- `devin-needs-info` - Devin is blocked and needs input
- `devin-stuck` - Session running for >2 hours
- `devin-done` - Investigation completed
- `devin-pr-created` - Devin created a PR
- `devin-failed` - Session failed/expired
- `devin-cancelled` - Session was cancelled

### 3. Copy the caller workflows

Copy the files from `caller-templates/` to your repo's `.github/workflows/` directory:

```bash
# From your repo root
mkdir -p .github/workflows

# Using curl
curl -o .github/workflows/devin-trigger.yml \
  https://raw.githubusercontent.com/propstreet/github-workflows/v2/caller-templates/devin-trigger.yml

curl -o .github/workflows/devin-monitor.yml \
  https://raw.githubusercontent.com/propstreet/github-workflows/v2/caller-templates/devin-monitor.yml
```

Or copy manually from [`caller-templates/`](./caller-templates/).

### 4. Commit and push

```bash
git add .github/workflows/
git commit -m "Add Devin integration workflows"
git push
```

## Usage

### Trigger Devin

1. **Via label**: Add the `devin` label to any issue
2. **Via command**: Comment `/devin` on any issue
3. **With context**: Comment `/devin please focus on the error handling`

### Interact with active sessions

- `/devin <message>` - Add context to the current session
- `/devin cancel` - Stop the current session

### Monitor sessions

The monitor workflow runs every 10 minutes and:
- Updates labels based on session status
- Posts comments on status changes (blocked, completed, PR created)
- Marks sessions as stuck after 2 hours

## Version Pinning

Always pin to a specific version in production:

```yaml
# Recommended - pin to major version
uses: propstreet/github-workflows/.github/workflows/devin-trigger.yml@v2

# More stable - pin to specific version
uses: propstreet/github-workflows/.github/workflows/devin-trigger.yml@v2.0.0

# Most stable - pin to commit SHA
uses: propstreet/github-workflows/.github/workflows/devin-trigger.yml@abc123def
```

### Major version history

| Tag | Devin API | Status |
|-----|-----------|--------|
| `v2` | v3 (`/v3/organizations/{org_id}/...`, service-user token, requires `DEVIN_ORG_ID`) | Current |
| `v1` | v1 (`/v1/...`, personal/service API key) | Frozen — Devin docs mark v1 as "will be deprecated soon" |

**Migrating from v1 → v2:** rotate `DEVIN_API_KEY` to a `cog_…` service-user token, add the `DEVIN_ORG_ID` repo secret, and bump the `@v1` reference in your caller workflow to `@v2`.

## Updating

When new versions are released:

1. Check the [releases](https://github.com/propstreet/github-workflows/releases) for breaking changes
2. Update the version in your caller workflows
3. Test on a non-critical issue first

## Troubleshooting

### "No active Devin session found"

- The session may have already completed or been cancelled
- Check the Devin dashboard for session history

### Session not starting

- Verify `DEVIN_API_KEY` is set correctly in repository secrets
- Check Actions logs for API errors
- Ensure the `devin` label exists

### Labels not updating

- The monitor runs every 10 minutes - changes aren't immediate
- Manually trigger the monitor workflow for debugging
- Check Actions logs for API failures

## Development

### Testing changes

1. Create a branch in this repo
2. Update caller workflows to reference your branch: `@your-branch`
3. Test in a non-production repository
4. Merge and tag when ready

### Releasing

```bash
# Tag a new version
git tag v1.0.0
git push origin v1.0.0

# Update the major version tag
git tag -f v1
git push -f origin v1
```
