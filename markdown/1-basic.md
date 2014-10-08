# Basic Ansible Usage

We'll be covering:

- Static inventory
- one off commands
- intro to using modules
- using ansible to manage edx-platform

---

# Create a simple inventory for mitxstack

Create an inf inventory file that will specifies all the connection
information you need to connect to mitxstack.
## Hints:

- IP is in Vagrantfile
- Vagrant ssh private key is at `/.vagrant.d/insecure_private_key`

---

# Inventory Solution

```ini

[mitxstack]
v1 ansible_ssh_host=192.168.33.10 ansible_ssh_user=vagrant
 ansible_ssh_private_key_file=~/.vagrant.d/insecure_private_key

```

Get at http://goo.gl/aM7nXA

---

# Fix ShellShock Bug on mitxstack

Repair the bash vulnerability in mitxstack using `ansible` and your
newly minted inventory file.

## Hint
Apt module makes this easy: http://docs.ansible.com/apt_module.html

---

# Shellshock solution

```terminal
ansible all -i mitxstack.ini -m apt -a name=bash state=latest update_cache=true
```

Get at http://goo.gl/VQp2xA

