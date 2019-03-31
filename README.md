## StrongSwan role

This role uses a [strongswan.conf-style syntax](https://wiki.strongswan.org/projects/strongswan/wiki/Swanctlconf) (referencing sections, since 5.7.0).

#### Variables

##### General

* `strongswan_swanctl_settings`: [required]: Set all settings for swanctl.conf

##### SrtongSwan (server|clients) settings:

* `strongswan_swanctl_config_dir`: [optional, default: `/etc/strongswan/swanctl`]: Directory which contains StrongSwan swanctl.conf
* `strongswan_swanctl_config_file`: [optional, default: `{{ strongswan_swanctl_config_dir }}/swanctl.conf`]: The name of StrongSwan swanctl configuration file
* `strongswan_letsencrypt_enable`: [optional, default: `true`]: Prepare StrongSwan server certificate with Let'sEncrypt
* `strongswan_firewalld_enable`: [optional, default `true`]: Set all needed firewall rules for StrongSwan server
* `strongswan_client`: [optional, default `false`]: Install and configure StrongSwan on clients
* `strongswan_download_cert`: [optional, default `false`]: Download StrongSwan certificate from server host. Should be used with `strongswan_letsencrypt_enable` = `true`
* `strongswan_upload_cert`: [optional, default `false`]: Upload StrongSwan certificate to clients. Should be used with `strongswan_letsencrypt_enable` = `true`

## Dependencies

 - geerlingguy.repo-epel
 - geerlingguy.certbot
 - shhirose.firewalld

#### Example

##### Basic example

Ansible inventory:  

```
[server]
moon.strongswan.org

[clients]
carol.strongswan.org
```  

Host vars for moon.strongswan.org:

```yaml
# Allow traffic for Let'sEncrypt certbot and StrongSwan
shhirose_firewalld:
  services:
    - service: http
      zone: public
      immediate: yes
      permanent: True
      state: enabled
    - service: ipsec
      zone: public
      immediate: yes
      permanent: True
      state: enabled
  ports:
    - port: "500/udp"
      zone: public
      immediate: yes
      permanent: True
      state: enabled
    - port: "4500/udp"
      zone: public
      immediate: yes
      permanent: True
      state: enabled
  masquerades:
    - masquerade: yes
      zone: public
      immediate: yes
      permanent: True


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
    rw:
      unique: replace
      version: 2
      send_certreq: "no"
      local_addrs: "{{ ansible_default_ipv4.address }}"
      children:
        net:
          local_ts: 192.168.2.0/24
          #updown: "/usr/libexec/strongswan/_updown iptables"
      local:
        auth: pubkey
        certs: Cert.pem
        id: moon.strongswan.org
      remote: 
        auth: eap-mschapv2
        eap_id: "%any"

  secrets:
    eap-carol:
      id: carol
      secret: Ar3etTnp
```  

Host vars for carol.strongswan.org:
```yaml
# Set StrongSwan client only options
strongswan_client: true
strongswan_letsencrypt_enable: false
strongswan_firewalld_enable: false

# Set StrongSwan swanctl configuration
strongswan_swanctl_settings:
  connections:
    home:
      remote_addrs: moon.strongswan.org
      local_addrs: 192.168.10.1
      local:
        auth: eap
        eap_id: carol
      remote:
        auth: pubkey
        id: moon.strongswan.org
      children:
        home:
          remote_ts: 192.168.2.0/24
          #updown: "/usr/libexec/strongswan/_updown iptables"
          start_action: start
      version: 2
      #encap: yes

  secrets:
    eap-carol:
      id: carol
      secret: Ar3etTnp
```

StrongSwan server playbook:

```yaml
- hosts: server
  roles:
    - serverbee.strongswan
```

StrongSwan clients playbook:

```yaml
- hosts:
    - server
  roles:
    - serverbee.strongswan
  vars:
    strongswan_client: true
    strongswan_download_cert: true

- hosts:
    - clients
  roles:
    - serverbee.strongswan
  vars:
    strongswan_upload_cert: true
```

#### Todo

1. Start using firewalld direct module option after merging pull request: https://github.com/ansible/ansible/pull/49514.
   It needs for skiping NAT for ipsec packets with a matching IPsec policy.
2. Remove some dependencies roles to achieve more flexibility.

#### License

BSD 2-clause "Simplified" license

#### Author Information

Vitaly Yakovenko
