# Ansible Role: ca-certificates

## Description

Ansible role originated from arillso.ca_certificates to manage CA certificates in the Linux and Windows system trust store. It's possible to add PEM formatted certificates from the local file system, a already trusted HTTP(s) URL, from raw content.

## Installation

```bash
git clone https://github.com/alexkross/ansible.ca-certificates.git alexkross.ca-certificates
```

## Requirements

none

## Role Variables

### ca_certificates_root_directory

Location where the certificates are stored under windows before
they are imported into the certificate store of Windows.

```yml
ca_certificates_root_directory: '{{ ansible_env.TMP }}'
```

### ca_certificates_packages

Packages to be installed.

```yml
ca_certificates_packages:
  - ca-certificates
```

### ca_certificates_files

List of CA certificates that are to be added to the certificate store of the system. Each list element is a configuration directory that defines the source (URL, Files or Inline as variable) of the certificate. It must contain a key'name' and one of the following keys in order to use the certificate:

| Option         | Comments                                                                                                                                                                                      |
| :------------- | :-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| file           | Path to a file on the host running the Ansible playbook. Relative file paths are related to the role's `files/` directory.                                                                    |
| url            | URL to a PEM-formatted certificate file                                                                                                                                                       |
| content        | Certificate inline as PEM-formatted                                                                                                                                                           |
| store_name     | Optional in Windows. The store name to use when importing. See: [Ansible doc](https://docs.ansible.com/ansible/latest/modules/win_certificate_store_module.html#win-certificate-store-module) |
| store_location | Optional in Windows. See: [Ansible doc](https://docs.ansible.com/ansible/latest/modules/win_certificate_store_module.html#win-certificate-store-module)                                       |

```yml
ca_certificates_files: []
```

## Dependencies

None

## Example Playbook

```yml
---
- hosts: archlinux:centos:!luna-1v
  gather_facts: yes
  tasks:
  - name: Convert *.crt (DER) to *.pem ignoring errors
    local_action: command openssl x509 -in {{ item }} -inform der -out {{ basename }}.pem -outform pem
    vars:
      basename: '{{ item | splitext | first }}'
    with_fileglob: files/*.cer
    run_once: true
    ignore_errors: true
  roles:
  - role: alexkross.ca-certificates
    vars:
      ca_name_mark: _root_ # should include serial
      files: '{{ lookup("fileglob", "*.pem").split(",") | map("basename") | list }}'
      names: "{{ files | map('regex_replace', '^(.*)'~ca_name_mark~'.+$', '\\1') | list }}"
      ca_certificates_files: '{{ dict(names | zip(files)) | dict2items(key_name="name", value_name="file") }}'
```

## Author

- [Simon Bärlocher](https://sbaerlocher.ch)
- [Alexander Kross](mailto:Alexander.Kross@gmail.com)

## License

This project is under the MIT License. See the [LICENSE](https://sbaerlo.ch/licence) file for the full license text.

## Copyright

(c) 2020, Arillso
(c) 2021, AlexKross
