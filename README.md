ansible-ha-storage
==================

This role is used to create and manage filesystems using corosync / pacemaker.

Requirements
------------



Role Variables
--------------

A list of mountpoints needs to be given.

Dependencies
------------

A list of other roles hosted on Galaxy should go here, plus any details in regards to parameters that may need to be set for other roles, or variables that are used from other roles.

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
