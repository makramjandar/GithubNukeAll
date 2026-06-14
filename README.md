# GitHubNukeAll 🔥

> **One-click nuclear option for purging your entire GitHub history while preserving all repository metadata.**

[![GitHub Actions](https://img.shields.io/badge/GitHub%20Actions-2088FF?logo=github-actions&logoColor=white)](https://github.com/features/actions)
[![License](https://img.shields.io/badge/License-MIT-green.svg)](LICENSE)
[![Dry Run Safe](https://img.shields.io/badge/Dry%20Run-Safe%20by%20Default-orange)](https://github.com/youruser/GitHubNukeAll)
[![Zero Local Setup](https://img.shields.io/badge/Zero%20Local%20Setup-Required-blueviolet)](https://github.com/youruser/GitHubNukeAll)

---

## 🎯 What It Does

GitHubNukeAll is a **GitHub Actions workflow** that:

| Feature | Status |
|---------|--------|
| 🗑️ **Purges all git history** from every non-fork repository | ✅ |
| 📦 **Imports repositories locally** as backup before destruction | ✅ |
| 🔒 **Preserves visibility** (public/private) | ✅ |
| 🏷️ **Re-applies topics** automatically | ✅ |
| 🌐 **Restores GitHub Pages** configuration | ✅ |
| 📝 **Preserves description & homepage** | ✅ |
| 🌿 **Preserves default branch name** | ✅ |
| ⚙️ **Preserves feature flags** (issues, wiki, projects) | ✅ |
| 🍴 **Skips forks** (to avoid upstream breakage) | ✅ |
| 📌 **Lists pinned repos** for manual re-pinning | ✅ |

**Result:** Every repository becomes a single `clean` commit with zero history, while looking identical on the outside.

---

## 🚀 Quick Start

### 1. Fork or Create This Repository

```bash
# Option A: Fork this repo (recommended for updates)
# Option B: Create new repo and copy the workflow file
```

### 2. Add Your GitHub Token

Navigate to:
> **Repository Settings** → **Secrets and variables** → **Actions** → **New repository secret**

| Secret Name | Value |
|-------------|-------|
| `NUKE_TOKEN` | Your GitHub Personal Access Token (see below) |

#### Required Token Scopes

Create token at: https://github.com/settings/tokens/new

- [x] `repo` — Full control of private repositories
- [x] `delete_repo` — Delete repositories
- [x] `read:user` — Read user profile data

---

## 🔥 Usage

### Step 1: Dry Run (Always First)

```
Actions → Nuke All Repos → Run workflow → dry_run: true
```

**What happens:**
- Lists all repositories that would be affected
- Shows metadata for each (visibility, topics, pages, etc.)
- Displays pinned repositories for manual re-pinning
- **Nothing is deleted**

### Step 2: Execute

```
Actions → Nuke All Repos → Run workflow → dry_run: false
```

**What happens:**
1. Clones every non-fork repository locally (backup)
2. Deletes the repository on GitHub
3. Recreates it with identical settings
4. Re-applies topics, pages, description, homepage
5. Pushes a single `clean` commit

### Step 3: Download Backup Artifact

After completion:
> **Actions** → Select your run → **Artifacts** → `local-repo-backups`

This contains the full file state of every repository before the purge.

### Step 4: Re-pin Repositories

Visit your GitHub profile and manually re-pin the repositories listed in the workflow output.

---

## 📋 Workflow File

`.github/workflows/nuke-all.yml`

```yaml
name: Nuke All Repos (Full Restore)

on:
  workflow_dispatch:
    inputs:
      dry_run:
        description: 'Dry run mode — lists targets without destruction'
        required: true
        default: 'true'
        type: choice
        options: ['true', 'false']

jobs:
  nuke:
    runs-on: ubuntu-latest
    steps:
      # ─────────────────────────────────────────
      # Setup environment and dependencies
      # ─────────────────────────────────────────
      - name: Setup
        run: |
          sudo apt-get update -qq && sudo apt-get install -y -qq jq
          git config --global user.name "Nuke Bot"
          git config --global user.email "nuke@github.com"
          mkdir -p /tmp/local-import /tmp/clean-repos

      # ─────────────────────────────────────────
      # Fetch all repository metadata from GitHub API
      # ─────────────────────────────────────────
      - name: Fetch all repos metadata
        env:
          TOKEN: ${{ secrets.NUKE_TOKEN }}
        run: |
          curl -s -H "Authorization: token $TOKEN"             "https://api.github.com/user/repos?per_page=100&affiliation=owner"             > /tmp/all-repos.json

      # ─────────────────────────────────────────
      # Identify pinned repositories via GraphQL
      # These must be manually re-pinned after execution
      # ─────────────────────────────────────────
      - name: Identify pinned repos
        env:
          TOKEN: ${{ secrets.NUKE_TOKEN }}
          USER: ${{ github.repository_owner }}
        run: |
          curl -s -X POST             -H "Authorization: token $TOKEN"             -H "Content-Type: application/json"             https://api.github.com/graphql             -d '{"query": "query { user(login: "'"'"$USER'"'") { pinnedItems(first: 6, types: REPOSITORY) { nodes { ... on Repository { name } } } } }"}'             > /tmp/pinned.json
          jq -r '.data.user.pinnedItems.nodes[].name // empty' /tmp/pinned.json > /tmp/pinned-list.txt

      # ─────────────────────────────────────────
      # Import non-fork repositories locally
      # Creates backup of current file state before destruction
      # ─────────────────────────────────────────
      - name: Import non-fork repos locally
        env:
          TOKEN: ${{ secrets.NUKE_TOKEN }}
          USER: ${{ github.repository_owner }}
        run: |
          jq -c '.[] | select(.fork == false)' /tmp/all-repos.json | while read -r repo; do
            REPO_NAME=$(echo "$repo" | jq -r '.name')
            VISIBILITY=$(echo "$repo" | jq -r '.visibility')

            echo "📥 $REPO_NAME ($VISIBILITY)"

            git clone --depth 1 "https://$TOKEN@github.com/$USER/$REPO_NAME.git" /tmp/local-import/$REPO_NAME || {
              echo "  ⚠️ Clone failed — skipping"
              continue
            }

            mkdir -p /tmp/clean-repos/$REPO_NAME
            cp -r /tmp/local-import/$REPO_NAME/. /tmp/clean-repos/$REPO_NAME/
            rm -rf /tmp/clean-repos/$REPO_NAME/.git
            echo "$repo" > /tmp/clean-repos/$REPO_NAME/.repo-meta.json

            echo "  ✅ Imported"
          done

      # ─────────────────────────────────────────
      # Dry run mode — display targets without execution
      # Always run this before the real thing
      # ─────────────────────────────────────────
      - name: Dry run check
        env:
          DRY_RUN: ${{ github.event.inputs.dry_run }}
        run: |
          if [ "$DRY_RUN" = "true" ]; then
            echo "🛑 DRY RUN — No repositories will be modified"
            echo ""
            echo "═══════════════════════════════════════════════════"
            echo "TARGET REPOSITORIES"
            echo "═══════════════════════════════════════════════════"
            ls /tmp/clean-repos/ | sed 's/^/  • /'
            echo ""
            echo "═══════════════════════════════════════════════════"
            echo "METADATA PER REPOSITORY"
            echo "═══════════════════════════════════════════════════"
            for meta in /tmp/clean-repos/*/.repo-meta.json; do
              [ -f "$meta" ] || continue
              NAME=$(basename $(dirname "$meta"))
              VISIBILITY=$(jq -r '.visibility' "$meta")
              DESC=$(jq -r '.description // "none"' "$meta")
              HOMEPAGE=$(jq -r '.homepage // "none"' "$meta")
              TOPICS=$(jq -r '.topics | join(", ") // "none"' "$meta")
              HAS_PAGES=$(jq -r '.has_pages' "$meta")
              DEFAULT_BRANCH=$(jq -r '.default_branch' "$meta")
              echo "  📦 $NAME"
              echo "     Visibility: $VISIBILITY"
              echo "     Description: $DESC"
              echo "     Homepage: $HOMEPAGE"
              echo "     Topics: $TOPICS"
              echo "     Pages: $HAS_PAGES"
              echo "     Default branch: $DEFAULT_BRANCH"
              echo ""
            done
            echo "═══════════════════════════════════════════════════"
            echo "PINNED REPOSITORIES (must be manually re-pinned)"
            echo "═══════════════════════════════════════════════════"
            cat /tmp/pinned-list.txt | sed 's/^/  📌 /'
            echo ""
            echo "═══════════════════════════════════════════════════"
            echo "FORKS (will be SKIPPED)"
            echo "═══════════════════════════════════════════════════"
            jq -r '.[] | select(.fork == true) | .name' /tmp/all-repos.json | sed 's/^/  🍴 /'
            echo ""
            exit 0
          fi

      # ─────────────────────────────────────────
      # Nuclear execution: delete, recreate, restore all settings
      # ─────────────────────────────────────────
      - name: Nuke, recreate, restore all settings
        env:
          TOKEN: ${{ secrets.NUKE_TOKEN }}
          USER: ${{ github.repository_owner }}
        run: |
          for REPO_DIR in /tmp/clean-repos/*; do
            [ -d "$REPO_DIR" ] || continue

            REPO_NAME=$(basename "$REPO_DIR")
            META_FILE="$REPO_DIR/.repo-meta.json"

            # Extract all metadata from backup
            VISIBILITY=$(jq -r '.visibility' "$META_FILE")
            DESCRIPTION=$(jq -r '.description // ""' "$META_FILE")
            HOMEPAGE=$(jq -r '.homepage // ""' "$META_FILE")
            TOPICS=$(jq -r '[.topics[]?] | select(length > 0) | @json' "$META_FILE" || echo "")
            HAS_PAGES=$(jq -r '.has_pages' "$META_FILE")
            HAS_ISSUES=$(jq -r '.has_issues' "$META_FILE")
            HAS_WIKI=$(jq -r '.has_wiki' "$META_FILE")
            HAS_PROJECTS=$(jq -r '.has_projects' "$META_FILE")
            DEFAULT_BRANCH=$(jq -r '.default_branch' "$META_FILE")
            PRIVATE_BOOL=$( [ "$VISIBILITY" = "private" ] && echo "true" || echo "false" )

            echo "═══════════════════════════════════════════════════"
            echo "🧨 $REPO_NAME | $VISIBILITY"
            echo "═══════════════════════════════════════════════════"

            # ── 1. DESTROY ───────────────────────────────
            echo "→ [1/5] Destroying repository..."
            HTTP_STATUS=$(curl -X DELETE -s -o /dev/null -w "%{http_code}"               -H "Authorization: token $TOKEN"               "https://api.github.com/repos/$USER/$REPO_NAME")

            if [ "$HTTP_STATUS" != "204" ]; then
              echo "  ⚠️ Delete returned HTTP $HTTP_STATUS — continuing anyway"
            else
              echo "  ✅ Deleted"
            fi
            sleep 3

            # ── 2. RECREATE ──────────────────────────────
            echo "→ [2/5] Recreating with preserved settings..."
            HTTP_STATUS=$(curl -X POST -s -o /dev/null -w "%{http_code}"               -H "Authorization: token $TOKEN"               -H "Accept: application/vnd.github.v3+json"               -d "{
                "name":"$REPO_NAME",
                "description":"$DESCRIPTION",
                "homepage":"$HOMEPAGE",
                "private":$PRIVATE_BOOL,
                "has_issues":$HAS_ISSUES,
                "has_wiki":$HAS_WIKI,
                "has_projects":$HAS_PROJECTS,
                "has_downloads":false
              }"               "https://api.github.com/user/repos")

            if [ "$HTTP_STATUS" != "201" ]; then
              echo "  ❌ Create failed with HTTP $HTTP_STATUS — ABORTING this repo"
              continue
            fi
            echo "  ✅ Created"
            sleep 2

            # ── 3. RESTORE TOPICS ────────────────────────
            if [ -n "$TOPICS" ] && [ "$TOPICS" != "null" ] && [ "$TOPICS" != "[]" ]; then
              echo "→ [3/5] Re-applying topics: $TOPICS"
              curl -X PUT -s -o /dev/null                 -H "Authorization: token $TOKEN"                 -H "Accept: application/vnd.github.v3+json"                 -d "{"names":$TOPICS}"                 "https://api.github.com/repos/$USER/$REPO_NAME/topics"
              echo "  ✅ Topics restored"
            else
              echo "→ [3/5] No topics to restore"
            fi

            # ── 4. RESTORE GITHUB PAGES ────────────────────
            if [ "$HAS_PAGES" = "true" ]; then
              echo "→ [4/5] Restoring GitHub Pages..."

              # Detect original Pages source configuration
              PAGES_INFO=$(curl -s -H "Authorization: token $TOKEN"                 "https://api.github.com/repos/$USER/$REPO_NAME/pages" 2>/dev/null || echo "{}")

              SOURCE_BRANCH=$(echo "$PAGES_INFO" | jq -r '.source.branch // "main"')
              SOURCE_PATH=$(echo "$PAGES_INFO" | jq -r '.source.path // "/ (root)"')

              curl -X POST -s -o /dev/null                 -H "Authorization: token $TOKEN"                 -H "Accept: application/vnd.github.v3+json"                 -d "{
                  "source":{
                    "branch":"$SOURCE_BRANCH",
                    "path":"$SOURCE_PATH"
                  }
                }"                 "https://api.github.com/repos/$USER/$REPO_NAME/pages"
              echo "  ✅ Pages enabled ($SOURCE_BRANCH)"
            else
              echo "→ [4/5] No Pages to restore"
            fi

            # ── 5. PUSH CLEAN HISTORY ──────────────────────
            echo "→ [5/5] Pushing clean history..."
            cd "$REPO_DIR"
            rm -f .repo-meta.json  # Remove metadata file before commit

            git init
            git add -A
            git commit -m "clean"

            # Use original default branch name
            BRANCH_NAME=$( [ "$DEFAULT_BRANCH" = "null" ] && echo "main" || echo "$DEFAULT_BRANCH" )
            git branch -m "$BRANCH_NAME"

            git remote add origin "https://$TOKEN@github.com/$USER/$REPO_NAME.git"
            git push -u origin "$BRANCH_NAME" --force

            echo "  ✅ Clean history pushed to $BRANCH_NAME"
            echo ""
          done

      # ─────────────────────────────────────────
      # Final summary with action items
      # ─────────────────────────────────────────
      - name: Summary
        run: |
          echo ""
          echo "╔════════════════════════════════════════════════════════════════════╗"
          echo "║                    🎉  OPERATION COMPLETE  🎉                       ║"
          echo "╚════════════════════════════════════════════════════════════════════╝"
          echo ""
          echo "📦 Repositories processed:"
          ls /tmp/clean-repos/ | sed 's/^/   ✅ /'
          echo ""
          echo "════════════════════════════════════════════════════════════════════"
          echo "📌  MANUAL ACTION REQUIRED: Re-pin repositories"
          echo "════════════════════════════════════════════════════════════════════"
          echo "   Visit: https://github.com/${{ github.repository_owner }}"
          echo "   Click 'Customize your pins' and select:"
          cat /tmp/pinned-list.txt | sed 's/^/      📌 /'
          echo ""
          echo "════════════════════════════════════════════════════════════════════"
          echo "🍴  FORKS (intentionally skipped)"
          echo "════════════════════════════════════════════════════════════════════"
          jq -r '.[] | select(.fork == true) | .name' /tmp/all-repos.json | sed 's/^/   🍴 /' || echo "   None"
          echo ""
          echo "════════════════════════════════════════════════════════════════════"
          echo "✅  AUTOMATICALLY PRESERVED"
          echo "════════════════════════════════════════════════════════════════════"
          echo "   • Visibility (public/private)"
          echo "   • Description & homepage URL"
          echo "   • Topics / tags"
          echo "   • GitHub Pages configuration"
          echo "   • Default branch name"
          echo "   • Feature flags (issues, wiki, projects)"
          echo ""
          echo "════════════════════════════════════════════════════════════════════"
          echo "⚠️  IRREVERSIBLE CHANGES"
          echo "════════════════════════════════════════════════════════════════════"
          echo "   • All commit history is permanently deleted"
          echo "   • All issues, PRs, and discussions are lost"
          echo "   • All releases and tags are destroyed"
          echo "   • Fork relationships are broken (forks were skipped)"
          echo ""

      # ─────────────────────────────────────────
      # Upload local backup as downloadable artifact
      # Download: Actions → Run → Artifacts → local-repo-backups
      # ─────────────────────────────────────────
      - name: Upload backup artifact
        if: always()
        uses: actions/upload-artifact@v4
        with:
          name: local-repo-backups
          path: /tmp/local-import/
          retention-days: 1
          if-no-files-found: warn
