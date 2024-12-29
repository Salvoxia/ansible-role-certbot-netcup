# Ansible Role: cerbot_netcup

An [Ansible](https://www.ansible.com) role to install [Certbot](https://certbot.eff.org/) and issue certificates with the DNS01 challenge by using Netcup's DNS servers.


## Requirements

- Netcup DNS setup
- Netcup API Key and Password

## Dependencies

- [geerlingguy.pip](https://galaxy.ansible.com/ui/standalone/roles/geerlingguy/pip/)
- [geerlingguy.certbot](https://galaxy.ansible.com/ui/standalone/roles/geerlingguy/certbot/) (or some fork of it)
- [community.general](https://galaxy.ansible.com/ui/repo/published/community/general/)

Install dependencies with
```shell
ansible-galaxy install -r requirements.yml
```

## Role Variables

```yaml
# Netcup customer ID
certbot_netcup_customer_id: ""
```
The Netcup Customer ID

```yaml
# Netcup API Key
certbot_netcup_api_key: ""
```

The Netcup API Key

```yaml
# Netcup API Password
certbot_netcup_api_password: ""
```

The Netcup API Password

```yaml
# Path where to store netcup credentials file
certbot_netcup_credentials_path: "/etc/letsencrypt/netcup_credentials.ini"
```

The path where to store the Netcup credentials for the Certbot DNS plugin. Usually it's not required to change this.

```yaml
# DNS propagation time for DNS plugin to wait until trying to verify the DNS challenge
certbot_netcup_propagation_seconds: 1200
```

The time to wait in seconds after creating the DNS TXT records for the new records to propagate so they can be verified.

```yaml
certbot_certs:
  - email: "admin@example3.com"
    domains:
      - *.example3.com
```

The primary reason for using the `DNS01` challenge is to provision wildcard certificates. But of course this method can be used for non-wildcard certificates as well, if no webserver should be spun up.

```yaml
certbot_cloudflare_acme_server: "{{ certbot_cloudflare_acme_test }}"
```
or 
```yaml
certbot_cloudflare_acme_server: "{{ certbot_cloudflare_acme_live }}"
```
    
Let's Encrypt server to use, defaults to test.


```yaml
# The base role for installing and configuring certbot. This role was developed
# with geerlingguy.certbot v5.2.1, so any fork using the same role variable names and providing the task "create-cert-standalone.yml"
# should work.
certbot_netcup_certbot_base_role: "geerlingguy.certbot"
```

The role to use for installing certbot in the first place.
This role re-uses the following role variables from `geerlingguy.certbot`:
- `certbot_create_standalone_stop_services`
- `certbot_create_if_missing` 
- `certbot_dir`
- `certbot_install_method`

This role expects the certbot role to build the certbot `create` command the same way `geerlingguy.certbot` does. It then explicitly calls
the certbot rule's `create-cert-standalone.yml` task if  `certbot_create_if_missing` is set to `true`.
If `certbot_install_method` is set to `snap`, the Netcup DNS plugin will be installed using Snap as well, otherwise it is installed using pip.

Example Playbook
----------------

Including an example of how to use your role (for instance, with variables passed in as parameters) is always nice for users too:
```yaml
- name: "Install Certbot and issue wildcard certificate"
  hosts: servers
  vars:
    certbot_netcup_customer_id: "123456"
    certbot_netcup_api_key: "0123456789abcdef0123456789abcdef01234567"
    certbot_netcup_api_password: "abcdef0123456789abcdef01234567abcdef0123"
    certbot_admin_email: "mail@example3.com"
    # Issue the certificate as part of the playbook
    certbot_create_if_missing: true
    certbot_certs:
        - domains:
          - *.example3.com
  roles:
      - salvoxia.certbot-netcup
```

## License

MIT


## Author Information

This role was created by Salvoxia (salvoxia@blindfish.info).
