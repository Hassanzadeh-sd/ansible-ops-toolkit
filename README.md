# ansible-ops-toolkit

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
