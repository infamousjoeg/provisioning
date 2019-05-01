# Provisioning

An Ansible role to ease the task of onboarding accounts into CyberArk's Core Privileged Access Security (PAS) using CyberArk's Management API.

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
| `cyberark_acct_name` | Credential object's unique "Name" value from PVWA | No
| `cyberark_acct_address` | Credential object's address value from PVWA | No
| `cyberark_acct_username` | Credential object's username value from PVWA | No
| `cyberark_acct_password` | Credential object's password value from PVWA | No
| `cyberark_acct_platformId` | Platform to manage the credential object | Yes
| `cyberark_acct_safeName` | Safe to store the credential object within | Yes
| `cyberark_acct_secretType` | The type of secret being onboarded (password/key) | Yes
| `cyberark_acct_autoManagement` | Boolean value to enable/disable automatic management of the credential object (yes/no) | No
| `cyberark_acct_manualReason` | The reason automatic management is being disabled on the credential object | No
| `cyberark_acct_logonDomain` | The logon domain to be used for the credential object (LOGONDOMAIN\Administrator) | No
| `cyberark_acct_port` | The port used for automatic management and connections using the credential object | No
| `cyberark_acct_remoteMachines` | List of remote machines the credential object may connect to (Example: server1.cyberark.com;server2.cyberark.com) | No
| `cyberark_acct_accessRestrictedToRemoteMachines` | Boolean value to restrict access only to specific remote machines listed (yes/no) | No

## Example Playbook

An example of deploying a LAMP stack and onboarding the resulting MySQL database administrator that is created during initialization of MySQL.

```yaml
---
- hosts: all

  pre_tasks:
    - name: Install Apache & PHP
      yum:
        name: "{{ item }}"
        state: present
      with_items:
          - httpd
          - php
          - php-mysql

    - name: Install Web Role Specific Dependencies
      yum:
        name: "{{ item }}"
        state: present
      with_items:
          - git
          - wget
          - curl
          - jq
          - libsemanage-python

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
        name: "{{ item }}"
        state: present
      with_items:
          - mariadb-server
          - MySQL-python

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
        name: sa
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
    - role: provisioning
      cyberark_api_base_url: https://pvwa.cyberark.com
      cyberark_auth_type: LDAP
      cyberark_validate_certs: yes
      cyberark_username: Svc_CYBR_Ansible_DEV
      cyberark_password: Password123
      # cyberark_acct_name: Not provided to automatically generate on add
      cyberark_acct_address: "{{ mysql_address }}"
      cyberark_acct_username: "{{ mysql_username }}"
      cyberark_acct_password: "{{ mysql_password }}"
      cyberark_acct_platformId: MySQL
      cyberark_acct_safeName: CyberArk_Example_SafeName
      cyberark_acct_secretType: password
      cyberark_acct_autoManagement: yes
      # cyberark_acct_manualReason: Not needed in this example
      # cyberark_acct_logonDomain: Not needed in this example
      cyberark_acct_port: 3306
      # cyberark_acct_remoteMachines: Not needed in this example
      cyberark_acct_accessRestrictedToRemoteMachines: no
```

## License

MIT
