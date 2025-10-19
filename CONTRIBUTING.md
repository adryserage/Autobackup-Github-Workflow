# contributing

contributions welcome. keep it simple.

## what to contribute

- new tech stack examples
- bug fixes in existing workflows
- documentation improvements
- optimization suggestions

## adding new tech stack

1. create folder in `examples/your-tech-stack/`
2. add `version-and-release.yml` workflow
3. update `examples/README.md` with your tech stack
4. update main `README.md` customization section

### workflow requirements

your workflow must:
- read current version from project files
- calculate semantic version bump
- update version in source files
- update changelog.md
- create git tag with v prefix
- create github release
- skip with `[skip ci]` commit message

## testing workflows

before submitting:
1. test manual workflow dispatch
2. test auto version detection from commits
3. verify changelog gets updated
4. verify git tags are created correctly
5. verify github release is created

## pull request process

1. fork the repo
2. create branch from main
3. make changes
4. test thoroughly
5. submit pr with clear description

## commit messages

use conventional commits:
- `feat:` new feature
- `fix:` bug fix
- `docs:` documentation only
- `chore:` maintenance

examples:
```
feat: add ruby/rails workflow template
fix: correct version regex in python workflow
docs: improve readme installation steps
```

## code style

workflows:
- use lowercase for names
- indent with 2 spaces
- keep steps focused on single task
- add comments for complex logic
- use descriptive step names

documentation:
- use lowercase headers
- keep sentences short
- include code examples
- show real commands

## questions

open an issue if you're unsure about anything.
