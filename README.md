# Ansible Role: ca_trust

![GitHub](https://img.shields.io/github/license/jomrr/ansible-role-ca_trust)
![GitHub last commit](https://img.shields.io/github/last-commit/jomrr/ansible-role-ca_trust)
![GitHub issues](https://img.shields.io/github/issues-raw/jomrr/ansible-role-ca_trust)
[![dev](https://img.shields.io/github/actions/workflow/status/jomrr/ansible-role-ca_trust/dev.yml?branch=dev&event=push&label=dev)](https://github.com/jomrr/ansible-role-ca_trust/actions/workflows/dev.yml?query=branch%3Adev)
[![main](https://img.shields.io/github/actions/workflow/status/jomrr/ansible-role-ca_trust/main.yml?branch=main&event=push&label=main)](https://github.com/jomrr/ansible-role-ca_trust/actions/workflows/main.yml?query=branch%3Amain)

Ansible role for installing CA certificates in the operating system trust store.

## Purpose

Add the specified CA certificates to the operating system trust store on
AlmaLinux, Debian, Fedora, openSUSE Leap, openSUSE Tumbleweed and Ubuntu.
The role installs the trust-store tools, validates each candidate certificate
with OpenSSL, and refreshes the trust store when a certificate changes.

## Scope

### Managed

- Installation of ca-certificates and openssl.
- Named CA certificates in the platform's local trust-anchor directory.
- Refresh of the system trust store before the role returns.

### Not Managed

- Certificate issuance, private keys, renewal or revocation.
- Removal of certificates omitted from ca_trust_certificates or renamed in the
  input.
- Application-specific trust stores and application restarts.

## Dependencies

```yaml
collections:
  - name: community.general
    version: '>=12.0.0'
```

## Role Variables

### `ca_trust_certificates`

Type: `list`. Required: `false`.

CA certificates to add or update; unlisted certificates are preserved.

Default:

```yaml
ca_trust_certificates: []
```

## Managed Files

- `Debian and Ubuntu: /usr/local/share/ca-certificates/<name>.crt`
- `AlmaLinux and Fedora: /etc/pki/ca-trust/source/anchors/<name>.crt`
- `openSUSE: /etc/pki/trust/anchors/<name>.crt`

## Check Mode

Reports package and certificate changes without writing certificates or
refreshing the trust store.

- Candidate certificate validation runs when a changed certificate is actually
  installed, not during check mode.

## Service Behavior

Changed certificates trigger one platform-specific trust-store refresh; no
service is managed.

### Handlers

- Run update-ca-certificates on Debian, Ubuntu and openSUSE, or update-ca-trust
  extract on AlmaLinux and Fedora.

## Security Notes

- Certificates are installed as root-owned files with mode 0644; no private keys
  are needed.
- Only supply CA certificates whose issuers should be trusted by system
  applications.

## Operational Notes

- Each entry requires name and content. Names must be unique, start with an
  ASCII letter or digit, and contain only ASCII letters, digits, dots,
  underscores or hyphens; the role appends .crt to form the filename.
- Each content value must contain exactly one PEM-encoded CA certificate, not a
  certificate chain or DER data.
- Reusing a name replaces that certificate; an empty list preserves all existing
  trust anchors.
- The OpenSSL validator checks certificate encoding; selecting trusted CA
  issuers remains an inventory decision.
- Applications that cache trusted certificates may require a separate restart
  after the role completes.

## Supported Platforms

| OS Family | Distribution | Version | Container Image |
| --------- | ------------ | ------- | --------------- |
| RedHat | AlmaLinux | latest | [jomrr/molecule-almalinux:latest](https://hub.docker.com/r/jomrr/molecule-almalinux) |
| Debian | Debian | latest | [jomrr/molecule-debian:latest](https://hub.docker.com/r/jomrr/molecule-debian) |
| RedHat | Fedora | latest | [jomrr/molecule-fedora:latest](https://hub.docker.com/r/jomrr/molecule-fedora) |
| Suse | OpenSuse Leap | latest | [jomrr/molecule-opensuse-leap:latest](https://hub.docker.com/r/jomrr/molecule-opensuse-leap) |
| Suse | OpenSuse Tumbleweed | latest | [jomrr/molecule-opensuse-tumbleweed:latest](https://hub.docker.com/r/jomrr/molecule-opensuse-tumbleweed) |
| Debian | Ubuntu | latest | [jomrr/molecule-ubuntu:latest](https://hub.docker.com/r/jomrr/molecule-ubuntu) |

## Example Playbook

### Trust CA certificates from controller files

Load public CA certificates from files available to the Ansible controller.

```yaml
- name: Configure system CA trust
  hosts: all
  gather_facts: true
  roles:
    - role: jomrr.ca_trust
      ca_trust_certificates:
        - name: organization-root
          content: >-
            {{ lookup('ansible.builtin.file', 'certificates/root.pem') }}
        - name: partner-root
          content: >-
            {{ lookup('ansible.builtin.file', 'certificates/partner.pem') }}

```

## References

- [Debian trust store](https://manpages.debian.org/unstable/ca-certificates/update-ca-certificates.8.en.html)
- [Red Hat trust store](https://docs.redhat.com/en/documentation/red_hat_enterprise_linux/10/html/securing_networks/using-shared-system-certificates)
- [SUSE trust store](https://documentation.suse.com/smart/security/html/tls-certificates/index.html)

## Author

[Jonas Mauer](https://github.com/jomrr)

## License

This project is licensed under the MIT License.
See [LICENSE](LICENSE) for the full license text.

Copyright (c) 2026 Jonas Mauer.
