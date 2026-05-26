# CodeScene MR Refactoring Agent for GitLab

> **Note**: This project is under active development and not yet ready for use.

A GitLab CI template that enables CodeScene's MR Refactoring Agent so reviewers can trigger Code Health-guided refactoring directly from merge requests.

It keeps refactoring inside the normal review flow while giving teams a consistent way to improve maintainability and prevent regressions.

## Features

- 🔍 **Automatic code health analysis** - Identifies technical debt and code smells
- 🤖 **AI-guided refactoring** - Uses state-of-the-art LLMs to suggest and apply improvements
- 📊 **CodeScene integration** - Leverages CodeScene's battle-tested code health metrics
- 🔄 **MR-driven workflow** - Trigger refactorings directly from merge request pipelines
- 🎯 **Skill-based execution** - Pre-built refactoring skills for common scenarios

## Quick Start

1. Configure your project CI/CD variables for CodeScene and at least one supported AI provider.
2. Add the template include and job definitions below to your `.gitlab-ci.yml`.
3. Run the refactoring agent by clicking the play button on the job in the merge request pipeline.

## Required: Configure CI/CD variables

In your GitLab project, go to **Settings → CI/CD → Variables** and add:

- `CS_ACCESS_TOKEN` - Get it using the option that matches your setup:
  - CodeScene Cloud PAT: create it at [Create a Personal Access Token](https://codescene.io/users/me/pat).
  - CodeScene on-prem PAT: log in to your CodeScene instance, open `Configuration`, go to `Authentication`, then create a token under `Personal Access Tokens`. You can also go directly to `https://<your-cs-host><:port>/configuration/user/token`.
- `GITLAB_TOKEN` - A project access token with `api` and `write_repository` scopes. Go to **Settings → Access Tokens** to create one with role `Developer` or `Maintainer`.

**At least one AI provider:**
- `ANTHROPIC_API_KEY` - Get from [Anthropic](https://console.anthropic.com)
- `OPENAI_API_KEY` - Get from [OpenAI](https://platform.openai.com)
- `GOOGLE_API_KEY` - Get from [Google AI Studio](https://aistudio.google.com)
- `OPENCODE_AUTH_JSON` - OpenCode auth JSON.

### Required: add the template to your `.gitlab-ci.yml`

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

Then click the play button on the job in any merge request pipeline to trigger the agent.

The template automatically:
- Downloads the refactoring agent binary
- Configures git
- Runs the refactoring
- Pushes changes to the MR branch
- Posts a note with the result

## 💡 The quality of the agent depends on the model

The refactoring quality of the agent depends heavily on the strength of the backing LLM, so use one of the strongest available models from a supported provider.

The refactoring agent supports models from Anthropic, OpenAI, and Google.

## Available Skills

The agent includes two pre-built refactoring skills:

- `skill:fix-code-health-degradations` - Fix only the Code Health regressions introduced by the MR, without touching pre-existing debt
- `skill:uplift-code-health` - Raise Code Health for selected files toward a target score, in measurable incremental steps

## Variables

| Variable | Description | Required |
|----------|-------------|----------|
| `COMMAND` | The refactoring command to execute | Yes |
| `MODEL` | AI model to use | Yes |
| `CS_ACCESS_TOKEN` | CodeScene API access token | Yes |
| `GITLAB_TOKEN` | GitLab project access token | Yes |
| `REFACTORING_AGENT_VERSION` | Version of the agent to use | No |
| `ANTHROPIC_API_KEY` | Anthropic API key | No |
| `OPENAI_API_KEY` | OpenAI API key | No |
| `GOOGLE_API_KEY` | Google API key | No |
| `OPENCODE_AUTH_JSON` | OpenCode auth JSON | No |

## Example Configurations

### Multiple predefined jobs

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

uplift-code-health:
  extends: .refactoring-agent-base
  variables:
    COMMAND: "skill:uplift-code-health"
    MODEL: "anthropic/claude-sonnet-4-20250514"
  rules:
    - if: '$CI_PIPELINE_SOURCE == "merge_request_event"'
      when: manual
      allow_failure: true
```

## Platform Support

The template supports Linux runners (amd64, aarch64).

## Self-Hosted GitLab

This template works with self-hosted GitLab instances. The `CI_API_V4_URL` environment variable is automatically set by GitLab CI and points to your instance's API endpoint — no additional configuration needed.

## How It Works

1. **Download**: The template downloads the appropriate pre-built binary for your platform
2. **Analyze**: CodeScene analyzes your code for health issues and technical debt
3. **Refactor**: The AI model generates and applies improvements based on CodeScene's guidance
4. **Commit**: Changes are automatically committed and pushed to your branch

## License

Copyright CodeScene AB. See [LICENSE](LICENSE) for terms.

## Support

- [GitHub Issues](https://github.com/codescene-oss/pr-refactoring-agent-gitlab/issues)
- [Documentation](https://codescene.com/docs)
