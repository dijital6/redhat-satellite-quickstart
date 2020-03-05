Red Hat Satellite 6 Quick Start
===============================

This role will configure your Satellite server using the foreman-ansible-modules. It models the [10 Steps to Build an SOE: How Red Hat Satellite 6 Supports Setting up a Standard Operating Environment](https://access.redhat.com/articles/1585273).

This role does the following:

 - Create your organization(s)
 - Adds a manfiest to the organization
 - Creates locations
 - Create a sync plans
 - Create content veiws
 - Enables Red Hat products
 - Create life cycle environments


Products and Content Views
--------------------------

- vars/satellite_products.yml: This will define all the Red Hat products and repos.
- vars/satellite_content_views.yml: This defines all the Content and Composite views.

**Products**

Currently supported products:

- ansible
- rhel8
- rhel7
- osp14
- ceph3

Requirements
------------
Requires:

- [Ansible Modules](https://github.com/theforeman/foreman-ansible-modules)

**Install the Foreman Ansible Modules collection on the control node**

Currently using the collections via Copr.

```
sudo wget https://copr.fedorainfracloud.org/coprs/evgeni/foreman-ansible-modules/repo/epel-7/ -O /etc/yum.repos.d/evgeni-foreman-ansible-modules-epel-7.repo
sudo yum install -y ansible-collection-theforeman-foreman
```

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

**Installation**

```
ansible-galaxy install git+https://github.com/flyemsafe/satellite-day-two-ops.git
```
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
  hosts: sat01
  remote_user: root
  become: true
  gather_facts: true
  vars:
    satellite_admin_user: admin
    satellite_user_pass: password
    satellite_url: https://sat01.example.com
    satellite_verify_ssl: no
    rhsm_password: username
    rhsm_user: password
    manifest_download_path: /root
    local_manifest_path: ""
    default_subscription:
      - name: "Red Hat Cloud Suite (2-sockets), Premium"
    satellite_orgs:
      - name: CloudTeam
        state: present
        manifest: ""
        manifest_state: present
        manifest_force_upload: False
        sync_right_away: True
        use_local_manifest: False
        cdn_url: https://cdn.redhat.com
        wait_for_respo_sync: false
        pool:
          - id: 8a85f99b6977b7c0016979464ee772cb
            pool_state: present
            quantity: 7
        location:
          - name: Clearwater
            state: present
        products:
          - rhel7
          - rhel8
          - ansible
        lifecycle_environments:
          - name: Dev
            prior: Library
            state: present
            description: Dev environment
          - name: QA
            prior: Dev
            state: present
            description: QA environment
          - name: Production
            prior: QA
            state: present
            description: Production environment
        sync_plans:
          - name: "Red Hat Sync Plan"
            date: "2019/10/09 00:00:00 +0000"
            interval: daily
            enabled: true
            description: "Sync Plan for Red Hat products"
        domains:
          - name: example.com
            locations:
              - Clearwater
            organizations:
              - ACME
              - CloudTeam
            state: present
            description: example.com
        subnets:
          - name: lunchnet
            locations:
              - Clearwater
            organizations:
              - CloudTeam
            network: 192.24.24.0
            mask: 255.255.255.0
            gateway: 192.24.24.1
            from_ip: 192.24.24.200
            to_ip: 192.24.24.205
            domains:
              - example.com
            state: present
            description: 192.24.24.0
        compute_resource:
          - name: lunchnet
            locations:
              - Clearwater
            organizations:
              - CloudTeam
            state: present
            provider: libvirt
            provider_params:
              url: qemu+ssh://root@kvmhost/system
              display_type: vnc
            description: KVM Compute Host
  collections:
    - theforeman.foreman

  tasks:
  - name: TASK| runnning ansible-role-configure-katello
    include_role:
      name: satellite-day-two-ops
    tags: [always,manifest]
```
