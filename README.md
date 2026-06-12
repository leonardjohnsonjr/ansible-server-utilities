# Ansible Server Utilities

This project manages server utility tasks with Ansible roles. The current primary workflow installs software packages on Linux hosts, with package lists stored next to the inventory that uses them.

The active playbook is:

```text
playbooks/site.yml
```

It currently runs:

```yaml
roles:
  - role: install_packages
```

## Project Structure

```text
ansible-server-utilities/
├── .gitignore
├── README.md
├── ansible.cfg
├── inventories/
│   ├── group_vars/
│   │   ├── all.yml
│   │   ├── debian.yml
│   │   └── redhat.yml
│   ├── inventory-sample.yml
│   └── inventory.yml
├── playbooks/
│   ├── site.yml
│   └── update_os.yml
├── requirements.yml
└── roles/
    ├── install_packages/
    │   ├── defaults/
    │   │   └── main.yml
    │   └── tasks/
    │       ├── main.yml
    │       └── linux.yml
    ├── update_os/
    │   └── tasks/
    │       ├── debian.yml
    │       ├── main.yml
    │       ├── redhat.yml
    │       └── windows.yml
    ├── create_folder/
    ├── create_local_user/
    └── create_domain_user/
```

`inventories/group_vars/` is the source of truth for inventory variables. Keeping group variables beside `inventories/inventory.yml` makes `ansible-inventory` and `ansible-playbook` resolve the same values when playbooks live in the `playbooks/` directory.

## Inventory

The inventory is configured in [ansible.cfg](ansible.cfg):

```ini
inventory = inventories/inventory.yml
roles_path = roles
```

The current inventory uses OS-family groups:

- `debian`
- `redhat`
- `macos`
- `windows`
- `domain_controllers`

Package installation currently targets Linux hosts. For example, `dell-inspiron` is in the `redhat` group, so it receives variables from:

```text
inventories/group_vars/redhat.yml
```

## Package Installation

The `install_packages` role dispatches Linux package installation from:

```text
roles/install_packages/tasks/main.yml
```

For Debian and RedHat hosts, it includes:

```text
roles/install_packages/tasks/linux.yml
```

The Linux task installs each package in `software_packages`:

```yaml
software_packages:
  - curl
  - git
  - rclone
```

Group-specific package lists live here:

```text
inventories/group_vars/debian.yml
inventories/group_vars/redhat.yml
```

The role also has fallback defaults in:

```text
roles/install_packages/defaults/main.yml
```

Those defaults are used only if `software_packages` is not defined for a host.

## Variables

Common variables live in:

```text
inventories/group_vars/all.yml
```

OS-specific package lists live in:

```text
inventories/group_vars/debian.yml
inventories/group_vars/redhat.yml
```

To confirm what a host receives:

```bash
ansible-inventory -i inventories/inventory.yml --host dell-inspiron
```

To confirm a group variable directly:

```bash
ansible -i inventories/inventory.yml redhat -m debug -a var=software_packages
```

## Run

Syntax check:

```bash
ansible-playbook -i inventories/inventory.yml playbooks/site.yml --syntax-check
```

Run package installation for RedHat hosts:

```bash
ansible-playbook -i inventories/inventory.yml playbooks/site.yml --limit redhat -v
```

Run package installation for Debian hosts:

```bash
ansible-playbook -i inventories/inventory.yml playbooks/site.yml --limit debian -v
```

Preview changes with check mode:

```bash
ansible-playbook -i inventories/inventory.yml playbooks/site.yml --limit redhat --check -v
```

## Optional Roles

The repository still contains additional roles:

- `update_os`
- `create_folder`
- `create_local_user`
- `create_domain_user`

These are not currently active in `playbooks/site.yml` unless you add them back to the playbook. Windows and Active Directory workflows require the collections listed in `requirements.yml`.

Install those collections with:

```bash
ansible-galaxy collection install -r requirements.yml
```

## Notes

- Keep active inventory variables under `inventories/group_vars/`.
- `group_vars` files at the repository root are not the preferred layout for this project.
- Linux hosts require SSH access and privilege escalation if package installation needs root permissions.
- Windows hosts require WinRM configuration.
- Sensitive values should be stored with Ansible Vault and excluded from version control.
