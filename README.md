## StrongSwan role

[![CI](https://github.com/hybridadmin/ansible-role-strongswan/actions/workflows/build.yml/badge.svg?branch=master)](https://github.com/hybridadmin/ansible-role-strongswan/actions/workflows/build.yml)

Tested on:
* Ubuntu 18.04 and 20.04
* CentOS 7 and 8
* Debian 9 and 10

This role uses a [swanctl.conf-style syntax](https://wiki.strongswan.org/projects/strongswan/wiki/Swanctlconf) (referencing sections, since 5.7.0).

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

## Variables

### Host vars for StrongSwan Server:

```yaml
# Enable variable below only for first certificate generating
certbot_create_if_missing: yes

certbot_certs:
 - email: email@example.com
   domains:
     - "{{ inventory_hostname }}"

certbot_create_standalone_stop_services: []
```

```yaml
# Set StrongSwan swanctl configuration
# https://wiki.strongswan.org/projects/strongswan/wiki/UsableExamples
strongswan_swanctl_settings:
  connections:
    ikev2-eap:
      version: 2
      rekey_time:  0s
      fragmentation: yes
      proposals: aes192gcm16-aes128gcm16-prfsha256-ecp256-ecp521,aes192-sha256-modp3072,default
      encap: yes
      pools: primary-pool-ipv4
      dpd_delay: 30s
      local-1:
        certs: cert.pem
        id: myid
      remote-1:
        auth: eap-dynamic
        eap_id: "%any"
      children:
        ikev2-eap:
          local_ts: 0.0.0.0/0,::/0
          rekey_time: 0s
          dpd_action: clear
          esp_proposals = aes192gcm16-aes128gcm16-prfsha256-ecp256-modp3072,aes192-sha256-ecp256-modp3072,default
  pools:
    primary-pool-ipv4:
      addrs: 192.168.252.0/24
  secrets:
    eap-user:
      id: user
      secret: Ar3e73tTnp02
```

### Host vars for carol.strongswan.org:
```yaml
strongswan_swanctl_settings:
  connections:
    home:
      encap: yes
      vips: 0.0.0.0
      remote_addrs: moon.strongswan.org
      version: 2
      children:
        home:
          remote_ts: 0.0.0.0/0,::/0
          start_action: none
      local:
        auth: eap-aka
        eap_id: carol
      remote:
        auth: pubkey
        id: moon.strongswan.org
  secrets:
    eap-carol:
      id: carol
      secret: Ar3etTnp
```

## Playbook Examples


### StrongSwan server with PKI certificates playbook:

```yaml
- hosts: server
  vars:
    strongswan_letsencrypt_enable: false
  roles:
    - hybridadmin.strongswan
```

### StrongSwan server with Letsencrypt certificates playbook:

```yaml
- hosts: server
  vars:
    strongswan_letsencrypt_enable: true
    certbot_certs:
      - email: janedoe@example.com
        domains:
          - vpn.example.com
  roles:
    - hybridadmin.strongswan
```

### StrongSwan clients playbook:

```yaml
- hosts:
    - server
  roles:
    - hybridadmin.strongswan
  vars:
    strongswan_client: true
    strongswan_download_cert: true

- hosts:
    - clients
  roles:
    - hybridadmin.strongswan
  vars:
    strongswan_upload_cert: true
```

#### Todo
1. Remove some dependencies roles to achieve more flexibility.

#### License

BSD 2-clause "Simplified" license

#### Author Information

hybridadmin
