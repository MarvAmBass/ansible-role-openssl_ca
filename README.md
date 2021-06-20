# Ansible Role: openssl_ca

with this role you can use `openssl` to setup a CA and generate/sign multiple certificate with several options.

I recommend generating those on your (hopefully secured) localhost and transfering the needed files to the servers using a custom extra step.

```
- name: create ca with certificates locally
  hosts: 127.0.0.1
  connection: local
  roles:
   - openssl_ca
  vars:
    [...]
```

This way you can maintain multiple CAs for multiple purposes: `mail-transfer`, `rsyslog central log system` and even `openvpn` should work.

It has support for `subjectAltNames` and `extendedKeyUsage` customisations.

## Missing functionality

### Database and Revokation

I skipped implementing a CA database and revokation, since you can simply remove your CA and generate a entirely new one which replaces the current used CA and certifcates.

If you ever dealt with revocation lists and thier propagation you'll know that if you configured everything using ansible - recreating the whole CA is easier then the whole process of making sure revoked certificates are no longer trusted in your software.

If you use this for OpenVPN with several client devices, I'd recommend revokation and maybe `easyrsa` to manage your CA.

### Automate cert renew

As with the revoke, instead of renewing single certificates I recommend regenerating the whole CA with it's certificates.

## Configuration Variables

You can use several Variables to configure this role.

### Variables with predefined default value

- `path` - (default: `.`) path to store the CA folder.

- `ca_days` - (default: `3650`) days the CA will be valid
- `ca_bits` - (default: `4096`) bits for CA generation

- `cert_days` - (default: `1825`) days a Certificate will be valid
- `cert_bits` - (default: `4096`) bits for Certificate generation

### required variables

- `ca_name` - __required__ name of the CA and the name of the CA root folder
- `certs` - array of `cert` objects (see `cert` description below) for each certificate to generate/sign.

### other 
`subject` - _optional_ a `openssl -subj` string like `/C=US/ST=TX/L=Houston/O=My Test CA/OU=Infrastructe as Code Department`

__MUST NOT__
- contain `CN` (which is configured using the cert.cn option)
- end with a `/` (will be added automatically when combined with `CN`)

### `certs` array values

```
     certs:
      - cn: server.system.tld
        extendedKeyUsage: serverAuth
        sans:
        - server1.tld
        ips:
        - 127.0.10.1
        - 10.0.10.1
```

- `cn` - __required__ common name for certificate
- `keyUsage` - _optional_ (default: _not set_) can be any `keyUsage`-option or a comma seperated list
- `extendedKeyUsage` - _optional_ (default: `serverAuth,clientAuth`) can be any `extendedKeyUsage`-option or a comma seperated list
- `sans`- _optional_ array of alternative DNS names
- `ips`- _optional_ array of alternative IP-Addresses

## Example Playbook

```
---
- name: create ca with certificates locally
  hosts: 127.0.0.1
  connection: local
  roles:
   - openssl_ca
  vars:
   ca_name: "Test CA"
   subject: "/C=US/ST=TX/L=Houston/O=My Test CA/OU=Infrastructure as Code Department"
   certs:
      - cn: test
        sans:
        - test.tld
        - test.com
        - test.de
        extendedKeyUsage: serverAuth,clientAuth
      - cn: bob-client
        extendedKeyUsage: clientAuth
      - cn: tech
        sans:
        - tec1.tld
        - tec2.tld
        - tec3.tld
        ips:
        - 127.0.0.1
        - 10.0.0.1
      - cn: server.system.tld
        extendedKeyUsage: serverAuth
        sans:
        - server1.tld
        ips:
        - 127.0.10.1
        - 10.0.10.1
```