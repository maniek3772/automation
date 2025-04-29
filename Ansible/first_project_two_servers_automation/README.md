# Ansible — Two Servers Automation

This project shows how to set up **Ansible** to manage multiple Linux servers using **SSH keys** and **sudo passwords** stored securely with **Ansible Vault**.

---

## 🖥️ Project Structure

```ini
├── ansible.cfg #Ansible configuration file 
├── hosts # Inventory with server IPs and users 
├── vault.yml # Encrypted file with sudo passwords
└── playbook.yml # Main Ansible playbook
```
---

## ⚙️ Step-by-Step Setup

### 1. Inventory File (`hosts`)

Define your servers and SSH users:

```ini
[web_servers]
192.168.1.186 ansible_user=matr

[db_servers]
192.168.1.187 ansible_user=matr2
```
### 2. Vault File (vault.yml)

Store sudo passwords per server IP. Example content before encryption:

```ini
become_passwords:
  "192.168.1.186": "your_sudo_password_for_matr"
  "192.168.1.187": "your_sudo_password_for_matr2"
```

Encrypt the file using:
```ini
ansible-vault encrypt vault.yml
```

### 3. Ansible configuration (ansible.cfg)

```ini
[defaults]
host_key_checking = False #Only in TEST ENVIRONMENT
inventory = hosts

[ssh_connection]
ssh_args = -o StrictHostKeyChecking=no -o UserKnownHostsFile=/dev/null
```

### 3. Add playbook (playbook.yml)

```ini

# Playbook to create a directory on all servers, handling sudo passwords dynami>

- hosts: all
  gather_facts: false  # <--- Disable gathering facts
  become: yes
  vars_files:
    - vault.yml

  pre_tasks:
    - name: Set the sudo password dynamically for each server
      set_fact:
        ansible_become_password: "{{ become_passwords[inventory_hostname] }}"

    - name: Gather facts manually
      setup:

  tasks:
    - name: Create a test directory
      file:
        path: /tmp/ansible_test_directory
        state: directory
        mode: '0755'
```

❓ What is "Gathering Facts"?
Facts are automatic information about the servers (IP addresses, RAM, OS type, etc.) collected by Ansible at the beginning of a playbook.
We manually gather facts after setting ansible_become_password to avoid sudo password issues.


### 4. Run playbook
Execute:
```ini
ansible-playbook -i hosts playbook.yml --ask-vault-password
```



## 📚 Useful Commands
| Command | Description |
| --- | --- |
| ansible all -m ping | Test connection to all hosts |
| ansible-playbook -i hosts playbook.yml	--ask-vault-password | Run the full playbook |
| ansible-vault encrypt vault.yml |	Encrypt Vault file |
| ansible-vault decrypt vault.yml |	Decrypt Vault file (for editing) |