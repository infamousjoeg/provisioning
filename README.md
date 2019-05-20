# Provisioning

An Ansible role to ease the task of onboarding accounts into CyberArk's Core Privileged Access Security (PAS) using CyberArk's Management API.

[Available on Ansible Galaxy](https://galaxy.ansible.com/infamousjoeg/provisioning)

## Requirements

Ansible >= v2.5

## Role Variables

| Variable | Description | Required?
| --- | --- | --- |
| `cyberark_api_base_url` | The base URL of the CyberArk Web Services Management API (Example: https://pvwa.cyberark.com) | Yes
| `cyberark_auth_type` | The authentication method to use (CyberArk/LDAP/Radius/Windows) | Yes
| `cyberark_validate_certs` | Boolean value to validate SSL (yes/no) | No
| `cyberark_username` | API authorized username | Yes
| `cyberark_password` | API authorized username's password | Yes
| `cyberark_acct_name` | Credential object's unique "Name" value from PVWA | Yes
| `cyberark_acct_address` | Credential object's address value from PVWA | No
| `cyberark_acct_username` | Credential object's username value from PVWA | No
| `cyberark_acct_password` | Credential object's password value from PVWA | No
| `cyberark_acct_platformId` | Platform to manage the credential object | Yes
| `cyberark_acct_safeName` | Safe to store the credential object within | Yes
| `cyberark_acct_secretType` | The type of secret being onboarded (password/key) | Yes
| `cyberark_acct_autoManagement` | Boolean value to enable/disable automatic management of the credential object (yes/no) | No
| `cyberark_acct_manualReason` | The reason automatic management is being disabled on the credential object | No

## Example Playbook

An example of deploying a LAMP stack and onboarding the resulting MySQL database administrator that is created during initialization of MySQL. 

```yaml
---
- hosts: localhost

  pre_tasks:
    - name: Install Apache & PHP
      yum:
        name: ['httpd', 'php', 'php-mysql']
        state: present

    - name: Install Web Role Specific Dependencies
      yum:
        name: ['git', 'wget', 'curl', 'jq', 'libsemanage-python']
        state: present

    - name: Start Apache
      service:
        name: httpd
        state: started
        enabled: yes

    - name: Configure SELinux to Allow httpd Connection to Remote Database
      seboolean:
        name: httpd_can_network_connect_db
        state: true
        persistent: yes

    - name: Create index.php Start Page
      copy:
        dest: "/var/www/html/index.php"
        content: |
          <?php echo "Hello World!"; ?>
      
    - name: Install MariaDB Package
      yum:
        name: ['mariadb-server', 'MySQL-python']
        state: present

    - name: Configure SELinux to Start MySQL on Any Port 
      seboolean:
        name: mysql_connect_any
        state: true
        persistent: yes

    - name: Start MySQL Service
      service:
        name: mariadb
        state: started
        enabled: yes

    - name: Create a New Database
      mysql_db:
        name: demo
        state: present
        collation: utf8_general_ci

    - name: Set Fact with Current Remote Machine Hostname
      set_fact:
        mysql_address: "{{ inventory_hostname }}"

    - name: Set Fact with MySQL Database Username
      set_fact:
        mysql_username: demo

    - name: Set Fact with Randomized Password for MySQL Database User
      set_fact:
        mysql_password: "{{ lookup('password', '/dev/null length=15 chars=ascii_letters') }}"
      no_log: yes

    - name: Create a Database User
      mysql_user:
        name: "{{ mysql_username }}"
        password: "{{ mysql_password }}"
        priv: "*.*:ALL"
        host: localhost
        state: present

    - name: Copy sample data to local filesystem /tmp/
      copy:
        src: files/dump.sql
        dest: /tmp/dump.sql
    
    - name: Insert sample data into demo table in MySQL
      shell: "mysql -u {{ mysql_username }} -p{{ mysql_password }} demo < /tmp/dump.sql"

    - name: Restart Apache 
      service:
        name: httpd
        state: restarted

    - name: Install Database Connection PHP Script db.php
      copy:
        src: files/db.php
        dest: /var/www/html/db.php

  roles:
    - role: infamousjoeg.provisioning
      cyberark_api_base_url: https://components.cyberarkdemo.example
      cyberark_auth_type: LDAP
      cyberark_validate_certs: no
      cyberark_username: Svc_ProvTest_Fedora
      cyberark_password: Cyberark1
      cyberark_acct_name: TEST-AUTO-ONBOARD-{{ mysql_address }}-{{ mysql_username }}
      cyberark_acct_address: "{{ mysql_address }}"
      cyberark_acct_username: "{{ mysql_username }}"
      cyberark_acct_password: "{{ mysql_password }}"
      cyberark_acct_platformId: MySQL
      cyberark_acct_safeName: TEST-AUTO-ONBOARD
      cyberark_acct_secretType: password
      cyberark_acct_autoManagement: no
      cyberark_acct_manualReason: For demo purposes
```

[![asciicast](https://asciinema.org/a/245088.svg)](https://asciinema.org/a/245088)

## Test

### Requirements

* Python 2.7.x
* Docker CE
* Ansible >= 2.5
* `pip install --user molecule`
* `pip install molecule[docker]`

### Usage

Test using Ansible Molecule:

```
molecule test
```

## License

MIT
