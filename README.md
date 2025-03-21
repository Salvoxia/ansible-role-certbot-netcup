# Ansible Role: certbot_netcup
[![Build Status](https://img.shields.io/github/actions/workflow/status/salvoxia/ansible-role-certbot-netcup/ci.yml?label=molecule&logo=ansible&style=flat-square)](https://github.com/Salvoxia/ansible-role-certbot-netcup/actions/workflows/ci.yml)
[![GitHub tag (latest by date)](https://img.shields.io/github/v/tag/salvoxia/ansible-role-certbot-netcup?color=EE0000&label=release&logo=ansible&style=flat-square)](https://galaxy.ansible.com/ui/standalone/roles/salvoxia/certbot_netcup/)
[![Ansible Galaxy Downloads](https://img.shields.io/badge/dynamic/json?color=blueviolet&label=Galaxy%20Downloads&logo=ansible&style=flat-square&query=%24.download_count&url=https%3A%2F%2Fgalaxy.ansible.com%2Fapi%2Fv1%2Froles%2F39857%2F%3Fformat%3Djson)](https://galaxy.ansible.com/ui/standalone/roles/salvoxia/certbot_netcup/)
[![MIT LIcense](https://img.shields.io/github/license/salvoxia/ansible-role-certbot-netcup?style=flat-square)](https://github.com/Salvoxia/ansible-role-certbot-netcup/blob/main/LICENSE)

An [Ansible](https://www.ansible.com) role that determines how Certbot is installed (via `snap` or via `pip`), installs the [certbot-dns-netcup](https://pypi.org/project/certbot-dns-netcup/) authenticator using the appropriate way (`pip` or `snap`) and optionally issues certificates using the DNS01 challenge and Netcup's DNS servers.

## Requirements

- Netcup DNS setup
- Netcup API Key and Password
- Certbot installed (e.g. via [geerlingguy.certbot](https://galaxy.ansible.com/ui/standalone/roles/geerlingguy/certbot/))

## Dependencies

- [geerlingguy.pip](https://galaxy.ansible.com/ui/standalone/roles/geerlingguy/pip/)
- [community.general](https://galaxy.ansible.com/ui/repo/published/community/general/)

Install dependencies with
```shell
ansible-galaxy install -r requirements.yml
```

## Disclaimer

This role was inspired by [ansible-certbot-cloudflare](https://github.com/michaelpporter/ansible-role-certbot-cloudflare) by [Michael Porter](https://www.michaelpporter.com/).

The task for generating certificates from Ansible was inspired by [geerlingguy.certbot](https://galaxy.ansible.com/ui/standalone/roles/geerlingguy/certbot/).


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
# Decide whether certificates should be created in the play
certbot_netcup_create_if_missing: false
```
Flag indicating whether certificates should be created as part of the playbook. The following role variables only have any effect if set to `true`:
  - `certbot_netcup_certs`
  - `certbot_netcup_admin_email`
  - `certbot_netcup_testmode`
  - `certbot_netcup_hsts`
  - `certbot_netcup_create_extra_args`

```yaml
# Global email address used for ACME account creation if non is set per domain.
# Any one of these must be set.
certbot_netcup_admin_email: ""
```
Global mail address used for ACME account creation and certificate notifications. It is possible to set domain-specific mail addresses when defining the domains to create certificates for, which override this global address.
If `certbot_netcup_create_if_missing` is `true`, either `certbot_netcup_admin_email` must be set to a valid mail address, or an email address must be provided for each domain explicitly.

```yaml
# List of certificates to create. Each entry in the dictionary will result in a separate certificate
# based on the first domain in the list. For each domain a separate admin address may be specified. If
# none is specified, certbot_netcup_admin_email is used.
certbot_netcup_certs: []
  # - domains:
  #     - local.example.com
  #     - *.local.example.com
  #   email: admin@example.com
  #   services:
  #     - nginx
  #   pre_hook_additional_content: ""
  #   post_hook_additional_content: ""
  #   deploy_hook_additional_content: |
  #     postfix reload
  # - domains:
  #     - example2.com
```

The primary reason for using the `DNS01` challenge is to provision wildcard certificates. But of course this method can be used for non-wildcard certificates as well, if no webserver should be spun up.  

`certbot_netcup_certs` is a list of domain specifictions where each entry must at least have the `domains` attribute, listing all domains for which to request a certificate.  
One certificate file will be created per entry in `certbot_netcup_certs`.

`email` is an optional attribute only if `certbot_netcup_admin_email` is set. Otherwise it is mandatory.

`services` is a list of services to stop before certificate renewal and start again after. From these services a shell script as `pre` and `post` hook is created.

`pre_hook_additional_content`, `post_hook_additional_content`, `deploy_hook_additional_content` are strings that represent additional content to be written to the corresponding hook shell scripts. This gives the flexibility to perform tasks other than starting or stopping services. The example above reloads the Postfix configfuration to refresh TLS certificates after renewal without Postfix downtime.

```yaml
certbot_cloudflare_acme_server: "{{ certbot_cloudflare_acme_test }}"
```
or 
```yaml
certbot_cloudflare_acme_server: "{{ certbot_cloudflare_acme_live }}"
```
    
Let's Encrypt server to use, defaults to test.


```yaml
# Extra arguments to pass to certbot when creating a certificate
certbot_netcup_create_extra_args: ""
```
Additional arguments to pass to `certbot` when creating certificates.

```yaml
# Passes --test-cert to certbot if true
certbot_netcup_testmode: false
```
Enable test mode to only run a test request without actually creating certificates.

```yaml
# Passes --hsts to certbot if true
certbot_netcup_hsts: false
```
Enable (HTTP Strict Transport Security) for the certificate generation.

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
    certbot_netcup_admin_email: "mail@example.com"
    # Issue the certificate as part of the playbook
    certbot_netcup_create_if_missing: true
    certbot_netcup_certs:
        - domains:
          - "*.example.com"
  roles:
      - salvoxia.certbot-netcup
```

## License

MIT


## Author Information

This role was created by Salvoxia (salvoxia@blindfish.info).
