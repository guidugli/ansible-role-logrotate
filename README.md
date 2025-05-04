Ansible Role: logrotate
=========

An Ansible Role that install and configure logrotate on Fedora and Debian/Ubuntu.

Requirements
------------

No requirements.

Role Variables
--------------

**Available variables are listed below, along with default values (see defaults/main.yml):**

    #logrotate_config_files: 
    #   - chrony

Specify the files to be copied to logrotate.d directory. Add the file to be copied into the "files"
ansible folder, uncomment the variable and add the filename(s). 

    logrotate_compress: no

Set to true/yes to set default configuration to compress rotated files

    logrotate_use_date_extension: yes

Set to true/yes to use date extension on file name, instead of just numbers

    logrotate_weeks2keep: 24

Set the number of weeks to keep rotated logs

**The variables listed below do not need to be changed for targeted systems (see vars/main.yml):**

    logrotate_packages:

Packages to install logrotate.

    logrotate_conf:

Configuration file location.

Dependencies
------------

No Dependencies.

Example Playbook
----------------

    - hosts: servers
      roles:
         - { role: guidugli.logrotate }

License
-------

MIT / BSD

Author Information
------------------

This role was created in 2020 by Carlos Guidugli.
