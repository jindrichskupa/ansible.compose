Docker Compose Ansible Role
=========

Creates `docker-compose.yml`, (config) files and directories. Creates systemd units for composes.

Requirements
------------

* debian based distribution
* docker installed, you can use [this](https://galaxy.ansible.com/jindrichskupa/ansible_docker) role

Example Playbook
----------------

```yaml
---
- name: Manage Docker Application Server
  hosts: docker.example.com
  remote_user: ansible
  become: true
  vars:
    ansible_python_interpreter: /usr/bin/python3
  roles:
    - role: jindrichskupa.ansible_docker
    - role: jindrichskupa.ansible_compose
      vars:
        # project/directory/unit name
        docker_app_name: promtail
        # docker compose base, default: `/srv/docker`
        docker_compose_base: /srv/docker
        # docker compose version, default: 1.29.2
        docker_compose_version: 1.29.2
        # template of `docker-compose.yml`
        docker_compose_template: compose/promtail/docker-compose.yml
        # create systemd unit, default: false
        docker_systemd: true
        # custom/generated files, like config files
        docker_compose_files:
          - template: compose/promtail/config.yaml
            destination: promtail/config.yaml
        # will create empty dirs in project directory
        docker_compose_dirs:
          - promtail
        # custom variables used in templates place here:
        docker_image_backend: grafana/promtail:master
    # real life example
    - role: jindrichskupa.ansible_compose
      vars:
        docker_app_name: app1
        docker_compose_template: compose/app1/docker-compose.yml
        docker_compose_files:
          - template: compose/app1//Dockerfile
            destination: app1/Dockerfile
          - template: compose/app1/entrypoint.sh
            destination: app1/entrypoint.sh
        docker_compose_dirs:
          - data
          - config
        docker_compose_database_user: app1_user
        docker_compose_database_pass: !vault |
                  $ANSIBLE_VAULT;1.1;AES256
                  35626337626164313637383463396533366465626466393639316565393231393637
        docker_compose_database_host: postgres
```

Template Example
----------------

```yaml
version: "3"
services:
  promtail:
    image: {{ docker_image_backend }}
    volumes:
      - $PWD/promtail:/etc/promtail
      - /var/log:/var/log
      - /var/lib/docker/containers:/var/lib/docker/containers
    command: "-config.file=/etc/promtail/config.yaml"
{% raw %}
    logging:
      options:
        tag: "{{.ImageName}}|{{.Name}}|{{.ImageFullID}}|{{.FullID}}"
{% endraw %}
    restart: always
```

License
-------

MIT

Author Information
------------------

Jindrich Skupa
