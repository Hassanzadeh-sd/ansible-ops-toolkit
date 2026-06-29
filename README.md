<div align="center">

# ansible-ops-toolkit

[![License: MIT](https://img.shields.io/badge/license-MIT-3da639.svg)](LICENSE)
[![Claude Code](https://img.shields.io/badge/Claude%20Code-SKILL-D97757?logo=anthropic&logoColor=white)](https://code.claude.com/docs/en/skills)
[![Skill](https://img.shields.io/badge/%2Fansible--ops-skill-2088FF?logo=ansible&logoColor=white)](skills/ansible-ops/SKILL.md)
[![Ansible Core](https://img.shields.io/badge/Ansible%20Core-2.14%2B-EE0000?logo=ansible&logoColor=white)](https://docs.ansible.com/)
[![Validate CI](https://github.com/Hassanzadeh-sd/ansible-ops-toolkit/actions/workflows/lint.yml/badge.svg)](https://github.com/Hassanzadeh-sd/ansible-ops-toolkit/actions/workflows/lint.yml)

</div>

```text
   ▛▀▀▜
   ▌◍◍▐   Claude Code · Opus 4.8
   ▙▄▄▟   ansible-ops-toolkit/
          └ SessionStart says: [ansible-ops 0.1.0]
──────────────────────────────────────────────────────────────

    _    _   _ ____ ___ ____  _     _____    ___  ____  ____
   / \  | \ | / ___|_ _| __ )| |   | ____|  / _ \|  _ \/ ___|
  / _ \ |  \| \___ \| ||  _ \| |   |  _|   | | | | |_) \___ \
 / ___ \| |\  |___) | || |_) | |___| |___  | |_| |  __/ ___) |
/_/   \_\_| \_|____/___|____/|_____|_____|  \___/|_|   |____/

──────────────────────────────────────────────────────────────
❯ /ansible-ops  —  write · review · lint · convert · debug Ansible
```

A [Claude Code](https://claude.com/claude-code) **skill** for writing, reviewing, linting,
converting, and debugging Ansible — playbooks, roles, inventory, and Jinja2 templates —
following production best practices. Ships with a small, lint-clean example role you can copy.

It encodes the conventions that hold up at fleet scale: FQCN modules, idempotency,
validation-first tasks, `notify`/handler patterns, hierarchical tags, `<role>_<knob>`
variable naming, `no_log` on secrets, multi-OS guards, and pinned collections.

## What it enforces

| Area | Rule |
|------|------|
| Modules | Fully-qualified collection names (`ansible.builtin.*`) |
| Tasks | Every play/task named; the *why*, capitalized |
| Idempotency | Modules over shell; `changed_when` / `creates` when you must shell out |
| Variables | `<role>_*`, defaulted in `defaults/main.yml` |
| Templates | `ansible_managed` header + `validate:` before write |
| Secrets | Ansible Vault + `no_log: true` |
| Files | Explicit `mode:` on `copy`/`template`/`file` |
| Safety | Never overwrite silently — diff & confirm first |
| Lint | `yamllint` + `ansible-lint` (production profile) + pre-commit |

## Layout

```
.claude-plugin/        # plugin + marketplace manifests
skills/ansible-ops/    # the skill: SKILL.md, references/, evals/
examples/nginx-role/   # tiny, lint-clean reference project (CI-verified)
.github/workflows/     # yamllint + ansible-lint on every push
```

## Install

**Plugin marketplace (recommended):**

```
/plugin marketplace add Hassanzadeh-sd/ansible-ops-toolkit
/plugin install ansible-ops-toolkit
```

**npx skills:**

```
npx skills add Hassanzadeh-sd/ansible-ops-toolkit
```

**Manual:** copy `skills/ansible-ops/` into `~/.claude/skills/` (global) or a project's
`.claude/skills/`.

Then invoke with `/ansible-ops` or just ask in natural language ("write an Ansible role for
nginx", "lint my playbooks", "convert this bash script to Ansible", "why is this play
failing").

## Try the example

```bash
cd examples/nginx-role
ansible-playbook --syntax-check site.yml
yamllint -c ../../.yamllint .
ansible-lint
```

## Credits

Built by [@Hassanzadeh-sd](https://github.com/Hassanzadeh-sd). Inspired by
[hello-ansible-skills](https://github.com/sigridjineth/hello-ansible-skills),
[ansible-skills](https://github.com/gitjfmd/ansible-skills), and
[ansible-designer](https://github.com/3A2DEV/ansible-designer). MIT licensed.
