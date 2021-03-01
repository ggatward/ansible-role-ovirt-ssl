# set up SSL for oVirt engine

This role sets up TLS certificates for `httpd`, `ovirt-imageio-proxy` and `ovirt-websockets-proxy` on ovirt-engine machine:

* `httpd`'s `mod_ssl` is configured always
* `ovirt-imageio-proxy` is configured only if it's configuration file is present
* `ovirt-websocket-proxy` is configured always (configuration file `99-ssl.conf` containing just key and certificate paths is dropped to configuration directory)

Limitation: the role doesn't deal with different configurations for different virtual hosts in `ssl.conf`.

Services are restarted when they are known on the system and enabled no matter if they are running or not.

## Requirements

PEM-encoded key and certificate files have to be present in the system. Certificate of httpd certificate issuer or any CA up the chain has to be either in the system trustore or specified as a contents of optional `httpd_ca_cert` variable.

## Role Variables

| Name | Required | Description, Default |
|------|----------|----------------------|
| `httpd_key_file` | Required | File with PEM-encoded private key with no encryption.<br>Default: UNDEF |
| `httpd_cert_file` | Required | File with PEM-encoded certificate. For httpd ≥ 2.4.8, it should also contain intermediate CA certificates<br>Default: UNDEF |
| `httpd_cert_chain_file` | Optional | certificate chain between httpd certificate and trusted CA. Required to be separate in httpd < 2.4.8 where `SSLCertificateChainFile` became obsolete. When omitted on system with httpd < 2.4.8, lines with `SSLCertificateChainFile` will get commented out<br>Default: UNDEF |
| `httpd_ca_cert` | Optional | PEM-encoded certificate of CA issuing `httpd` certificate or up the chain<br>Default: UNDEF |
| `httpd_ca_keystore_path` | Optional  | Key store to hold the certificate from `httpd_ca_cert` variable<br>Default: `/etc/pki/ovirt-engine/external_ca.jks` |
| `httpd_ca_keystore_password` | Optional | Password for key store<br>Default: `changeit` |
| `httpd_ssl_protocol` | Optional | Line specifying TLS protocols in `ssl.conf`. Default is unspecified for systems with `crypto-policies` package or TLS 1.2+ for systems without it<br>Default: UNDEF or `SSLProtocol all -SSLv2 -SSLv3 -TLSv1 -TLSv1.1` |


## Dependencies

None.

## Example Playbook

```yaml
---
- hosts: engines
  vars:
    httpd_key_file: /etc/pki/tls/private/localhost.key
    httpd_cert_file: /etc/pki/tls/certs/localhost.crt
    # When we don't want engine to trust all the default CAs, just the issuer
    # of the httpd certificate
    httpd_ca_cert: |
      -----BEGIN CERTIFICATE-----
      MII.....
      -----END CERTIFICATE-----
    tasks:
      - name: setup/update ovirt-engine
        ...

      - name: set up TLS for engine services
        import_role: djasa.ovirt-ansible-engine-ssl
```
