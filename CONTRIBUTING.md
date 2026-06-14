# Contributing to GitHubNukeAll

First off, thanks for taking the time to contribute! 🎉

## Getting Started

### Prerequisites

- A GitHub account
- Basic understanding of GitHub Actions
- Familiarity with `jq` and `curl` (for debugging)

### Fork & Clone

```bash
# Fork the repository on GitHub, then:
git clone https://github.com/YOUR_USERNAME/GitHubNukeAll.git
cd GitHubNukeAll
```

## Development Workflow

### 1. Create a Branch

```bash
git checkout -b feature/your-feature-name
# or
git checkout -b fix/your-bugfix-name
```

### 2. Make Changes

- Edit `.github/workflows/nuke-all.yml` for workflow changes
- Edit `README.md` for documentation updates
- Follow existing code style and comment conventions

### 3. Test Your Changes

Since this is a GitHub Actions workflow, testing requires a real GitHub environment:

1. **Create a test repository** with dummy content
2. **Push your branch** to your fork
3. **Run the workflow** on your fork with `dry_run: true` first
4. **Verify output** matches expectations
5. **Only then** test with `dry_run: false` on disposable repos

> ⚠️ **Never test destructive operations on production repositories.**

### 4. Commit

```bash
git add .
git commit -m "feat: add support for custom commit messages"
```

Follow [Conventional Commits](https://www.conventionalcommits.org/) format:

| Prefix | Use for |
|--------|---------|
| `feat:` | New features |
| `fix:` | Bug fixes |
| `docs:` | Documentation only |
| `refactor:` | Code changes that neither fix nor add features |
| `perf:` | Performance improvements |
| `test:` | Adding or correcting tests |
| `chore:` | Maintenance tasks |

### 5. Push & Pull Request

```bash
git push origin feature/your-feature-name
```

Then open a Pull Request on GitHub with:
- Clear description of what changed and why
- Reference to any related issues
- Screenshots of dry-run output (if applicable)

## Areas for Contribution

### High Priority

- [ ] Support for GitHub Enterprise Server (custom API endpoints)
- [ ] Batch operations for organizations (not just user repos)
- [ ] Selective repository targeting (whitelist/blacklist)
- [ ] Automatic re-pinning via headless browser automation
- [ ] Preservation of GitHub Actions secrets and variables

### Nice to Have

- [ ] Pre/post-nuke hooks (custom scripts)
- [ ] Slack/Discord notifications on completion
- [ ] Support for preserving specific tags/releases
- [ ] Configurable commit message template
- [ ] Multi-language support for output messages

### Documentation

- [ ] Video tutorial walkthrough
- [ ] FAQ section for common edge cases
- [ ] Troubleshooting guide for API rate limits
- [ ] Security best practices guide

## Code Style

### YAML Workflow

- Use 2-space indentation
- Comment every logical section with `# ──` separators
- Keep step names descriptive and action-oriented
- Always quote strings in shell commands to prevent word splitting

### Shell Scripts

- Use `#!/bin/bash` explicitly
- Quote all variables: `"$VAR"`
- Use `||` for graceful failure handling
- Prefer `jq` over `grep`/`sed` for JSON parsing

Example:
```bash
# Good
REPO_NAME=$(echo "$repo" | jq -r '.name')
curl -X DELETE -s -o /dev/null -w "%{http_code}"   -H "Authorization: token $TOKEN"   "https://api.github.com/repos/$USER/$REPO_NAME"

# Bad
REPO_NAME=$(echo $repo | grep -o '"name":"[^"]*"' | cut -d'"' -f4)
curl -X DELETE https://api.github.com/repos/$USER/$REPO_NAME
```

## Reporting Issues

When reporting bugs, please include:

1. **Workflow run URL** (if public) or **run logs**
2. **Repository count** being processed
3. **Dry-run output** showing affected repos
4. **Expected vs actual behavior**
5. **GitHub token scopes** (sanitized)

### Security Issues

If you discover a security vulnerability, **do not open a public issue**.

Instead, email security concerns to the repository owner privately, or use GitHub's private vulnerability reporting feature if enabled.

## Code of Conduct

This project adheres to the following principles:

- **Be respectful** — destructive tools require responsible discussion
- **Be precise** — when dealing with data loss, ambiguity is dangerous
- **Be helpful** — guide newcomers through the dry-run process
- **Be cautious** — never encourage skipping the dry-run step

## Recognition

Contributors will be listed in the README and release notes. Significant contributions may be invited to become maintainers.

## Questions?

Open a [Discussion](https://github.com/youruser/GitHubNukeAll/discussions) for:
- Usage questions
- Feature requests
- General feedback

Open an [Issue](https://github.com/youruser/GitHubNukeAll/issues) for:
- Bug reports
- Specific technical problems
- Security concerns

---

Thank you for helping make GitHubNukeAll safer and more powerful for everyone! 🚀
