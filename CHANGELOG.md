# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

### Added
- Initial workflow with full metadata preservation
- Dry-run mode for safe preview before execution
- Automatic topic restoration via GitHub API
- GitHub Pages configuration restoration
- Default branch name preservation
- Feature flags preservation (issues, wiki, projects)
- Local backup artifact generation
- Pinned repository detection and listing
- Fork detection and exclusion
- Comprehensive console output with progress indicators

### Security
- Token-based authentication with minimal required scopes
- No secrets logged in workflow output
- Backup artifact retention limited to 1 day

## [1.0.0] - 2026-06-14

### Added
- First stable release of GitHubNukeAll
- Complete repository history purge with single clean commit
- Full metadata restoration pipeline
- GitHub Actions native execution (zero local setup)
- MIT License
- Comprehensive documentation (README, CONTRIBUTING, CODE_OF_CONDUCT)

### Features
- **Nuke Mode**: Delete and recreate repositories with clean history
- **Dry Run Mode**: Preview all targets without modification
- **Metadata Preservation**:
  - Repository visibility (public/private)
  - Description and homepage URL
  - Topics/tags
  - GitHub Pages source configuration
  - Default branch name
  - Feature flags (issues, wiki, projects, downloads)
- **Backup System**: Local clone artifact downloadable after execution
- **Safety Mechanisms**:
  - Forks automatically excluded
  - Pinned repositories listed for manual re-pinning
  - Mandatory dry-run before execution
  - Step-by-step progress logging

### Technical
- GitHub REST API v3 for repository operations
- GitHub GraphQL API for pinned repository detection
- Shallow cloning (depth 1) for performance
- `jq` for robust JSON parsing
- `curl` with HTTP status validation
- Artifact upload via `actions/upload-artifact@v4`

[Unreleased]: https://github.com/youruser/GitHubNukeAll/compare/v1.0.0...HEAD
[1.0.0]: https://github.com/youruser/GitHubNukeAll/releases/tag/v1.0.0
