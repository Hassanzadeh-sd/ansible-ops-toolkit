# Linting & CI

Drop-in configs for a clean Ansible repo. Tune line length and excludes to taste.

## `.yamllint`

```yaml
---
extends: default
rules:
  line-length:
    max: 160
    level: warning
  truthy:
    allowed-values: ["true", "false", "yes", "no"]
    check-keys: false        # avoids false positives on GitHub Actions `on:`
  comments:
    min-spaces-from-content: 1
  document-start:
    present: true
ignore: |
  .venv/
```

## `.ansible-lint`

```yaml
---
profile: production          # strictest built-in profile; drop to `moderate` while migrating
exclude_paths:
  - .venv/
  - .github/
# Keep vault files out of the linter:
# - inventory/**/vault.yml
```

The `production` profile enforces FQCN, named plays/tasks, `changed_when` on commands,
`no_log` discipline, risky-permission checks, role var prefixing, and `galaxy_info`
completeness. Start with `moderate` on a legacy codebase and ratchet up.

## `.pre-commit-config.yaml`

```yaml
---
repos:
  - repo: https://github.com/pre-commit/pre-commit-hooks
    rev: v4.6.0
    hooks:
      - id: trailing-whitespace
      - id: end-of-file-fixer
      - id: check-merge-conflict
      - id: detect-private-key
      - id: check-yaml
        args: [--allow-multiple-documents]
  - repo: https://github.com/adrienverge/yamllint
    rev: v1.35.1
    hooks:
      - id: yamllint
  - repo: https://github.com/ansible/ansible-lint
    rev: v24.2.0
    hooks:
      - id: ansible-lint
```

Install with `pip install pre-commit && pre-commit install`. It then runs on every commit;
bypass a single commit with `git commit --no-verify` when you must.

## Local commands

```bash
ansible-playbook --syntax-check site.yml     # fast structural check
yamllint .                                   # YAML style
ansible-lint                                 # Ansible rules
ansible-playbook --check --diff site.yml     # dry run against real hosts
```

## CI (GitHub Actions)

```yaml
---
name: lint
"on":
  push:
  pull_request:
jobs:
  lint:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-python@v5
        with:
          python-version: "3.12"
      - name: Install linters
        run: pip install ansible-lint yamllint
      - name: yamllint
        run: yamllint .
      - name: ansible-lint
        run: ansible-lint
```

A provider-agnostic pipeline is just two ordered stages — **syntax/style** (yamllint +
`--syntax-check`) then **ansible-lint** — gating merges. Add a staging `--check` run on a
canary host if you have one.

## Pinning collections

`requirements.yml` (pin to your `ansible-core` line so CI and dev match):

```yaml
---
collections:
  - name: community.general
    version: ">=8.0.0,<9.0.0"
  - name: ansible.posix
    version: ">=1.5.0,<2.0.0"
```

Install with `ansible-galaxy collection install -r requirements.yml`. Pin deliberately:
newer collection majors often require a newer `ansible-core` than your control node ships.
