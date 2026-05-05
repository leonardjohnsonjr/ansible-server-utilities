# Cross-Platform Ansible User and Folder Management

This project demonstrates a role architecture that dispatches OS-specific task files from a single role entry point. It supports:

- Creating a folder on Linux, macOS, and Windows
- Creating a local user on Linux, macOS, and Windows
- Creating a Windows Active Directory domain user from a Windows domain controller or RSAT-enabled Windows host

## Architecture

Each role has a `tasks/main.yml` file that detects the operating system using gathered Ansible facts, then includes the correct OS-specific task file.

Example dispatch pattern:

```yaml
- name: Dispatch folder creation tasks for host OS family
  ansible.builtin.include_tasks: "{{ create_folder_task_file }}"
  vars:
    create_folder_task_file: >-
      {{ 'windows.yml' if ansible_os_family == 'Windows'
         else 'macos.yml' if ansible_system == 'Darwin'
         else 'linux.yml' }}
```

This keeps shared logic in `main.yml` while separating platform-specific implementation into:

```text
roles/<role_name>/tasks/linux.yml
roles/<role_name>/tasks/macos.yml
roles/<role_name>/tasks/windows.yml
```

## Project Tree

```text
ansible-cross-platform-user-folder/
├── ansible.cfg
├── group_vars/
│   └── all/
│       ├── main.yml
│       └── vault.example.yml
├── inventories/
│   └── inventory.yml
├── playbooks/
│   └── site.yml
├── requirements.yml
└── roles/
    ├── create_domain_user/
    │   ├── defaults/main.yml
    │   └── tasks/
    │       ├── main.yml
    │       └── windows.yml
    ├── create_folder/
    │   ├── defaults/main.yml
    │   └── tasks/
    │       ├── linux.yml
    │       ├── macos.yml
    │       ├── main.yml
    │       └── windows.yml
    └── create_local_user/
        ├── defaults/main.yml
        └── tasks/
            ├── linux.yml
            ├── macos.yml
            ├── main.yml
            └── windows.yml
```

## Requirements

Install collections:

```bash
ansible-galaxy collection install -r requirements.yml
```

Required collections:

- `ansible.windows`
- `microsoft.ad`

## Inventory

Edit `inventories/inventory.yml` with your real hosts, usernames, and connection settings.

The sample inventory includes four groups:

- `linux`
- `macos`
- `windows`
- `domain_controllers`

The first play runs local user and folder creation against Linux, macOS, and Windows hosts. The second play runs Active Directory user creation against the `domain_controllers` group.

## Variables

Edit `group_vars/all/main.yml` for folder, local user, and domain user settings.

Sensitive values should go in an encrypted vault file:

```bash
ansible-vault create group_vars/all/vault.yml
```

Use `group_vars/all/vault.example.yml` as a starting point.

## Run

Syntax check:

```bash
ansible-playbook playbooks/site.yml --syntax-check
```

Run all plays:

```bash
ansible-playbook playbooks/site.yml --ask-vault-pass
```

Run only folder and local user creation:

```bash
ansible-playbook playbooks/site.yml --limit 'linux:macos:windows' --ask-vault-pass
```

Run only domain user creation:

```bash
ansible-playbook playbooks/site.yml --limit domain_controllers --ask-vault-pass
```

## Notes

- Windows domain user creation is intentionally isolated to the `domain_controllers` group.
- macOS and Linux cannot directly create a Windows Active Directory user without calling a Windows domain controller, RSAT-enabled Windows host, or external directory API.
- Password variables should be stored with Ansible Vault.
- Windows hosts require WinRM configuration.
- Linux and macOS hosts require SSH access and privilege escalation for user and system folder creation.

## Verification Performed

The project was checked for:

- YAML parse validity
- Expected role and playbook file structure
- OS-dispatch pattern in each role
- Presence of required Windows and Active Directory collections in `requirements.yml`
- Separation of secrets into a vault example file

