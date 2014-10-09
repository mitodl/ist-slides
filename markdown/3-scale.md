# Production Scale

Not really different than mitxstack

Differences largely:
 - Caring about availability
 - Caring about data
 - "common cluster"
 - nginx templates
 - HAProxy

---

# Demo: OpenStack Horizon

- Networks
- Instances
- Naming Conventions

---

# Demo: Zenoss Monitoring

- Infrastructure
- Events
- Network Map

---

# Demo: Fix ShellShock

- Same as earlier, but swap the inventory
```terminal
ansible tag_env_rp-os -i nova.py -m apt \
-a 'name=bash state=latest update_cache=true' -s
```

Get all demo commands at: http://goo.gl/u2hBqy

---

# Demo: Upgrade to latest platform

Running an "in-production" deploy

Not much different than running `edxapp.yml` from earlier

Adds more advanced `serial` concept of removing app server,
upgrading, adding it back.

```terminal
time ../shell/app_deploy_os.sh -d -v dev rp-os d_edxapp.yml &\
time ../shell/app_deploy_os.sh -d -v prod rp-os p_edxapp.yml
```

---

# Mongo Cluster Management

Find the master node with ansible

```terminal
brandon will do this
```

Failing mongo master and recovery

```terminal
ansible 'tag_group_apps:tag_group_papps:&tag_env_rp-os' -i nova.py \
-m shell -a '/edx/bin/supervisorctl restart edxapp:*;\
/edx/bin/supervisorctl restart edxapp_worker:*;' -s -f 1
```


---

# RabbitMQ Cluster Management

Show current queue status:

```terminal
ansible 'tag_group_mongo:&tag_env_rp-os' -i nova.py \
-m shell -a 'rabbitmqctl cluster_status' -s
```

```terminal
ansible 'tag_group_mongo:&tag_env_rp-os' -i nova.py \
-m shell -a 'rabbitmqctl list_vhosts' -s
```

```terminal
ansible 'tag_group_mongo:&tag_env_rp-os' -i nova.py \
-m shell -a 'rabbitmqctl list_queues -p /dev' -s
```

```terminal
ansible 'tag_group_mongo:&tag_env_rp-os' -i nova.py \
-m shell -a 'rabbitmqctl list_queues -p /prod' -s
```

---

# Production Kibana

Login to https://log-rp-os.mitx.mit.edu

Mess around as there should be many more logs than on mitxstack

Open up **OpenStack Residential Environment** dashboard

- Try filtering by host
- Add pie chart (terms) for logs by host
- Find yourself in tracking logs from browsing
  https://prod-rp-os.mitx.mit.edu


???

Import demo course and ask them to find messages based on it
