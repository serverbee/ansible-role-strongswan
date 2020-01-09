## StrongSwan role
[![Build Status](https://travis-ci.org/hybridadmin/ansible-role-strongswan-swanctl.svg?branch=master)](https://travis-ci.org/hybridadmin/ansible-role-strongswan-swanctl)

Tested on Ubuntu 18.04 and CentOS 7.

This role uses a [strongswan.conf-style syntax](https://wiki.strongswan.org/projects/strongswan/wiki/Swanctlconf) (referencing sections, since 5.7.0).

#### Variables

##### General

* `strongswan_swanctl_settings`: [required]: Set all settings for swanctl.conf

##### StrongSwan (server|clients) settings:

* `strongswan_swanctl_config_dir`: [optional, default: `/etc/strongswan/swanctl`]: Directory which contains StrongSwan swanctl.conf
* `strongswan_swanctl_config_file`: [optional, default: `{{ strongswan_swanctl_config_dir }}/swanctl.conf`]: The name of StrongSwan swanctl configuration file
* `strongswan_letsencrypt_enable`: [optional, default: `true`]: Prepare StrongSwan server certificate with Let'sEncrypt
* `strongswan_firewalld_enable`: [optional, default `true`]: Set all needed firewall rules for StrongSwan server
* `strongswan_client`: [optional, default `false`]: Install and configure StrongSwan on clients
* `strongswan_download_cert`: [optional, default `false`]: Download StrongSwan certificate from server host. Should be used with `strongswan_letsencrypt_enable` = `true`
* `strongswan_upload_cert`: [optional, default `false`]: Upload StrongSwan certificate to clients. Should be used with `strongswan_letsencrypt_enable` = `true`

## Dependencies

 - geerlingguy.certbot

#### Example

##### Basic example

Host vars for StrongSwan Server:

```yaml
# Enable variable bellow only for first certificate generating
certbot_create_if_missing: yes

certbot_certs:
 - email: email@example.com
   domains:
     - "{{ inventory_hostname }}"

certbot_create_standalone_stop_services: []


# Set StrongSwan swanctl configuration
strongswan_swanctl_settings:
  connections:
    ikev2-eap:
      version: 2
      proposals: aes192gcm16-aes128gcm16-prfsha256-ecp256-ecp521,aes192-sha256-modp3072,default
      rekey_time:  0s
      encap: yes
      pools: primary-pool-ipv4
      fragmentation: yes
      dpd_delay: 30s
      local-1:
        certs: cert.pem
        id: myid
      remote-1:
        auth: eap-dynamic
        eap_id: %any
      children:
        ikev2-eap:
          local_ts: 0.0.0.0/0,::/0
          rekey_time: 0s
          dpd_action: clear
          esp_proposals: aes192gcm16-aes128gcm16-prfsha256-ecp256-modp3072,aes192-sha256-ecp256-modp3072,default
  pools:
    primary-pool-ipv4:
      addrs: 192.168.252.0/24
      dns: 1.1.1.1, 9.9.9.9
  secrets:
    eap-user:
      id: user
      secret: Ar3e73tTnp02
```  

StrongSwan server playbook:

```yaml
- hosts: server
  roles:
    - hybridadmin.strongswan_swanctl
```

StrongSwan clients playbook:

```yaml
- hosts:
    - server
  roles:
    - hybridadmin.strongswan_swanctl
  vars:
    strongswan_client: true
    strongswan_download_cert: true

- hosts:
    - clients
  roles:
    - hybridadmin.strongswan_swanctl
  vars:
    strongswan_upload_cert: true
```

#### Todo
1. Remove some dependencies roles to achieve more flexibility.

#### License

BSD 2-clause "Simplified" license

#### Author Information

hybridadmin
