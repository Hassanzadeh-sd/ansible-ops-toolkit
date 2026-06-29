# Writing conventions

Copy-pasteable idioms for production-grade Ansible. These are generic distillations of
patterns that hold up well at fleet scale.

## Role layout

```
roles/<role>/
├── defaults/main.yml      # all tunables, prefixed <role>_*
├── vars/main.yml          # internal constants only (rarely needed)
├── tasks/main.yml         # entry point; import focused sub-task files
├── handlers/main.yml      # named, FQCN handlers
├── templates/*.j2         # Jinja2 templates with ansible_managed header
├── files/                 # static files
└── meta/main.yml          # galaxy_info + dependencies
```

## A well-formed task

```yaml
- name: Deploy the service configuration
  ansible.builtin.template:
    src: service.conf.j2
    dest: /etc/service/service.conf
    owner: root
    group: root
    mode: "0644"
    validate: "service --test-config %s"   # use the target's checker when one exists
  notify: Reload service
  tags: [service, service_config]
```

Rules on display: FQCN, descriptive name, explicit `mode:`, `validate:` before write,
restart via `notify`, and hierarchical tags.

## Loops with readable output

```yaml
- name: Create application users
  ansible.builtin.user:
    name: "{{ item.name }}"
    groups: "{{ item.groups | default(omit) }}"
    state: present
  loop: "{{ app_users }}"
  loop_control:
    label: "{{ item.name }}"   # keeps -v output clean
```

## Validation-first (assert prerequisites)

```yaml
- name: Validate required per-host variables are set
  ansible.builtin.assert:
    that:
      - data_disks is defined
      - data_disks | length >= 1
    fail_msg: "data_disks must list at least one device for this host"
    quiet: true
  tags: [always]
```

## Umbrella role with per-module toggles

`tasks/main.yml`:

```yaml
- name: Configure SSH hardening
  ansible.builtin.import_tasks: ssh.yml
  when: hardening_ssh_enabled | default(true)
  tags: [hardening, hardening_ssh]

- name: Configure the firewall
  ansible.builtin.import_tasks: firewall.yml
  when: hardening_firewall_enabled | default(false)
  tags: [hardening, hardening_firewall]
```

Operators can then run `--tags hardening_ssh` or opt a host out with
`hardening_firewall_enabled: false` — no forking the role.

## Secrets

```yaml
- name: Write the API token
  ansible.builtin.copy:
    content: "{{ vault_service_api_token }}"
    dest: /etc/service/token
    owner: root
    group: root
    mode: "0600"
  no_log: true          # never print the secret, even with -v
```

Store `vault_*` values in an Ansible Vault file (`ansible-vault encrypt`), reference them
from `group_vars`, and keep the vault password out of the repo.

## Templates

```jinja2
# {{ ansible_managed }}
# Managed by the <role> role — local edits will be overwritten.
worker_processes {{ service_worker_processes }};
{% for name, value in service_options.items() %}
{% if value not in [none, ''] %}
{{ name }} {{ value }};
{% endif %}
{% endfor %}
```

## Inventory & variable hierarchy

```
inventory/
├── hosts.ini
├── group_vars/
│   ├── all/vars.yml           # global defaults + secret references
│   └── web/vars.yml           # group overrides
└── host_vars/
    └── web1.yml               # host-specific overrides
```

Precedence (later wins): role `defaults` → `group_vars/all` → `group_vars/<group>` →
`host_vars/<host>`. Keep environments (staging/production) in separate inventories.

## Rolling deploys with health checks

```yaml
- name: Roll out the web tier one host at a time
  hosts: web
  serial: 1
  tasks:
    - name: Apply the web role
      ansible.builtin.include_role:
        name: web
    - name: Verify the service answers after deploy
      ansible.builtin.uri:
        url: "http://{{ ansible_host }}:{{ web_listen_port }}/healthz"
        status_code: 200
      retries: 5
      delay: 3
      register: health
      until: health.status == 200
```
