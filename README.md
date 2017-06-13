Postgresql Master/Slave
=======================

Installs postgresql in master/slave and Failover using Pacemaker.

Requirements
------------

Two node setup installed with CentOS/Redhat7.

Roles and Variables
--------------

A description of the mandatory variables for this playbook is as given here. All these variables need to specify inside main (install.yml) playbook. 
**postgres :-** It will enable postgres 9.6 repo and install postgres in standalone mode. No variables required to use this role.
**postgres-master :-** It will configure postgres server to act as master server and allow replication. Need to specify `master_ip` variable value as server IP itself to restrict self replication.
**postgres-slave :-** It will configure slave server. Need to specify `master_ip` variable to know the slave about primary connection info.


Dependencies
------------

A list of other roles hosted on Galaxy should go here, plus any details in regards to parameters that may need to be set for other roles, or variables that are used from other roles.

Example Playbook
----------------

Including an example of how to use your role (for instance, with variables passed in as parameters) is always nice for users too:
```
---
- name: Install Postgres 9.6
  hosts: all
  roles:
   - postgres

- name: Configure Postgres 9.6 master
  hosts: master
  vars:
    master_ip: 172.16.0.200
  roles:
   - postgres-master

- name: Configure Postgres 9.6 slave
  hosts: slave
  vars:
    master_ip: 172.16.0.200
  roles:
   - postgres-slave

- name: Install Pacemaker and Corosync
  hosts: all
  vars:
   hacluster_password: hacluster
   cluster_name: cluster_pgsql
   cluster_nodes: postgres1 postgres2
   master_vip: 172.16.0.210
   master_cidr: 24
  roles:
   - postgres-pcsinstall
  tasks:
   - shell: 'systemctl stop postgresql-9.6'

- name: Configure Pacemaker Cluster
  hosts: master
  tasks:
   - shell: '/bin/bash < /tmp/cluster_pgsql.sh'
```

License
-------

MIT

Author Information
------------------


Derived from 

