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

A goal of this role is quickly setup Satellite for Day-2 operations leveraging the foreman-ansible-modules. This role will model the [10 Steps to Build an SOE: How Red Hat Satellite 6 Supports Setting up a Standard Operating Environment](https://access.redhat.com/articles/1585273).

Products and Content Views
--------------------------

- vars/satellite_products.yml: This will define all the Red Hat products and repos.
- vars/satellite_content_views.yml: This defines all the Content and Composite views.

**Products**

Currently supported products:

- ansible
- rhel8 (broken) - you have to manually enable the repos, everything else works.
- rhel7
- osp14
- ceph3


Requirements
------------
Requires:

1. [Ansible Modules](https://github.com/theforeman/foreman-ansible-modules)
2. [Nailgun](https://github.com/SatelliteQE/#nailgun.git@master#egg=nailgun)
3. Packages: python-pip, python-six, pytz, python-netaddr
4. EPEL: provides python-pip on RHEL 7

Items 2-4 can be setup by setting the variable `configure_katello_nailgun_install`to true.

This was tested using the 

**Nailgun install manual steps on RHEL 7**

Nailgun is installed on the Satellite server.

```
sudo yum install https://dl.fedoraproject.org/pub/epel/epel-release-latest-7.noarch.rpm
sudo yum-config-manager --disable epel
sudo yum install python2-pip --enablerepo="epel"
sudo pip install git+https://github.com/SatelliteQE/nailgun.git@master#egg=nailgun
#sudo pip install apypie
sudo yum install python-six pytz python-netaddr
```
Note: This role was developed using the nailgun branch of the foreman-ansible-modules make sure you checkout this branch.

Role Variables
--------------
**Variable**|**Required**|**Default**|**Description**
-----|-----|-----|-----
rhsm_username|:x:|null|RHSM user name with priv to create manifest and attach subs on acces.redhat.com
rhsm_password|:x:|null|RHSM password
manifest_download_path|:heavy_check_mark: |/root|path on Satellite where manifest will be stored
rhsm_verify_ssl|:x:|false|"default is to use basic auth
satellite_orgs|:heavy_check_mark: |see ```defaults/main.yml```|dictionary describing your satellite configuration details
local_manifest_path|:heavy_check_mark: |null|where on the controll node to find manifest to copy to Satellite
satellite_admin_user|:heavy_check_mark: |null|satellite admin user
satellite_user_pass|:heavy_check_mark: |null|satellite admin password
satellite_url|:heavy_check_mark: |null|satellite server url https://satellite.com
satellite_verify_ssl|:x:|no|"default is to use basic auth
force_manifest_upload|:x:|false|force the upload of a manifest to Satellite

How To Use This Role
--------------------

**Products**

This role is built around enabling the Red Hat products you have a subscription entitlement to. For each product or product group listed it will:
  - enable the product repository
  - create a content view or composite content
  - promote the content view to Library
  - create corresponding activation key
  - create a sync plan

**Subscriptions**

By default actication keys are created without any associated activation keys. 
You will need to edit ```defaults/activation_keys.yml``` with your subscription details for each activation key you want created.

For conveince, the variable *default_subscription* can be set instead of editing ```defaults/activation_keys.yml```. This is useful if you have an subscription or a subscription that bundles lots of products like the one use in the example below.

Example Playbook
----------------

**Create your first ORG and upload a manifest**

This example will create and download the manifest required for Satellite. You will need a subscription pool to continue with this example.

How to find your subscription pool:

1. https://access.redhat.com/management/products
2. Click on your product
3. Select the subscriptions Tab next to Overview
4. Select the subscription number

```
- name: PLAY| setup organization and location with a manifest from RHSM and enable RHEL7 and Ansible products
  hosts: sat01
  remote_user: root
  become: false
  gather_facts: true
  vars:
    satellite_admin_user: "{{ satellite_user }}"
    satellite_user_pass: "{{ satellite_pass }}"
    satellite_url: "https://{{ satellite_hostname }}.{{ satellite_domain }}"
    satellite_verify_ssl: no
    rhsm_password: "{{ vault_rhsm_pass }}"
    rhsm_user: "{{ vault_rhsm_user }}"
    manifest_download_path: /root
    default_subscription: 
      - name: "Red Hat Cloud Suite (2-sockets), Premium"
    satellite_orgs:
      - name: MyORG
        state: present
        manifest: MyORGSatelliteManifest
        manifest_state: present
        manifest_force_upload: False
        use_local_manifest: False
        cdn_url: https://cdn.redhat.com
        pool:
          - id: 8a85f99b6977b7c0016979464ee772cb
            pool_state: present
            quantity: 7
        location:
          - name: MyLocation
            state: present
        products:
          - rhel7
          - ansible
        lifecycle_environments:
          - Library
          - Dev
          - QA
          - Production
        sync_plans:
          - name: "Red Hat Sync Plan"
            date: "2019/06/16 00:00:00 +0000"
            interval: daily
            enabled: true
            description: "Sync Plan for Red Hat products"

  tasks:
  - name: TASK| runnning ansible-role-configure-katello
    include_role:
      name: ansible-role-configure-katello
    tags: [always,manifest]
```
