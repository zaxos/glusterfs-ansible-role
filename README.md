[![Build Status](https://travis-ci.org/zaxos/glusterfs-ansible-role.svg?branch=master)](https://travis-ci.org/zaxos/glusterfs-ansible-role)
[![Ansible Galaxy](https://img.shields.io/badge/galaxy-_zaxos.glusterfs--ansible--role-blue.svg)](https://galaxy.ansible.com/zaxos/glusterfs-ansible-role/)

glusterfs-ansible-role
======================

Ansible role to install and configure GlusterFS.

Requirements
------------
* centos 7
* ansible >= 2.2

Installation
------------
```
$ ansible-galaxy install zaxos.glusterfs-ansible-role
```

Example Inventory
-----------------
```
[glusterfs_example_cluster]
node1.glusterfs.example
node2.glusterfs.example
node3.glusterfs.example
...
```

Example Playbook
----------------
```yaml
- hosts: glusterfs_example_cluster
  vars:
    glusterfs_version: "3.10"
    glusterfs_default_bricks_dir: /bricks
    glusterfs_configure_firewalld: True
    glusterfs_volumes:
    - volume: volume1  # Replicated with arbiter node
      state: present       
      replica: 3
      arbiter: 1
      mount:
        path: "/mnt/volume1"
            
    - volume: volume2  # Distributed in 2 nodes
      state: present
      nodes:
        - node2.glusterfs.example
        - node3.glusterfs.example
      options:
        performance.cache-size: 256MB
      mount:
        path: "/mnt/volume2"
        owner: exampleuser
        group: examplegroup
        mode: "0770"
            
  roles:
    - role: zaxos.glusterfs-ansible-role
```

Example volume
--------------
```yaml
- volume: example
  state: present/absent  # optional, default is 'present', set to 'absent' for removal #
  
  # Fill accordingly depending on the type of volume to be created #
  replica:
  arbiter:
  stripe:
  disperse:
  redundancy:
  
  transport:  # optional, default is tcp #
  force:  # optional, default is yes #
  nodes:  # optional, custom nodes list #
    - node1
    - node2
    - ...
  bricks:  # optional, custom brick path and/or multiple bricks per node #
    - /../brick1
    - /../brick2
    - ...
  options:  # optional, glusterfs volume options #
    performance.cache-size: 256MB
    ...
  mount:  # optional, mount volume in each node #
    path:  # required if mount: is defined #
    owner:  # optional, default is "root" #
    group:  # optional, default is "root" #
    mode:  # optional, default is "0755" #
    options:  # optional, default is "defaults,_netdev,backupvolfile-server=..." #
```

Role Variables
--------------
Some variables that require review:
- `glusterfs_default_bricks_dir`: Default directory for brick storage. Each volume will create its own subdirectory e.g. "/bricks/volume1".
- `glusterfs_version`: GlusterFS version to be installed using CentOS Storage SIG Packages. Latest stable version is "3.10".
- `glusterfs_nodes`: List of nodes in GlusterFS cluster. By default this list is populated by the defined group in inventory (glusterfs_example_cluster in example inventory).
- `glusterfs_volumes`: List of volumes.
- `glusterfs_configure_firewalld`: If set to "True", firewalld will be installed and properly configured. Default value is "False".
- `glusterfs_auto_remount`: If set to "True", when the mount path of a volume is changed, the old mount path will be automatically unmounted and removed from fstab. Default value is "True".
- `glusterfs_delete_bricks_dir_after_removal`: Default value is "True". Set this variable to "False" if you want to preserve volume brick directories in all nodes after a volume is removed (state: absent).
