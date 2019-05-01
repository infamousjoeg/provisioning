# Provisioning

An Ansible role to ease the task of onboarding accounts into CyberArk's Core Privileged Access Security (PAS) using CyberArk's Management API.

## Requirements

Ansible >= v2.5

## Role Variables

### prov_api_base_url

The base URL of the CyberArk Web Services Management API.

```yaml
prov_api_base_url: https://components.cyberarkdemo.example
```

### prov_validate_certs

Boolean value to validate SSL

```yaml
prov_validate_certs: <yes/no>
```

### prov_username

API authorized username

```yaml
prov_username: <api_username>
```

### prov_password

API authorized username's password

```yaml
prov_password: <api_password>
```

### prov_acct_name

Credential object's unique "Name" value from PVWA

```yaml
prov_acct_name: <objectName>
```

### prov_acct_address

Credential object's address value from PVWA

```yaml
prov_acct_address: <address>
```

### prov_acct_username

Credential object's username value from PVWA

```yaml
prov_acct_username: <username>
```

### prov_acct_password

Credential object's password value from PVWA

```yaml
prov_acct_password: <password>
```

#### Example Secure Method to Randomize Password

```yaml
- name: Set Fact with Randomized Password for User
  set_fact:
    prov_acct_password: "{{ lookup('password', '/dev/null length=15 chars=ascii_letters') }}"
  no_log: yes
```

## Example Playbook

Including an example of how to use your role (for instance, with variables passed in as parameters) is always nice for users too:

```yaml
- hosts: localhost
  roles:
    - { role: cyberark.provisioning }
```

## License

MIT
