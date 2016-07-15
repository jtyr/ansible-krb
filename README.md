krb
===

Ansible role which helps to configure the Kerberos `krb.conf` file.

The configuration of the role is done in such way that it should not be
necessary to change the role for any kind of configuration. All can be
done either by changing role parameters or by declaring completely new
configuration as a variable. That makes this role absolutely
universal. See the examples below for more details.

Please report any issues or send PR.


Examples
--------

```
---

- name: Example of how use the krb role
  hosts: all
  vars:
    # Custom domain
    krb_domain: example.cloud.local
    # Custom server name in that domain
    krb_server_name: kerberos01
    # Add .hostname as the first domain realm
    krb_config_domain_realm__pre: .{{ ansible_hostname }} = {{ krb_domain | upper }}
  roles:
    - krb
```


Role variables
--------------

```
# Package to be installed (explicit version can be specified here)
krb_pkg: krb5-libs

# Location of the config file
#krb_config_file: /etc/krb5.conf
krb_config_file: /tmp/krb.conf

# Kerberos domain
krb_domain: example.com

# Kerberos server name
krb_server_name: kerberos

# Kerberos server
krb_server: "{{ krb_server_name ~ '.' ~ krb_domain | lower }}"


# Values of the default Logging section of the krb config
krb_config_logging_default: FILE:/var/log/krb5libs.log
krb_config_logging_kdc: FILE:/var/log/krb5kdc.log
krb_config_logging_admin_server: FILE:/var/log/kadmind.log

# Default Logging section of the krb config
krb_config_logging__default:
  default: "{{ krb_config_logging_default }}"
  kdc: "{{ krb_config_logging_kdc }}"
  admin_server: "{{ krb_config_logging_admin_server }}"

# Cutom Logging section of the krb config
krb_config_logging__custom: {}

# Final Logging section of the krb config
krb_config_logging: "{{
  krb_config_logging__default.update(krb_config_logging__custom) }}{{
  krb_config_logging__default }}"


krb_config_libdefaults_default_realm: "{{ krb_domain | upper }}"
krb_config_libdefaults_dns_lookup_realm: "false"
krb_config_libdefaults_dns_lookup_kdc: "false"
krb_config_libdefaults_ticket_lifetime: 24h
krb_config_libdefaults_renew_lifetime: 7d
krb_config_libdefaults_forwardable: "true"

# Default Lib Defaults section of the krb config
krb_config_libdefaults__default:
  default_realm: "{{ krb_config_libdefaults_default_realm }}"
  dns_lookup_realm: "{{ krb_config_libdefaults_dns_lookup_realm }}"
  dns_lookup_kdc: "{{ krb_config_libdefaults_dns_lookup_kdc }}"
  ticket_lifetime: "{{ krb_config_libdefaults_ticket_lifetime }}"
  renew_lifetime: "{{ krb_config_libdefaults_renew_lifetime }}"
  forwardable: "{{ krb_config_libdefaults_forwardable }}"

# Custom Lib Defaults section of the krb config
krb_config_libdefaults__custom: {}

# Final Lib Defaults section of the krb config
krb_config_libdefaults: "{{
  krb_config_libdefaults__default.update(krb_config_libdefaults__custom) }}{{
  krb_config_libdefaults__default }}"


# Default realms domain
krb_config_realms_domain__default:
  kdc: "{{ krb_server }}"
  admin_server: "{{ krb_server }}"

# Custom realms domain
krb_config_realms_domain__custom: {}

# Final realms domain
krb_config_realms_domain: "{{
  krb_config_realms_domain__default.update(krb_config_realms_domain__custom) }}{{
  krb_config_realms_domain__default | encode_ini(delimiter=' = ') | trim | comment(prefix='{', postfix='}', decoration='  ') }}"


# Unfortunately the realm "EXAMPLE.COM" can not be parametrized as it should be
# the name of the option which is the dict key. So we fake it a bit like this.
krb_config_realms_item: "{{ krb_domain | upper }} = {{ krb_config_realms_domain }}"

# Default Realms section of the krb config
krb_config_realms__default:
  "#": "#\n{{ krb_config_realms_item }}"

# Custom Realms section of the krb config
krb_config_realms__custom: {}

# Final Realms section of the krb config
krb_config_realms: "{{
  krb_config_realms__default.update(krb_config_realms__custom) }}{{
  krb_config_realms__default }}"


# Default part of the Realms section of the krb config
krb_config_domain_realm__default: |-
  .{{ krb_domain | lower }} = {{ krb_domain | upper }}
  {{ krb_domain | lower }} = {{ krb_domain | upper }}

# Custom part of the Realms section of the krb config which goes in front of
# the __default
krb_config_domain_realm__pre: ""

# Custom part of the Realms section of the krb config which goes behind
# the __default
krb_config_domain_realm__post: ""

# Final Realms section of the krb config
krb_config_domain_realm:
  # Unfortunately we can not be parametrized this bit with key/value pairs as
  # Ansible doesn't allow to have variable in place of the dict key. So we fake
  # it a bit like this.
  "#": "#\n{{
    krb_config_domain_realm__default
      if krb_config_domain_realm__pre == '' and krb_config_domain_realm__post == '' else
    [krb_config_domain_realm__pre, krb_config_domain_realm__default] | join('\n')
      if krb_config_domain_realm__post == '' else
    [krb_config_domain_realm__default, krb_config_domain_realm__post] | join('\n')
      if krb_config_domain_realm__pre == '' else
    [krb_config_domain_realm__pre, krb_config_domain_realm__default, krb_config_domain_realm__post] | join('\n') }}"


# Default krb config
krb_config__default:
  logging: "{{ krb_config_logging }}"
  libdefaults: "{{ krb_config_libdefaults }}"
  realms: "{{ krb_config_realms }}"
  domain_realm: "{{ krb_config_domain_realm }}"

# Custom krb config
krb_config__custom: {}

# Final krb config
krb_config: "{{
  krb_config__default.update(krb_config__custom) }}{{
  krb_config__default }}"
```


Dependencies
------------

- [`config_encoder_filters`](https://github.com/jtyr/ansible-config_encoder_filters)


License
-------

MIT


Author
------

Jiri Tyr
