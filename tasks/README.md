Role Name
=========

This role configures your Satellite:

 - Create organizations
 - Uploads manifest
 - Creates locations
 - Create sync plans
 - Create contentveiws
 - Enables products
 - Create life cycle environments

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
Set this to configure the Nailgun requirement, set to false to disable.

```
configure_katello_install_deps: true
configure_katello_nailgun_package: "git+https://github.com/SatelliteQE/#nailgun.git@master#egg=nailgun"
```

Provide connection details for Satellite. For EC2 instances ansible_fqdn pick up the internal hostname.

```
configure_katello_username: admin
configure_katello_password: "{{ your_vault_pass_variable | default('changeme') }}"
configure_katello_server_url: "https://{{ satellite_server_hostname }}"
configure_katello_verify_ssl: no
```

#### Working with manifest


- To create subscription manifest on RHSM:

```
configure_katello_subscriptions
configure_katello_copy_manifest: false
```

- To just download a existing manifest from RHSM:

```
configure_katello_download_manifest: true
configure_katello_copy_manifest: false
```

- To copy a manifest from a file system path to the satellite

```
configure_katello_copy_manifest: true
configure_katello_download_manifest: false
```

Local path to your manifest, the manifest will be copied from here to the Satellite server.

```
configure_katello_manifest_path: files/satellite_manifest
```

Force the upload of the manifest.

```
configure_katello_force_manifest_upload: true
```

#### Connections 

- Set the following if you need to use a proxy to access the internet.

```
configure_katello_proxy: http://bungabunga.com
configure_katello_proxy_username: myuser
configure_katello_proxy_password: itspassword
```

- Credentials to log into the customer portal.

```
configure_rhsm_username: myuser
configure_rhsm_password: "{{ vault_rhsm_password | default('changeme') }}"
configure_rhsm_verify_ssl: false
```

#### Organizations, Locations and Subscriptions

- state: set to absent to delete manifest
- manifest_url: url path to a manifest to download
- pool.id: this is the Pool ID for the subscription, example `8a85f99967102c1a0167488d370c6264`
- pool.quantity: quantity associated with the pool
- pool.state: absent to delete subscription
- location:

```
configure_katello_organizations:
  - name: ACME
    state: present
    manifest: acme_satellite
    manifest_url:
    pool:
      - id: 999
        state: present
        quantity: 9
    location:
      - name: Orlando
        state: present

  - name: Default Organization
    state: present
    manifest:
    manifest_url:
    pool:
      - id: 999
        state: present
        quantity: 9
    location:
      - name: Orlando
        state: present

  - name: Cloud Team
    state: present
    manifest: cloud_team_satellite
    manifest_url:
    pool:
      - id: 999
        state: present
        quantity: 9
    location:
      - name: Orlando
        state: present

  - name: Enterprise 
    state: present
    manifest: enterprise_team_satellite
    manifest_url:
    pool:
      - id: 999
        state: present
        quantity: 9
    location:
      - name: Orlando
        state: present
    products: "{{ rhel }} + {{ ansible }}"


            
```

*Locations*

This will be moved into `configure_katello_organizations.

```
configure_katello_locations:
  - name: Orlando
    state: present
    orgs:
      - ACME
```

This will be moved into configure_katello_organizations.

```
configure_katello_repository_sets:
  - name: Red Hat Enterprise Linux 7 Server (RPMs)
    repo_name: Red Hat Enterprise Linux 7 Server RPMs x86_64 7Server
    organization: ACME
    product: Red Hat Enterprise Linux Server
    state: enabled
    repositories:
      - releasever: 7Server
        basearch: x86_64
```

Configure sync plans

````
configure_katello_sync_plans:
  - name: Red Hat Sync Plan
    organization: ACME
    interval: daily
    enabled: true
    sync_date: "2017-01-01 00:00:00"
    products:
      - name: Red Hat Enterprise Linux Server
      - name: Red Hat Enterprise Linux Fast Datapath
      - name: Red Hat OpenShift Container Platform
```

Configure Lifecycle Environment

```
configure_katello_lifecycle_environment:
  - name: Testing
    organization: ACME
    prior: Library
    state: present
  - name: Production
    organization: ACME
    prior: Testing
    state: present
```
Configure Content Views

```
configure_katello_content_views:
  - name: cv-os-rhel-7server
    environment: Testing
    organization: ACME
    repositories:
      - name: Red Hat Enterprise Linux 7 Server RPMs x86_64 7Server
        product: Red Hat Enterprise Linux Server
      - name: Red Hat Satellite Tools {{ satellite_version }} for RHEL 7 Server RPMs x86_64
        product: Red Hat Enterprise Linux Server
      - name: Red Hat Enterprise Linux 7 Server Kickstart x86_64 7Server
        product: Red Hat Enterprise Linux Server
      - name: Red Hat Enterprise Linux 7 Server - RH Common RPMs x86_64 7Server
        product: Red Hat Enterprise Linux Server
    activation_key: act-testing-os-rhel-7server-x86_64
    subscriptions:
      - name: Red Hat Enterprise Linux Server with Smart Management
  - name: cv-app-openshift
    environment: Testing
    organization: ACME
    repositories:
      - name: Red Hat Enterprise Linux 7 Server RPMs x86_64 7Server
        product: Red Hat Enterprise Linux Server
      - name: Red Hat Satellite Tools {{ satellite_version }} for RHEL 7 Server #RPMs x86_64
        product: Red Hat Enterprise Linux Server
      - name: Red Hat Enterprise Linux 7 Server - Extras RPMs x86_64
        product: Red Hat Enterprise Linux Server
      - name: Red Hat Enterprise Linux Fast Datapath RHEL 7 Server RPMs x86_64 #7Server
        product: Red Hat Enterprise Linux Fast Datapath
      - name: Red Hat OpenShift Container Platform 3.6 RPMs x86_64
        product: Red Hat OpenShift Container Platform
    activation_key: openshift
    subscriptions:
       - name: Red Hat OpenShift Container Platform, Standard (1-2 Sockets)
```

Content View Filters

```
configure_katello_content_view_filters:
  - rule_name: rhel7u5
    content_view: cv-os-rhel-7server
    filter_type: rpm
    organization: ACME
    date_type:
    end_date: 2018-10-29
    errata_id:
    inclusion: True
    max_version:
    min_version:
    start_date:
    types:
    version:
    filter_state: present
    rule_state: present
    repositories:
      - name: Red Hat Enterprise Linux 7 Server RPMs x86_64 7Server
        product: Red Hat Enterprise Linux Server
      - name: Red Hat Satellite Tools {{ satellite_version }} for RHEL 7 Server RPMs #x86_64
        product: Red Hat Enterprise Linux Server
      - name: Red Hat Enterprise Linux 7 Server Kickstart x86_64 7Server
        product: Red Hat Enterprise Linux Server
      - name: Red Hat Enterprise Linux 7 Server - RH Common RPMs x86_64 7Server
        product: Red Hat Enterprise Linux Server
```

Configure Settings

```
configure_katello_settings:
 - { name: default_download_policy, value: immediate }
```

Dependencies
------------

Just the requirements.

Example Playbook
----------------
- name: PLAY| configure satellite
  hosts: localhost
  become: true
  gather_facts: yes
  vars_files:
    - configure_katello.yml

  tasks:
  - name: TASK| runnning ansible-role-configure-katello
    include_role:
      name: ansible-role-configure-katello


License
-------

BSD

Author Information
------------------

An optional section for the role authors to include contact information, or a website (HTML is not allowed).
