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
        ha-storage:
          luns:
            - id: 23234
              alias: lun-dc-1
            - id: 45345
              alias: lun-dc-2
          volume_group: db
          filesystems:
            - lv: data
              size: 20
              fstype: xfs
              mountpoint: /srv/data
            - lv: xlog
              size: 2
             fstype: xfs
             mountpoint: /srv/xlog

A dictionnary called ha-storage is needed. Three keys are present:
* luns: a list of devices to use (the wwn and an alias to use)
* volume_group: the name of the LVM VG to create
* filesystems: a list of volumes (name, size, fstype, mountpoint)

License
-------

GPL-3+

Author Information
------------------

Mathieu GRZYBEK
