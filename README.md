# Ansible Role: ansible_bender

[![Build](https://github.com/coglinev3/ansible-role-ansible_bender/actions/workflows/build.yml/badge.svg)](https://github.com/coglinev3/ansible-role-ansible_bender/actions/workflows/build.yml) ![GitHub tag (latest by date)](https://img.shields.io/github/v/tag/coglinev3/ansible-role-ansible_bender) [![License](https://img.shields.io/badge/License-BSD%203--Clause-blue.svg)](https://raw.githubusercontent.com/coglinev3/ansible-role-ansible_bender/master/LICENSE)

Are you tired of building containers with Dockerfiles?

This role installs
[ansible-bender](https://github.com/ansible-community/ansible-bender), a tool
which bends containers using Ansible playbooks and turns them into container
images.

With ansible-bender, you no longer have to build and configure containers
differently than you do traditional virtual machines or bare-metal systems.
You can now apply the power of Ansible and re-use your existing Ansible
content for your containerized ecosystem. Use templates, copy files, drop in
encrypted data, handle errors, add conditionals, and more. Everything Ansible
brings to orchestrating your infrastructure can now be applied to the image
build process.

The supported Linux distributions for this role are:
* Alpine Linux 3.16,
* Alpine Linux 3.17,
* Alpine Linux 3.18,
* Alpine Linux 3.19,
* Alpine Linux 3.20,
* Alpine Linux 3.21,
* Alpine Linux 3.22,
* Enterprise Linux 9, 
* Enterprise Linux 10, 
* Debian 11 (Bullseye),
* Debian 12 (Bookworm),
* Debian 13 (Trixie),
* Linux Mint 20 (Ulyana),
* Ubuntu 22.04 LTS (Jammy Jellyfish),
* Ubuntu 24.04 LTS (Noble Numbat).


## Requirements

Ansible-bender requires a few binaries to be present on your host system:

* Python 3.9 or later
* Ansible 
* Buildah
* Podman

All requirements are installed with this role.

## Role Variables

Available variables are listed below, along with default values (see defaults/main.yml):

```yml
# use admin rights with sudo or not
ab_become: true

# state for installed packages: absent | present | latest
ab_package_state: latest

# ansible-bender needs python3.6 or higher
ab_dependencies:
  - python3-pip
  - python3-virtualenv
  - buildah
  - podman

ab_dependencies_package_state: present

ab_python_dependencies:
  - ansible

ab_python_package_state: present

# Ansible-bender version. If empty, defaults to latest.
ab_version: ''

# comma separated list of registries
ab_container_search_registry: >-
  'docker.io',
  'registry.fedoraproject.org',
  'quay.io',
  'registry.access.redhat.com',
  'registry.centos.org'

ab_repo_prefix: ""

# a list user which can use the rootless mode:
ab_users: []
```

## Dependencies

None.

## Example Playbook

```yml
---
# file: roles/ansible-bender/tests/test.yml

- hosts: all
  vars:
    ab_users:
      - your_username
  roles:
    - { role: coglinev3.ansible_bender }
```

## Example how to use ansible-bender

### Create an image

If you want to test ansible-bender, you first need an Ansible playbook. You can create a playbook template with:

```sh
ansible-bender init
```

Now open the `playbook.yml` file, change the *ansible_bender* dictionary variable and add some tasks. The following simple example playbook creates a nginx container based on Alpine Linux.

```yml
---
- name: Containerized version of nginx
  hosts: all
  vars:
    # configuration specific for ansible-bender
    ansible_bender:
      # ansible-bender needs an image with preinstalled Python 3 to work
      base_image: python:3-alpine
      target_image:
        # command to run by default when invoking the container
        cmd: "nginx -g \"daemon off;\""
        name: bender-nginx
        ports: ['80', '443']
        working_dir: /var/www/localhost/htdocs
        labels:
          build-by: "{{ ansible_user }}"
      working_container:
        volumes:
        # mount this git repo to the working container at /src
        - "{{ playbook_dir }}:/src:Z"
  tasks:
  - name: install dependencies needed to run project bender-nginx
    apk:
      name: nginx
      state: present
  - name: Ensure directory /run/nginx exists
    file:
      path: /run/nginx
      state: directory
      mode: '0750'
      owner: nginx
      group: nginx
```

And now you can build the example image with:

```sh
ansible-bender build ./playbook.yml
```

### Run the container

After the image is successfully created you can start a new container with
podman.

```sh
podman run -d -p 8080:80 bender-nginx
```

Finally, you can use *curl* to test if the nginx container is working properly.

```sh
curl http://127.0.0.1:8080/ 
```

If you get an answer with "404 Not Found", nginx will work fine. Nginx informs you that the requested page was not found.

## Version

Release: 1.6.0

## License

BSD

## Author Information

Copyright &copy; 2019 - 2025 Cogline.v3.
