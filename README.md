# Ansible

[Ansible Documentation](https://docs.ansible.com/)

## Ansible Introduction

- Simple, Powerful and Agentless
- Provisioning,`configuration management`, continuous delivery, application deployment, security compliance

Ansible Controller - where we install ansible.

Ansible Control Machine

- Playbooks
- Inventory
- Modules

Control Machine - Linux Only

Managed Nodes - system we control using ansible.

- AnsibleController (Control Node)
  - install Ansible
  - create a file pratima.pem and upload the private key contents of pratima.pem to access another EC2 instance
  - give permission `chmod 600 pratima.pem`
  - `ssh -i pratima.pem ubuntu@<targetprivateip>`
  - eg. `ssh -i ~/key.pem ubuntu@172.31.82.32`

- AnsibleTarget01 (Managed Node) - `sudo apt-get update`
- AnsibleTarget02 (Managed Node)- `sudo apt-get update`

In the AnsibleController,

- create a folder to test. `mkdir ansible-test`
- `cd ansible-test`
- `touch inventory.txt`
  - add the target private ip address to the inventory.txt file.

    ```text
    [servers]
    172.31.82.32 ansible_user=ubuntu ansible_ssh_private_key_file=~/pratima.pem
    172.31.84.144 ansible_user=ubuntu ansible_ssh_private_key_file=~/pratima.pem
    ```

- `ansible servers -m ping -i ~/test-project/inventory.txt`
  
  Expected response:

    ```bash
        172.31.82.32 | SUCCESS => {
            "ansible_facts": {
                "discovered_interpreter_python": "/usr/bin/python3"
            },
            "changed": false,
            "ping": "pong"
        }
        172.31.84.144 | SUCCESS => {
            "ansible_facts": {
                "discovered_interpreter_python": "/usr/bin/python3"
            },
            "changed": false,
            "ping": "pong"
        }
    ```

- Undo Changes in Vim / Vi
  - Press esc to return to normal mode. Any character typed in normal mode will be interpreted as a vim command.
  - Press u, :u, or :undo to undo the last change (entry).

## Understanding YAML

- Key Value Pair
- Array/Lists
- Dictionary/Map
- Dictionary vs List vs List of Dictionaries

(Equal number of spaces is important)

`Dictionaries` are unordered while `lists and arrays` are ordered.

Agentless - not required to install any agent on the target machine.

## Inventory

An inventory file contains a list of the hosts you’ll manage using Ansible.

```text
# Sample Inventory file
# Web Servers
web1  ansible_host=server1.company.com ansible_connection=ssh ansible_user=root ansible_ssh_pass=Password
web2 ansible_host=server3.company.com ansible_connection=ssh ansible_user=root ansible_ssh_pass=Password

# Database Servers
db1  ansible_host=server2.company.com ansible_connection=winrm ansible_user=Administrator ansible_password=Password
db1 ansible_host=server4.company.com ansible_connection=winrm ansible_user=root ansible_password=Password

[web_servers]
web1
web2

[db_servers]    
db1
db2

[all_server:children]
web_servers
db_servers

```

Note: For Linux based hosts, use ansible_ssh_pass parameter and for Windows based hosts, use ansible_password parameter.

localhost ansible_connection=localhost

Inventory Parameters:

- ansible_connection - ssh/winrm/localhost
- ansible_port - 22/5986
- ansible_user - root/administrator
- ansible_ssh_pass - Password

Security: Ansible Vault

`student-node` :- This host will act as an Ansible master node where you will create playbooks, inventory, roles etc and you will be running your playbooks from this host itself.

`node01` :- This host will act as an Ansible client/remote host where you will setup/install some stuff using Ansible playbooks. Below are the SSH credentials for this host:

```text
User: bob
Password: caleston123
```

`node02` :- This host will also act as an Ansible client/remote host where you will setup/install some stuff using Ansible playbooks. Below are the SSH credentials for this host:

```text
User: bob
Password: caleston123
```

## Playbook

- A single YAML file
  - Play (Define a set of activities tasks) to be run on hosts
    - Tasks (An action to be performed on host)

      - `Executing command`

      - `Running a script`

      - `Installing a package`

      - `Shutdown / Restart`

Running a Playbook

Syntax:

`ansible-playbook playbook.yml`

`ansible-playbook -i inventory playbook.yaml`

## Modules

- System (user, group, hostname, iptables, lvg, lvol, make, mount, ping, timezone, user, systemmd, service,etc.)
- Commands (command, expect, raw, script, shell)
- Files (acl, archive, copy, file, find, lineinfile, replace, stat, template, unarchive, etc.)
- Database (cassandra, mysql, postgresql, sqlite, vertica,etc.)
- Cloud (aws, azure, docker, google, openstack, vmware, etc.)
- Windows (win_copy, win_file, win_find, win_get_url, win_group, win_msi, win_package, win_ping, win_regedit, win_service, win_shell, win_template, win_user, etc.)
- More...

## Command

Execute a command on a remote node.

Parameters:

- chdir - Change into this directory before running the command.
- creates - A filename or (since Ansible 2.3) glob pattern, when it already exists, this step will not be run.
- executable - Change the shell used to execute the command.
- free_form - The command module takes a free form command to run, as a string.
- removes - A filename or (since Ansible 2.3) glob pattern, when it does not exist, this step will not be run.
- warn - Turn task warnings on or off for this command.

## Script

- Runs a local script on a remote node after transferring it.
  1. Copy the script to the remote node
  2. Execute it on the remote node

## Service

- Manage services: start, stop, restart

`idempotency`

Why 'started' and not 'start'?

'start' the service httpd

`'started' the service httpd`

Ensure service httpd is started.

If httpd is not yet started &rarr; start it
If httpd is already started &rarr; do nothing

## lineinfile

- Ensure a particular line is in a file, or replace an existing line using a back-referenced regular expression.
- Search for a line in a file and replace it or add it if it doesn't exist.

```yaml
- name: Add DNS server to resolv.conf
  hosts: localhost
  tasks:
    - lineinfile:
        path: /etc/resolv.conf
        line: 'nameserver 10.1.250.10' 
```

## Loops

With_*

```yaml
-   name: Create users
    hosts: localhost
    tasks:
        - user: name= state=present
            loop:
                - joe
                - george
                - ravi
                - mani
                - kiran
                - jazlan
                - emaan
                - mazin
                - izaan
                - mike
                - menaal
                - shoeb
                - rani
```

## Conditionals

### Conditionals - when

### Operator - or, and

```yaml
- name: Install NGINX
  hosts:
  tasks:
    - name: Install NGINX on Debian
      apt:
        name: nginx
        state: present
      when: ansible_os_family == “Debian” and ansible_distribution_version == “16.04”
    - name: Install NGINX on RedHat
      yum:
        name: nginx
        state: present
      when: ansible_os_family == “RedHat” and ansible_distribution_version == “7.4”
```

### Conditionals in Loop

### Conditionals and Register

## Roles

A role is a set of playbooks and related files organized into a predefined structure that is known by Ansible. Roles facilitate reusing and repurposing playbooks into shareable packages of granular automation for specific goals, such as installing a web server, installing a PHP environment, or setting up a MySQL server.
