class: title, center, middle

# Basic Ansible Usage

### We'll be covering:

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
- Vagrant ssh private key is at `~/.vagrant.d/insecure_private_key`
- http://docs.ansible.com/intro_inventory.html

---

# Inventory Solution

```ini

[mitxstack]
v1 ansible_ssh_host=192.168.33.10 ansible_ssh_user=vagrant
 ansible_ssh_private_key_file=~/.vagrant.d/insecure_private_key

```

Get at http://goo.gl/aM7nXA

Can test with:

```terminal
ansible all -i mitxstack.ini -m ping
```
Should get a `pong` back

---

# Fix ShellShock Bug on mitxstack

Repair the bash vulnerability in mitxstack using `ansible` and your
newly minted inventory file.

Check if vulnerable first with:

```terminal
env x='() { :;}; echo vulnerable' bash -c "echo this is a test"
```

## Hint
Apt module makes this easy: http://docs.ansible.com/apt_module.html

---

# Shellshock solution

```terminal
ansible all -i mitxstack.ini -m apt -a 'name=bash state=latest update_cache=true' -s
```

Get at http://goo.gl/VQp2xA

Much easier than:

```terminal
vagrant ssh
sudo apt-get update -y
sudo apt-get install --only-upgrade bash
```

and you can run it on your entire fleet with the same one line

---

# Restart nginx in mitxstack

Use ansible to stop nginx, confirm it is down, and then start it.

## Hint
Service module makes this pretty slick: http://docs.ansible.com/service_module.html

---
# Nginx Solution

```terminal
ansible all -i mitxstack.ini -m service -a 'name=nginx state=stopped' -s
```

Go to http://192.168.33.10 to verify it times out
or for bonus points:

```terminal
ansible all -i mitxstack.ini -m wait_for -a 'port=80 delay=1 timeout=5'
```
should fail

```terminal
ansible all -i mitxstack.ini -m service -a 'name=nginx state=started' -s
```

and refresh browser to verify everything is back in mitxstack or reuse
the `wait_for` command

Get at http://goo.gl/GV8V3Z

---

# Cat a file or files on mitxstack

Maybe this is easier to do by hand with one host, but with a couple or dozens
this makes this chore much easier.

Use ansible to cat out `/var/log/syslog` and then cat out all the files
in `/edx/var/log/supervisor/` that start with lms and end with log

## Hint
The shell module in ansible is the great swiss army knife of ansible

---

# cat Solution


```terminal
ansible all -i mitxstack.ini -m shell -a 'cat /var/log/syslog' -s
ansible all -i mitxstack.ini -m shell -a 'cat /edx/var/log/supervisor/lms*.log' -s
```
should take care of cat'ing all the files to your terminal. This will also
work with multiple servers and will output by server so piping this into
`less` is handy.  You can even redirect it to a file on a loop and
tail that file for multiserver multifile tail. e.g.:

```terminal
while true; do \
ansible -i mitxstack.ini -m shell \
-a 'cat /edx/var/log/supervisor/lms*.log' all -s > tail.log; \
sleep 1; done & tail -f tail.log
```

Get at: http://goo.gl/blA64b

---

# Upgrade edx-platform without the script

edx-platform, as well as the other components of the stack, can be easily
updated using an update script which runs ansible. For example:

```terminal
sudo /edx/bin/update edx-platform mitx-release
```

Using ansible from outside your vagrant box, update your edx-platform to
`mitx-hotfix-20140919`, our latest hotfix branch.

## Hint:

You'll need to copy `server-vars.yml` file from the vagrant box to
your host machine.

It's all in the script.

---

# Upgrade solution

You need to copy the `/edx/app/edx_ansible/server-vars.yml` file from
the vagrant box to your host machine, then run the `edxapp` playbook as follows:

```terminal
ansible-playbook edx-east/edxapp.yml -i mitxstack.ini -e @server-vars.yml -e \
'edx_platform_version=mitx-hotfix-20140919' --tags deploy
```
Get at http://goo.gl/PqpHJz
