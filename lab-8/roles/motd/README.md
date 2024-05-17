Role Name
=========

Changes the motd of managed hosts to lab standards

Requirements
------------

none

Role Variables
--------------

|Variable|Description|
|---|---|
|`admin_email`|E-Mail for contacting system administrator|
 

Dependencies
------------

none

Example Playbook
----------------
    - name: use motd role
      hosts: servers
      vars:
        - admin_email: sysadmin@example.com
      roles:
         - role: motd

License
-------

BSD

Author Information
------------------

Written by <bernhard.gueth@fujitsu.com>
