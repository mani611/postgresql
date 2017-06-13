Postgresql Master/Slave
=======================

Installs postgresql in master/slave and Failover using Pacemaker.

Requirements
------------

Two node setup installed with CentOS/Redhat7.

Roles and Variables
--------------

A description of the mandatory variables for this playbook is as given here. All these variables need to specify inside main (install.yml) playbook.<br/> 
**postgres :-** It will enable postgres 9.6 repo and install postgres in standalone mode. No variables required to use this role.<br/>
**postgres-master :-** It will configure postgres server to act as master server and allow replication. Need to specify `master_ip` variable value as server IP itself to restrict self replication.<br/>
**postgres-slave :-** It will configure slave server. Need to specify `master_ip` variable to know the slave about primary connection info.<br/>
**postgres-pcsinstall :-** It will install all the pacemaker related necessary packages and and also copy the pcs configuration shell script under /tmp/cluster_pgsql.sh . Here we have some variables that will get replaced in shell script before copy it on target server.
``hacluster_password`` Password for hacluster user.
``cluster_name`` Name to setup cluster with.
``cluster_nodes`` Hostname of both cluster node separated by space (e.g. node1 node2).
``master_vip`` Virtual IP for Master node.
``master_cidr`` CIDR for Virtual IP address (e.g. 24).

Dependencies
------------

All the server should be setup with os base repository and accesible using hostname.

Example Playbook
----------------

Example of how to use your role in install.yml and also passed the varibales in this main playbokk itslef. 

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
**Note :** Kindly pay attention over "Install Pacemaker and Corosync" play. Here, we are directly stopping pacemaker service. It is required because, pacemaker will take care of postgres service.

Author Information
------------------

Shabbir Ahmad 

