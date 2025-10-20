# tech stack examples

complete version-and-release workflow templates for different tech stacks.

## available templates

- **nodejs/** - node.js / typescript / npm projects
- **python/** - python / pyproject.toml / pip projects
- **rust/** - rust / cargo projects
- **go/** - go / modules projects
- **java/** - java / maven projects
- **dotnet/** - c# / .net projects
- **php/** - php / composer projects

## usage

1. choose your tech stack folder
2. copy `version-and-release.yml` to `.github/workflows/`
3. adjust project-specific paths if needed
4. create a `CHANGELOG.md` with unreleased section

## changelog format

all templates expect a changelog file with this format:

```markdown
# changelog

## [unreleased]

<!-- new entries will be inserted below this line -->

## [1.0.0] - 2025-01-01

### changed
- initial release
```

## version file locations

each tech stack reads version from different files:

| tech stack | version file |
|-----------|--------------|
| node.js | `package.json` |
| python | `pyproject.toml` + `src/__init__.py` |
| rust | `Cargo.toml` |
| go | git tags (fallback to 0.0.0) |
| java | `pom.xml` |
| .net | `*.csproj` |
| php | `composer.json` |

## customization

### change version file location

edit the "get current version" step in your workflow:

```yaml
- name: get current version
  id: current_version
  run: |
    CURRENT_VERSION=$(your-custom-command-here)
    echo "version=$CURRENT_VERSION" >> $GITHUB_OUTPUT
```

### change version update logic

edit the "update version in source" step:

```yaml
- name: update version in source
  run: |
    NEW_VERSION="${{ steps.new_version.outputs.version }}"
    # your custom update logic here
```

### add custom build steps

insert after version update:

```yaml
- name: build project
  run: |
    # your build commands
```

## requirements

- changelog.md in project root
- version defined in appropriate file for your tech stack
- github token (auto-provided)
- write permissions in workflow

## testing

test manually via github actions tab:
1. go to actions → version and release
2. click "run workflow"
3. select version bump type
4. click "run workflow"

## notes

- all workflows skip when `[skip ci]` is in commit message
- all workflows skip for changes to .md, docs/, examples/
- all workflows use conventional commits for auto-detection
- all workflows create git tags with `v` prefix (v1.0.0)
