ansible-ha-storage
==================

This role is used to create and manage filesystems using corosync / pacemaker.

Two types of storage are supported: ZFS and cLVM / LVM.

According to the OS, pcs or crmsh are installed:

* pcs: RedHat-based systems ;
* crmsh: the others.

The playbook is tested on :

* CentOS and Ubuntu (on a daily basis) ;
* FreeBSD (sometimes).

Using crmsh allows us to generate crm scripts per pool / volume group.

Requirements
------------

Storage provisionning:

* in an Openstack environment, a valid openrc file and some existing volumes 
are required ;
* in a physical environment, the LUNs need to be mapped to the servers and 
their WWNs are known.


Cluster configuration:

* the resource placement is made thanks to attributes. The nodes must have these 
values defined ;
* the cluster must be healthy, ie. with quorum and all nodes online.

Role Variables
--------------

A list of mountpoints needs to be given.

Dependencies
------------

This playbook is a standalone playbook. However, it is co-developed with 
[ansible-ha-cluster](https://github.com/mgrzybek/ansible-ha-cluster).

Example Playbook
----------------

    - name: Create some clustered filesystems
      hosts: all
      roles:
        - { role: ansible-ha-storage }
      vars:
        ha_storage:
          luns:
            - id: 23234
              alias: lun-dc-1
            - id: 45345
              alias: lun-dc-2
          volume_group: db
          filesystems:
            - lv: data
              size: 20G
              fstype: xfs
              mountpoint: /srv/data
            - lv: xlog
              size: 2G
              fstype: xfs
              mountpoint: /srv/xlog

    - name: Create some clustered filesystems
      hosts: all
      roles:
        - { role: ansible-ha-storage }
      vars:
        ha_storage_fs: zfs
        ha_storage_zpool_mountpoint: /srv
        ha_storage_cluster_attribute_name: role
        ha_storage_cluster_attribute_value: database
        ha_storage:
          luns:
            - id: /dev/vdb
              alias: lun-dc-1
            - id: /dev/vdc
              alias: lun-dc-2
          volume_group: db
          filesystems:
            - lv: data
              size: 20GB
              mountpoint: data
            - lv: xlog
              size: 2GB
              mountpoint: xlog

    - name: Create some clustered filesystems using a Cinder backend
      hosts: all
      roles:
        - { role: ansible-ha-storage }
      vars:
        ha_storage:
          openrc: /etc/openrc.sh
          cinder_volumes:
            - 55659db9-1bea-46e1-a894-4b257be72bdd
            - 548ef897-32b6-4331-99b7-4823ac6680cd
          volume_group: db
          filesystems:
            - lv: data
              size: 20G
              fstype: xfs
              mountpoint: /srv/data
            - lv: xlog
              size: 2G
              fstype: xfs
              mountpoint: /srv/xlog

    - name: Clustered filesystems and append some resources to the group
      hosts: all
      pre_tasks:
        - name: Install postgresql
          apt: name={{ item }} state=present
          with_items:
            - postgresql 
            - postgresql-common

        - name: Create postgresql resource
          shell: >
            crm configure show pgsql \
              || crm configure primitive \
                   pgsql \
                   systemd:postgresql \
                   meta target-role=Stopped
          run_once: true

      roles:
      - { role: ansible-ha-storage }

      post_tasks:
        - name: Start postgresql resource
          command: crm resource start pgsql
          run_once: true

      vars:
        ha_storage:
          openrc: /etc/openrc.sh
          cinder_volumes:
            - 55659db9-1bea-46e1-a894-4b257be72bdd
            - 548ef897-32b6-4331-99b7-4823ac6680cd
          volume_group: db
          filesystems:
            - lv: data
              size: 20G
              fstype: xfs
              mountpoint: /srv/data
            - lv: xlog
              size: 2G
              fstype: xfs
              mountpoint: /srv/xlog
          append_resources:
            - name: pgsql

A dictionnary called ha-storage is needed. Three keys are present:
* luns: a list of devices to use (the wwn and an alias to use)
* volume_group: the name of the LVM VG or the zpool to create
* filesystems: a list of volumes (name, size, fstype, mountpoint)

License
-------

GPL-3+

Author Information
------------------

Mathieu GRZYBEK
