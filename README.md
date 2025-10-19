# github workflow automation

automated backup and version management workflows for any tech stack.

## what's included

two production-ready github actions workflows:

1. **backup-branch.yml** - automatic branch backups on every push
2. **version-and-release.yml** - semantic versioning and github releases

## features

### backup workflow

- creates timestamped backup branches on every main push
- generates backup tags for easy recovery
- maintains backup index in markdown table
- auto-cleanup (keeps last 10 backups)
- skip with `[skip backup]` in commit message
- verification step ensures backup integrity

### version/release workflow

- semantic versioning (major.minor.patch)
- auto-detects version bump from commit messages
- manual trigger with version type selection
- updates changelog automatically
- creates git tags and github releases
- skips on doc-only changes

## quick start

### 1. backup workflow

copy `.github/workflows/backup-branch.yml`:

```bash
mkdir -p .github/workflows
cp backup-branch.yml .github/workflows/
```

that's it. every push to main creates a backup branch.

### 2. version/release workflow

this workflow needs adaptation to your tech stack. see customization section.

## customization by tech stack

### node.js / typescript

replace php steps (lines 42-49, 52-56, 105-110) with:

```yaml
- name: setup node
  uses: actions/setup-node@v4
  with:
    node-version: "20"

- name: get current version
  id: current_version
  run: |
    CURRENT_VERSION=$(node -p "require('./package.json').version")
    echo "version=$CURRENT_VERSION" >> $GITHUB_OUTPUT

- name: update version in source
  run: |
    NEW_VERSION="${{ steps.new_version.outputs.version }}"
    npm version $NEW_VERSION --no-git-tag-version
```

### python

```yaml
- name: setup python
  uses: actions/setup-python@v4
  with:
    python-version: "3.11"

- name: get current version
  id: current_version
  run: |
    CURRENT_VERSION=$(grep -oP '(?<=version = ")[^"]*' pyproject.toml)
    echo "version=$CURRENT_VERSION" >> $GITHUB_OUTPUT

- name: update version in source
  run: |
    NEW_VERSION="${{ steps.new_version.outputs.version }}"
    sed -i "s/version = \".*\"/version = \"$NEW_VERSION\"/" pyproject.toml
    sed -i "s/__version__ = \".*\"/__version__ = \"$NEW_VERSION\"/" src/__init__.py
```

### rust

```yaml
- name: setup rust
  uses: actions-rs/toolchain@v1
  with:
    toolchain: stable

- name: get current version
  id: current_version
  run: |
    CURRENT_VERSION=$(grep -oP '(?<=version = ")[^"]*' Cargo.toml | head -1)
    echo "version=$CURRENT_VERSION" >> $GITHUB_OUTPUT

- name: update version in source
  run: |
    NEW_VERSION="${{ steps.new_version.outputs.version }}"
    sed -i "0,/version = \".*\"/s//version = \"$NEW_VERSION\"/" Cargo.toml
```

### go

```yaml
- name: setup go
  uses: actions/setup-go@v4
  with:
    go-version: "1.21"

- name: get current version
  id: current_version
  run: |
    CURRENT_VERSION=$(git describe --tags --abbrev=0 2>/dev/null | sed 's/^v//' || echo "0.0.0")
    echo "version=$CURRENT_VERSION" >> $GITHUB_OUTPUT

- name: update version in source
  run: |
    NEW_VERSION="${{ steps.new_version.outputs.version }}"
    # update version constant in main.go or version.go
    sed -i "s/const Version = \".*\"/const Version = \"$NEW_VERSION\"/" version.go
```

### java / maven

```yaml
- name: setup java
  uses: actions/setup-java@v3
  with:
    java-version: "17"
    distribution: "temurin"

- name: get current version
  id: current_version
  run: |
    CURRENT_VERSION=$(mvn help:evaluate -Dexpression=project.version -q -DforceStdout)
    echo "version=$CURRENT_VERSION" >> $GITHUB_OUTPUT

- name: update version in source
  run: |
    NEW_VERSION="${{ steps.new_version.outputs.version }}"
    mvn versions:set -DnewVersion=$NEW_VERSION
```

### c# / .net

```yaml
- name: setup dotnet
  uses: actions/setup-dotnet@v3
  with:
    dotnet-version: "8.0"

- name: get current version
  id: current_version
  run: |
    CURRENT_VERSION=$(grep -oP '(?<=<Version>)[^<]*' YourProject.csproj)
    echo "version=$CURRENT_VERSION" >> $GITHUB_OUTPUT

- name: update version in source
  run: |
    NEW_VERSION="${{ steps.new_version.outputs.version }}"
    sed -i "s|<Version>.*</Version>|<Version>$NEW_VERSION</Version>|" YourProject.csproj
```

## commit message conventions

version workflow auto-detects bump type from commits:

- `major:` or `breaking:` → major version (1.0.0 → 2.0.0)
- `feat:` or `feature:` → minor version (1.0.0 → 1.1.0)
- anything else → patch version (1.0.0 → 1.0.1)

examples:

```
feat: add user authentication
fix: resolve memory leak in worker
major: rewrite api with breaking changes
```

## skip triggers

- backup: add `[skip backup]` to commit message
- version: add `[skip ci]` to commit message
- version: change only .md, docs/, or examples/ files

## manual triggers

both workflows support manual dispatch from github actions tab:

version workflow lets you choose bump type:

- patch (1.0.0 → 1.0.1)
- minor (1.0.0 → 1.1.0)
- major (1.0.0 → 2.0.0)

## backup recovery

restore from backup branch:

```bash
# list backups
git fetch --all
git branch -r | grep backup

# restore specific backup
git checkout backup/main-20251019-203627

# or create new branch from backup
git checkout -b recovery-branch backup/main-20251019-203627
```

restore from backup tag:

```bash
# list backup tags
git tag | grep backup

# restore from tag
git checkout backup-20251019-203627
```

## requirements

- github repository with write permissions
- GITHUB_TOKEN (auto-provided in actions)
- main branch as default

## structure

```
.github/
  workflows/
    backup-branch.yml       # backup automation
    version-and-release.yml # version/release automation
  backups/
    backup-index.md        # auto-generated backup log
```

## troubleshooting

### backup not created

- check commit message doesn't contain `[skip backup]`
- verify workflow has write permissions
- check actions tab for error logs

### version bump failed

- ensure version file exists in your project
- verify commit follows conventional format
- check changelog.md exists

### old backups not cleaning up

- workflow keeps last 10 backups by default
- modify line 102 in backup-branch.yml to change retention

## configuration

### change backup retention

edit `backup-branch.yml` line 102:

```yaml
BACKUP_BRANCHES=$(git branch -r | grep 'origin/backup/main-' | sort -r | tail -n +11)
#                                                                              ^^
#                                                                        change this number
```

keeps last N backups where N = number - 1

### change backup branch pattern

edit `backup-branch.yml` line 33:

```yaml
BRANCH_NAME="backup/main-$TIMESTAMP" # customize this pattern
```

## license

public domain - use however you want

## contributing

open an issue or pr with improvements
