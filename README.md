ansible-role-configure-katello
===============================

:warning: Work in progress.

This role configures your Satellite:

 - Create organizations
 - Uploads manifest
 - Creates locations
 - Create sync plans
 - Create contentveiws
 - Enables products
 - Create life cycle environments

It's base on work I started here [my-satellite-post-config](https://github.com/flyemsafe/my-satellite-post-config)


Requirements
------------
Requires:

1. [Ansible Modules](https://github.com/theforeman/foreman-ansible-modules)
2. [Nailgun](https://github.com/SatelliteQE/#nailgun.git@master#egg=nailgun)
3. Packages: python-pip, python-six, pytz, python-netaddr
4. EPEL: provides python-pip on RHEL 7

Items 2-4 can be setup by setting the variable `configure_katello_nailgun_install` to true.

**Nailgun install manual steps on RHEL 7**

Nailgun is installed on the Satellite server.

```
sudo yum install https://dl.fedoraproject.org/pub/epel/epel-release-latest-7.noarch.rpm
sudo yum-config-manager --disable epel
sudo yum install python2-pip --enablerepo="epel"
sudo pip install git+https://github.com/SatelliteQE/nailgun.git@master#egg=nailgun
sudo yum install python-six pytz python-netaddr
```


Role Variables
--------------

See defaults/main.yml file 

Example Playbook
----------------

Including an example of how to use your role (for instance, with variables passed in as parameters) is always nice for users too:

    - hosts: servers
      roles:
         - { role: sean797.configure_katello }
