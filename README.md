[![Build Status](https://travis-ci.org/zaxos/glusterfs-ansible-role.svg?branch=master)](https://travis-ci.org/zaxos/guacamole-ansible-role)
[![Ansible Galaxy](https://img.shields.io/badge/galaxy-_zaxos.glusterfs--ansible--role-blue.svg)](https://galaxy.ansible.com/zaxos/guacamole-ansible-role/)

glusterfs-ansible-role
======================

Ansible role to install and configure GlusterFS.

Requirements
------------
* centos/rhel 7
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
    glusterfs_default_bricks_dir: /bricks
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
        nfs.disable: "off"
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
  state: present/absent  # required #
  
  # Fill accordingly depending on the type of volume to be created #
  replica:
  arbiter:
  stripe:
  disperse:
  redundancy:
  
  transport:  # optional, default is tcp #
  force:  # optional, default is True #
  nodes:  # optional, custom nodes list #
    - node1
    - node2
    - ...
  bricks:  # optional, custom brick path and/or multiple bricks per node #
    - /../brick1
    - /../brick2
    - ...
  options:  # optional, glusterfs volume options #
    nfs.disable: "off"
    auth.reject: "10.10.10.*"
    ...
  mount:  # optional, mount volume in each node #
    path:  # required if mount: is defined #
    owner:  # optional, default "root" #
    group:  # optional, default "root" #
    mode:  # optional, default "0755" #
```

Role Variables
--------------
Some variables that require review:
- `glusterfs_default_bricks_dir`: Default directory for brick storage. Each volume will create its own subdirectory e.g. "/bricks/volume1".
 - `glusterfs_version`: GlusterFS version to be installed using CentOS Storage SIG Packages. Default is the currently latest version available "3.9".
- `glusterfs_nodes`: List of nodes in GlusterFS cluster. By default this list is populated by the defined group in inventory (glusterfs_example_cluster in example inventory).
- `glusterfs_delete_bricks_dir_after_removal`: Default value is "True". Set this variable to "False" if you want to preserve volume brick directories in all nodes after a volume is removed (state: absent).
