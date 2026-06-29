# Best-practices checklist

A reviewable list to run any playbook or role against. Each item is something a reviewer
(human or this skill) can verify directly.

## Correctness & idempotency
- [ ] **Idempotent** — running twice produces no changes the second time. Prefer modules to
      `command`/`shell`; when shelling out, set `changed_when` / `creates` / `removes`.
- [ ] **Read-only commands** marked `changed_when: false`.
- [ ] **No state-changing commands inside conditionals** — compute facts first with
      `set_fact`, then branch on them.
- [ ] **`--check --diff` clean** — no task errors in check mode (no reads of files that only
      exist after a real run without guards).

## Validation
- [ ] **Prerequisites asserted** with `assert:` and `tags: [always]` before changes.
- [ ] **Templates validated** — `validate:` runs the target's checker (`sshd -t`, `nginx -t`,
      `visudo -cf`) before the file is put in place.

## Structure & naming
- [ ] **FQCN** on every module.
- [ ] **Every play and task named**, capitalized, explaining the *why*.
- [ ] **Role variables prefixed** `<role>_*` and defaulted in `defaults/main.yml`.
- [ ] **`meta/main.yml` complete** — `galaxy_info` with author, description, license,
      `min_ansible_version`, `platforms`.
- [ ] **Hierarchical tags** so operators can run slices (`--tags verify`).

## Handlers & ordering
- [ ] **Restarts/reloads via `notify`** + named handlers (consolidated, run once at end).
- [ ] **`meta: flush_handlers`** only where a later task depends on the restart.

## Portability
- [ ] **Multi-OS guards** (`when: ansible_os_family == ...`) instead of single-distro
      assumptions; use `package` or per-family modules.
- [ ] **`mode:` set explicitly** on every `copy`/`template`/`file`.

## Secrets & safety
- [ ] **Secrets in Vault**, never plaintext in the repo; `vault.yml.example` for structure.
- [ ] **`no_log: true`** on any task touching a secret.
- [ ] **Never overwrite silently** — show a diff and confirm before changing existing files;
      flag destructive changes (`state: absent`, prod inventory edits).

## Operations
- [ ] **Rolling changes** use `serial:` with health-check `post_tasks`/`until` retries.
- [ ] **Inventory in version control**, environments kept separate.
- [ ] **Collections & `ansible-core` pinned** with a documented reason for each pin.
- [ ] **Comment the why** for any non-obvious regex, ordering, or edge case.
- [ ] **Operator ergonomics** — clear variable names, useful `fail_msg`, worked examples in
      docs/runbooks.
