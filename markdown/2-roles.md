class: title, center, middle

# From Basics to Roles

### Jumping way ahead in ansible to roles, with some playbooks along the way.

- Logs are nice, but pretty logs are better:
  Install Kibana/Logstash/Elasticsearch
- Add some more MITx flavor to the mitxstack with gitreload:
  Sidebar on the sysadmin dashboard

---

# Playbooks and Roles and Something or Other

- Roles are like packages in ansible
- Playbooks run roles
- edx/configuration is full of treasures (65 roles currently)
- From here on out, we'll be running out of the edx/configuration repo
  in the `playbooks` directory

---

# Three Services, so Hard (ELK stack)

Install the elasticsearch, logstash, and kibana stack on your mitxstack
with kibana being available on port 10000.

## Hints
- KIBANA_NGINX_PORT is the var with port set
- The play to install the full ELK stack is already there and log(gy)
- playbooks run with `ansible-playbook` and not `ansible`

---

# Install ELK solution

The play to use is log_server.yml which is very simple:

```yaml
# Build a kibana/logstash/elasticsearch server for capturing and
# analyzing logs.
- name: Configure syslog server
  hosts: all
  sudo: yes
  roles:
    - common
    - oraclejdk
    - elasticsearch
    - logstash
    - kibana
    - role: nginx
      nginx_sites:
        - kibana
```

To run it with our overrides (to the port, which defaults to 80), we run

```terminal
ansible-playbook -i mitxstack.ini log_server.yml -e KIBANA_NGINX_PORT=10000
```

Once complete, verify by going to: http://192.168.33.10:10000

---

# Where are the logs?

So our kibana looks nice and all, but where are the logs?

You get this one for free, to enable rsyslog forwarding of logs run:

```terminal
ansible all -i mitxstack.ini -m shell -a 'echo "*.* @127.0.0.1" > \
/etc/rsyslog.d/99-syslogforward.conf' -s
```

But you have to restart the rsyslog service.

Get at: http://goo.gl/JGTjFA

---

# Syslog Restart and Confirmation

Restart with a module and command we have used already

```terminal
ansible all -i mitxstack.ini -m service -a 'name=rsyslog state=restarted' -s
```

You can generate a test message with:
```terminal
ansible all -i mitxstack.ini -m shell -a 'logger This is my log message'
```

To start getting edx logs change `server-vars.yml` to set `EDXAPP_SYSLOG_SERVER` to
'localhost' and re-run:

```terminal
ansible-playbook edx-east/edxapp.yml -i mitxstack.ini -e @server-vars.yml \
--tags deploy
```

Get at http://goo.gl/LpO8dp

---

# Extra Roles, gitreload, and secrets

Let's install a role not in edx/configuration with gitreload

Repository with role is at:
https://github.mit.edu/mitx-devops/gitreload-role
or tar balled at:
http://public.mitx.mit.edu/dist/gitreload.tar.gz

## Hints

- Check `edx/configuration/playbooks/ansible.cfg` for where to put the
  role.
- Check out `run_role.yml` for how to run this against mitxstack
- `group_vars/all` defines secure_dir and you will need to create it
  for this role and add a key (even if it is fake)
- Verify at https://192.168.33.10:8095/queue

---

# gitreload Solution

- Download role to `edx/configuration/../../ansible_roles`
- Run:
```terminal
ansible-playbook -i mitxstack.ini run_role.yml -e role=gitreload'
```
- Have that fail at step `install ssh key for the content repos` since
  you likely don't have
  `../../ops/edx/configuration/playbooks/path/to/secure_example/keys/gitreload`
- Create a folder somewhere (usually `edx/configuration../../secure_dir`) add a
  `keys` folder and either create a blank file or copy your private ssh key
  file to `gitreload` inside that directory.
- Run
```terminal
ansible-playbook -i mitxstack.ini run_role.yml -e role=gitreload \
-e secure_dir='../../secure_dir'
```

Get at: http://goo.gl/twGbWG

---

# Skipping Steps and Tags

Notice that we skipped a couple tasks in that play, well let's run those!

- Sysadmin Dashboard: http://192.168.33.10/sysadmin
- Delete demo course

## Tasks

- Run play such that the default course gets imported when running gitreload
- Run play such that only the course gets imported (no other tasks run)

## Hints
- `course_checkout`
- tags

---

# Skipping Steps and tags Solution

To run gitreload, check out `../../ansible_roles/gitreload/tasks/main.yml` and notice
there is a step with `when: course_checkout|bool` and run:

```terminal
ansible-playbook -i mitxstack.ini run_role.yml -e role=gitreload \
-e secure_dir='../../secure_dir' \
-e course_checkout=yes
```

Also notice the `tags: course_pull` statement, and run
```terminal
ansible-playbook -i mitxstack.ini run_role.yml -e role=gitreload \
-e secure_dir='../../secure_dir' \
-e course_checkout=yes --tags course_pull
```

to only run the course import.

Checkout the the `sysadmin dashboard` to confirm the course is loaded and what
sha1 it has to confirm it was git loaded

Get at: http://goo.gl/61oSw1
