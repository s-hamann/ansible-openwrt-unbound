OpenWrt Unbound
===============

This role configures [Unbound](https://unbound.net/) as a DNS resolver and/or DNS server on [OpenWrt](https://www.openwrt.org/) targets.

Requirements
------------

This role has no special requirements on the controller.

It does, however, require a working [Python](https://www.python.org/) installation on the target system or [gekmihesg's Ansible library for OpenWrt](https://github.com/gekmihesg/ansible-openwrt) on the Ansible controller.

Role Variables
--------------

* `unbound_luci`  
  Whether to install the LuCI app for Unbound.
  Default is `true`.
* `unbound_luci_i18n`  
  A list of languages to install for the LuCI app, e.g. `en` or `pt-br`.
  Refer to the available OpenWrt packages (`luci-i18n-unbound-*`) for a list of valid language codes.
  Ignored if `unbound_luci` is `false`.
  Optional.
* `unbound_uci_config_main`  
  A dictionary of UCI configuration options for Unbound.
  Dictionary keys are option names and the values are the respective configuration values.
  See https://github.com/openwrt/packages/blob/master/net/unbound/files/README.md for valid settings.
* `unbound_uci_config_zones`  
  A dictionary of UCI zone configuration options for Unbound.
  Dictionary keys are zone names and values are in turn dictionaries that describe the zone configuration.
  Of these dictionaries, the keys are UCI option names and value the respective configuration values.
  See https://github.com/openwrt/packages/blob/master/net/unbound/files/README.md for valid settings.
* `unbound_prefetch_root_zone`  
  Whether to cache the root zone all at once to speed up recursion.
  Default is `true`.
* `unbound_extra_options`  
  A dictionary of Unbound configuration options to set.
  Dictionary keys are clause names, keys are in turn dictionaries containing the settings for this clause.
  Their keys are option names and values their respective values.
  Refer to the [Unbound documentation](https://unbound.docs.nlnetlabs.nl/en/latest/) for options and their meaning.
  Note: Clause names are truncated at the first space character.
  This allows defining multiple clauses with the same name.
  Note: Values that contain a space are automatically quoted.
  This is not desired for options that expect multiple values (e.g. `access-control`).
  For these options, a (pair of) `"` should be included in the value, which prevent further quoting.
  If `manual_conf` is enabled in `unbound_uci_config_main`, a few recommended settings are added to this and the configuration is written to `/etc/unbound/unbound.conf`.
  Otherwise, the configuration is split between `/etc/unbound/unbound_srv.conf` and `/etc/unbound/unbound_ext.conf`.
  Optional.

Dependencies
------------

This role does not depend on any specific roles.

However, it is to note that this role only handles Unbound.
In the default setup, Unbound conflicts with Dnsmasq, since they would both bind to UDP port 53.
It is the responsibility of another role to reconfigure or disable Dnsmasq, if Unbound is set to use this port.

Example Configuration
---------------------

The following is a short example for some of the configuration options this role provides:

```yaml
unbound_uci_config_main:
  domain: example.com
  validator: true
  rebind_protection: 2
  root_age: 42
unbound_uci_config_zones:
  fwd_cloudflare:
    fallback: true
    tls_index: cloudflare-dns.com
    tls_upstream: true
    zone_type: forward_zone
    server:
      - 1.1.1.1
      - 1.0.0.1
      - 2606:4700:4700::1111
      - 2606:4700:4700::1001
    zone_name: .
unbound_prefetch_root_zone: false
unbound_extra_options:
  server:
    interface: 0.0.0.0@853
    tls-service-pem: /etc/unbound.pem
    tls-service-crt: /etc/unbound.key
    local-zone:
      - '"example.org" always_nxdomain'
    local-data:
      - 'host1.example.com. 3600 IN A 192.168.1.2'
      - 'host2.example.com. 3600 IN A 192.168.1.3'
    access-control-view:
      - '192.168.1.0/24 "server-net"'
      - '192.168.2.0/24 "client-net"'
  view (server-net):
    name: server-net
    local-data: 'gateway.example.com. 3600 IN A 192.168.1.1'
  view (client-net):
    name: client-net
    local-data: 'gateway.example.com. 3600 IN A 192.168.2.1'
```

License
-------

MIT
