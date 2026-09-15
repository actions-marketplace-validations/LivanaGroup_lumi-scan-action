# Lumi Accessibility Scan - GitHub Action

Scan any URL for WCAG 2.2 accessibility issues using [Lumi](https://lumi.livana.io) and fail your CI pipeline if the score drops below your threshold.

Lumi runs five scanning engines in parallel (axe-core, HTML_CodeSniffer, keyboard tests, 58 proprietary checks, and AI visual analysis) and returns a single score with a prioritised issue breakdown.

## Quick start

```yaml
name: Accessibility
on: [push, pull_request]

jobs:
  scan:
    runs-on: ubuntu-latest
    steps:
      - uses: LivanaGroup/lumi-scan-action@v1
        with:
          api-key: ${{ secrets.LUMI_API_KEY }}
          url: https://your-site.com
          fail-below: 70
```

## Inputs

| Input | Required | Default | Description |
|-------|----------|---------|-------------|
| `api-key` | Yes | | Your Lumi API key. Store as a [repository secret](https://docs.github.com/en/actions/security-guides/using-secrets-in-github-actions). |
| `url` | Yes | | The URL to scan. |
| `fail-below` | No | `0` | Fail the workflow if the score is below this threshold (0-100). |
| `wait-timeout` | No | `300` | Maximum seconds to wait for scan results. |
| `api-url` | No | `https://lumi.livana.io` | Lumi API base URL. |

## Outputs

| Output | Description |
|--------|-------------|
| `score` | Accessibility score (0-100). |
| `total-issues` | Total number of issues found. |
| `critical-count` | Number of critical issues. |
| `serious-count` | Number of serious issues. |
| `scan-id` | Scan ID for the Lumi dashboard. |
| `report-url` | Direct link to the full scan report. |
| `pass` | Whether the scan passed the threshold (`true`/`false`). |

## Examples

### Scan a staging deploy

```yaml
- uses: LivanaGroup/lumi-scan-action@v1
  with:
    api-key: ${{ secrets.LUMI_API_KEY }}
    url: ${{ steps.deploy.outputs.url }}
    fail-below: 70
```

### Scan multiple pages

```yaml
strategy:
  matrix:
    page: ["/", "/pricing", "/blog", "/contact"]
steps:
  - uses: LivanaGroup/lumi-scan-action@v1
    with:
      api-key: ${{ secrets.LUMI_API_KEY }}
      url: https://your-site.com${{ matrix.page }}
      fail-below: 60
```

### Use outputs in later steps

```yaml
- uses: LivanaGroup/lumi-scan-action@v1
  id: lumi
  with:
    api-key: ${{ secrets.LUMI_API_KEY }}
    url: https://your-site.com

- name: Comment on PR
  if: github.event_name == 'pull_request'
  uses: actions/github-script@v7
  with:
    script: |
      github.rest.issues.createComment({
        issue_number: context.issue.number,
        owner: context.repo.owner,
        repo: context.repo.repo,
        body: `### Accessibility scan\nScore: **${{ steps.lumi.outputs.score }}/100** | Issues: ${{ steps.lumi.outputs.total-issues }} | [Full report](${{ steps.lumi.outputs.report-url }})`
      })
```

### Block merges below a threshold

```yaml
name: Accessibility gate
on:
  pull_request:
    branches: [main]

jobs:
  accessibility:
    runs-on: ubuntu-latest
    steps:
      - uses: LivanaGroup/lumi-scan-action@v1
        with:
          api-key: ${{ secrets.LUMI_API_KEY }}
          url: https://staging.your-site.com
          fail-below: 70
```

Then set this workflow as a [required status check](https://docs.github.com/en/repositories/configuring-branches-and-merges-in-your-repository/managing-a-branch-protection-rule) on your `main` branch.

## Getting your API key

1. Sign in to [Lumi](https://lumi.livana.io)
2. Go to **Settings > API Keys**
3. Create a key with the `scan:trigger` scope
4. Add it as a repository secret named `LUMI_API_KEY`

## Job summary

The action writes a summary table to the GitHub Actions job summary, visible on the workflow run page:

| Metric | Value |
|--------|-------|
| **Score** | 82/100 |
| **Issues** | 14 (2 critical, 5 serious) |
| **Threshold** | 70 |
| **Result** | Passed |

## Requirements

- A Lumi account on any paid plan (Solo, Pro, or Agency)
- An API key with `scan:trigger` scope
- The scanned URL must be publicly accessible (or accessible from GitHub Actions runners)

## Support

- [Lumi documentation](https://lumi.livana.io/help)
- [Contact Livana](https://livana.io/contact)

## Licence

MIT
