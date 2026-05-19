# GitLab CI Integration for Refactoring Agent

> **Note**: This project is under active development and not yet ready for use.

This repository contains a GitLab CI/CD template for running the refactoring agent on GitLab merge requests.

## Quick Start

### 1. Configure CI/CD Variables

In your GitLab project, go to **Settings → CI/CD → Variables** and add:

| Variable | Description | Scope |
|----------|-------------|-------|
| `GITLAB_TOKEN` | Project access token with `api` and `write_repository` scopes | Required |
| `CS_ACCESS_TOKEN` | CodeScene API access token | Required |
| `ANTHROPIC_API_KEY` | Anthropic API key | Required (one of) |
| `OPENAI_API_KEY` | OpenAI API key | Required (one of) |
| `GOOGLE_API_KEY` | Google API key | Required (one of) |
| `OPENCODE_AUTH_JSON` | OpenCode auth JSON (for GitHub Copilot models) | Required (one of) |

**To create a GitLab project access token:**
1. Go to **Settings → Access Tokens**
2. Create token with:
   - Name: `refactoring-agent`
   - Role: `Developer` or `Maintainer`
   - Scopes: `api`, `write_repository`

### 2. Include Template and Define Jobs in `.gitlab-ci.yml`

Include the template and define manual jobs for the skills you want:

```yaml
include:
  - remote: 'https://raw.githubusercontent.com/codescene-oss/pr-refactoring-agent-gitlab/main/refactoring-agent.yml'

fix-code-health-degradations:
  extends: .refactoring-agent-base
  variables:
    COMMAND: "skill:fix-code-health-degradations"
    MODEL: "anthropic/claude-sonnet-4-20250514"
  rules:
    - if: '$CI_PIPELINE_SOURCE == "merge_request_event"'
      when: manual
      allow_failure: true
```

### 3. Use in Merge Requests

Open a merge request, go to **CI/CD → Pipelines**, and click the play button on the job you want to run. The agent will:

1. Download the refactoring agent binary
2. Create an initial MR note: "🤖 Refactoring Agent ⏳ Running..."
3. Execute the refactoring skill
4. Commit and push changes to the MR branch
5. Update the MR note with results: "✅ Completed successfully"

## Available Skills

- `skill:fix-code-health-degradations` — Fix only the Code Health regressions introduced by the MR, without touching pre-existing debt
- `skill:uplift-code-health` — Raise Code Health for selected files toward a target score, in measurable incremental steps

## Example Configurations

### Multiple Predefined Jobs

```yaml
include:
  - remote: 'https://raw.githubusercontent.com/codescene-oss/pr-refactoring-agent-gitlab/main/refactoring-agent.yml'

fix-code-health-degradations:
  extends: .refactoring-agent-base
  variables:
    COMMAND: "skill:fix-code-health-degradations"
    MODEL: "github-copilot/gpt-4o"
  rules:
    - if: '$CI_PIPELINE_SOURCE == "merge_request_event"'
      when: manual
      allow_failure: true

uplift-code-health:
  extends: .refactoring-agent-base
  variables:
    COMMAND: "skill:uplift-code-health"
    MODEL: "github-copilot/gpt-4o"
  rules:
    - if: '$CI_PIPELINE_SOURCE == "merge_request_event"'
      when: manual
      allow_failure: true
```

### Pin to Specific Agent Version

```yaml
fix-code-health-degradations:
  extends: .refactoring-agent-base
  variables:
    COMMAND: "skill:fix-code-health-degradations"
    REFACTORING_AGENT_VERSION: "v0.1.5"
  rules:
    - if: '$CI_PIPELINE_SOURCE == "merge_request_event"'
      when: manual
      allow_failure: true
```

## Self-Hosted GitLab

This template works with self-hosted GitLab instances. The `CI_API_V4_URL` environment variable is automatically set by GitLab CI and points to your instance's API endpoint — no additional configuration needed.

## Troubleshooting

### "Failed to download binary from GitHub Releases"

**Cause**: Network access to github.com is blocked, or the version doesn't exist.

**Solution**:
- Verify the GitLab runner has internet access to github.com
- Check that `REFACTORING_AGENT_VERSION` (if set) matches a valid release tag

### "PRIVATE-TOKEN: 401 Unauthorized"

**Cause**: Invalid or missing `GITLAB_TOKEN`.

**Solution**:
- Verify `GITLAB_TOKEN` is set in **Settings → CI/CD → Variables**
- Check the token has `api` and `write_repository` scopes
- Regenerate the token if expired

### MR note not created

**Cause**: `GITLAB_TOKEN` doesn't have `api` scope or the token is invalid.

**Solution**:
- Verify the token has `api` scope (not just `write_repository`)
- Refactoring will continue even if note creation fails

### Changes not pushed

**Cause**: `GITLAB_TOKEN` doesn't have `write_repository` scope.

**Solution**:
- Add `write_repository` scope to the token

## Differences from GitHub Action

| Feature | GitHub Action | GitLab CI |
|---------|---------------|-----------|
| **Trigger** | Comment `/cs-agent` on PR | Click play on manual job in pipeline |
| **Setup** | Composite action (`uses:`) | Include template + define jobs |
| **Permissions** | Auto via `GITHUB_TOKEN` | Manual project access token |
| **Comment skill** | `github-pr-comment` | `gitlab-mr-comment` |

## Support

- **Issues**: https://github.com/codescene-oss/refactoring-agent/issues
- **Documentation**: https://codescene.io/docs/
