[README.md](https://github.com/user-attachments/files/27096452/README.md)
# 🧠 Claude GitHub Actions — Maximal AI Integration

> Full-context AI-powered code review, commit analysis, and repository intelligence using Claude Opus 4.6 with extended thinking.

---

## 📦 What's Included

| Workflow | Trigger | Purpose |
|---|---|---|
| `claude-pr-review.yml` | PR open/update + `@claude` mentions | Deep code review on every PR |
| `claude-commit-analysis.yml` | Every push to main/feature branches | Commit quality score, security scan, auto-changelog |
| `claude-repo-intelligence.yml` | Weekly (Monday 9am) + manual | Full repo audit — security, architecture, performance, docs |

---

## 🚀 Setup (3 steps)

### 1. Add your Anthropic API key

```
GitHub Repo → Settings → Secrets and Variables → Actions → New repository secret

Name:  ANTHROPIC_API_KEY
Value: sk-ant-...
```

### 2. Copy workflows to your repo

```bash
cp -r .github/ /your-repo/.github/
```

### 3. Enable workflow permissions

```
GitHub Repo → Settings → Actions → General → Workflow permissions
→ ✅ Read and write permissions
→ ✅ Allow GitHub Actions to create and approve pull requests
```

That's it. Every PR and push now gets full Claude analysis automatically.

---

## 🔧 Workflow Details

### `claude-pr-review.yml` — PR Review

**Triggers:** PR opened, updated, re-opened, or any comment containing `@claude`

**What Claude sees:**
- Full diff (up to 180k chars)
- PR title, description, labels
- All commit messages in the PR
- Previous review comments (for context on re-runs)

**Output:** Posted as a PR comment with:
- 🔴 Critical Issues (blockers)
- 🟡 Warnings
- 🟢 Suggestions
- ✅ What's Done Well
- Final verdict: APPROVE / REQUEST CHANGES / COMMENT

**Ask Claude directly in a PR comment:**
```
@claude What are the security implications of the changes in auth.py?
@claude Can you explain the architectural decision made in this PR?
@claude Write unit tests for the new functions added here
```

---

### `claude-commit-analysis.yml` — Commit Intelligence

**Triggers:** Every push to `main`, `master`, `release/**`, `feat/**`, `fix/**`

**What Claude sees:**
- Full commit diff
- Last 20 commits for pattern context
- Repository file tree
- Author and commit message

**Output:**
- Commit quality score (1–10)
- Impact assessment
- Security scan (flags hardcoded secrets, unsafe patterns, etc.)
- Performance notes
- One-line changelog entry (written to `CHANGELOG.md` automatically)
- 3 follow-up suggestions

**Auto-updates `CHANGELOG.md`** with conventional commit entries on every push.

---

### `claude-repo-intelligence.yml` — Deep Repository Scan

**Triggers:** Every Monday at 9am UTC, or manually via Actions tab

**Scope options (manual trigger):**
| Scope | What it does |
|---|---|
| `full` | Complete audit — architecture, security, performance, docs, deps |
| `security` | Deep security-only scan with severity rankings |
| `architecture` | Design patterns, coupling, refactor recommendations |
| `performance` | Algorithmic complexity, DB patterns, memory leaks |
| `docs` | Documentation gaps and improvement plan |

**What Claude sees:**
- All source files packed into context (up to 150k chars)
- 30-day git history
- Contributor activity
- All dependency files (package.json, requirements.txt, etc.)

**Output:** Creates a labeled GitHub Issue with full findings.

**Run manually anytime:**
```
GitHub Repo → Actions → Claude AI — Repo Intelligence → Run workflow → Choose scope
```

---

## 🧪 Model & Capabilities Used

All workflows use **Claude claude-opus-4-5** — the most capable model.

| Feature | PR Review | Commit | Deep Scan |
|---|---|---|---|
| Model | claude-opus-4-5 | claude-opus-4-5 | claude-opus-4-5 |
| Extended Thinking | ✅ 8k budget | ❌ | ✅ 16k budget |
| Max context fed | ~180k chars | ~120k chars | ~150k chars |
| Max output tokens | 4,096 | 2,048 | 8,192 |

---

## 💰 Estimated Token Usage

| Workflow | Est. Input Tokens | Est. Output Tokens | Approx. Cost* |
|---|---|---|---|
| PR Review (small PR) | ~8,000 | ~1,500 | ~$0.10 |
| PR Review (large PR) | ~40,000 | ~2,500 | ~$0.50 |
| Commit Analysis | ~15,000 | ~800 | ~$0.18 |
| Deep Repo Scan | ~80,000 | ~3,000 | ~$1.00 |

*Estimates based on Claude Opus 4.6 pricing. Check https://anthropic.com/pricing for current rates.

---

## ⚙️ Customization

### Limit which branches trigger commit analysis
Edit `claude-commit-analysis.yml`:
```yaml
on:
  push:
    branches:
      - main          # Only main branch
```

### Change deep scan schedule
Edit `claude-repo-intelligence.yml`:
```yaml
schedule:
  - cron: '0 9 * * 1'    # Mon 9am UTC
  # - cron: '0 9 * * *'  # Daily
  # - cron: '0 9 1 * *'  # Monthly
```

### Adjust context window size
In each workflow's Node.js script, change the `.slice()` limit:
```javascript
const diff = fs.readFileSync('/tmp/full_diff.txt', 'utf8').slice(0, 180000); // Adjust this
```

### Skip CI on bot commits
The CHANGELOG auto-update uses `[skip ci]` in the commit message to prevent loops. This works with most CI systems automatically.

---

## 🔒 Security Notes

- `ANTHROPIC_API_KEY` is stored as a GitHub Secret — never hardcoded
- Workflows only have `contents: write` and `pull-requests: write` permissions
- No code is executed — Claude only reads and comments
- Diffs are processed in the runner, not sent to any third-party except Anthropic API

---

## 🐛 Troubleshooting

**Claude isn't commenting on PRs**
- Check `ANTHROPIC_API_KEY` is set in repo secrets
- Check Actions → Workflow permissions → Read and write enabled

**`@claude` mentions not working**
- The issue_comment trigger requires the PR to not be a draft
- Comment must be on the PR itself, not a commit

**CHANGELOG.md push fails**
- Ensure the workflow has `contents: write` permission
- Check branch protection rules — you may need to exclude the bot

**Rate limits**
- Anthropic API has rate limits on Opus. For high-volume repos, consider switching commit analysis to `claude-sonnet-4-6` to reduce costs and latency.
