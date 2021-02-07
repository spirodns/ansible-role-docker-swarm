# ansible-role-docker-swarm
====================

This role uses Docker Engine's "Swarm Mode" (https://docs.docker.com/engine/swarm/) to create a cluster of Docker nodes.

Requirements
------------

An Ansible 2.8 or higher installation.

Dependencies
------------

None. 

Role Variables
--------------

Available variables are listed below, along with default values (see `[defaults/main.yml](defaults/main.yml)`):

    docker_repo: "{{ docker_repo_ce_stable }}"

The repository proving the Docker packages.
To skip the configuration of the repository and use the system repositories set `skip_repo: true`.

    docker_package_name: "docker-ce"

Name of the package providing the Docker daemon.

    docker_package_version: ""

Version of the Docker package to be installed on the target hosts.
When set to `""` the latest available version will be installed.

    docker_package_state: present

Set it to `latest` to force the upgrade of the installed Docker package to the latest version.

    docker_dependencies: "{{ default_docker_dependencies }}"

Additional support packages to be installed alongside Docker by the role.
See `[vars/RedHat.yml](vars/RedHat.yml)`  for the definition of the `default_docker_dependencies` variable.

    docker_service_override: ""
    # docker_service_override: |
    # [Service]
    # ExecStart=
    # ExecStart=/usr/bin/dockerd

Contect written to the systemd unit drop-in overriding
the default Docker service definition.

    docker_service_state: "started"
    docker_service_enabled: "yes"

State of the Docker service.

    docker_daemon_config: {}

Dictionary of Docker deamon configuration options to be written to `/etc/docker/daemon.json`.
See [Daemon configuration file](https://docs.docker.com/engine/reference/commandline/dockerd/#daemon-configuration-file)

    docker_cli_package_name: "docker-ce-cli"

Name of the package providing the Docker CLI.

    docker_cli_package_version: ""

Version of the Docker CLI package to be installed on the target hosts.
When set to `""` the latest available version will be installed.

    docker_cli_package_state: present

Set it to `latest` to force the upgrade of the installed Docker CLI package to the latest version.

    containerd_package_name: "containerd.io"

Name of the package providing containerd.

    containerd_package_version: ""

Version of the containerd package to be installed.
When set to `""` the latest available version will be installed.

    containerd_package_state: present

Set it to `latest` to force the upgrade of the installed containerd package to the latest version.

    containerd_service_override: |
      [Service]
      ExecStartPre=

Contect written to the systemd unit drop-in overriding
the default containerd service definition.

    containerd_service_state: "started"
    containerd_service_enabled: "yes"

State of the containerd service.

    docker_compose_version: ""

Version of docker-compose to be installed via python-pip.
When set to `""` the latest available version will be installed.

    docker_py_package_name: "docker"

Name of the python-pip package providing docker-py

    docker_py_package_version: ""

Version of the docker-py package to be installed.

    docker_py_package_state: present

Installation state of the docker-py package.
Set it to 'latest' to upgrade the Docker CLI to the latest version.

    docker_group_name: "docker"
    docker_group_users:
      - "{{ ansible_user }}"

Name of the Docker group and list of users to be added to `docker_group_name` to manage the Docker daemon.
**NB**: The users must already exist in the system.

    docker_swarm_interface: "{{ ansible_default_ipv4['interface'] }}"

Network interface to be used for cluster inter-communication.

    docker_swarm_addr: "{{ hostvars[inventory_hostname]['ansible_' + docker_swarm_interface]['ipv4']['address'] }}"

Listen address for the Swarm raft API.
By default, the ip address of `docker_swarm_interface`.

    docker_swarm_port: 2377

Listen port for the Swarm raft API.

    skip_repo: false
    skip_depends: false 
    skip_docker_core: false
    skip_containerd: false
    skip_engine: false
    skip_cli: false
    skip_swarm: false
    skip_group: false
    skip_docker_py: false
    skip_docker_compose: false
    skip_group: false     
    skip_docker_compose: false

Switches allowing to disable specific functions of the role.
If you want to use this role to install `docker-engine` without enabling `swarm-mode` set `skip_swarm: true`.


Swarm node labels
-----------------

[Node labels](https://docs.docker.com/engine/swarm/manage-nodes/#add-or-remove-label-metadata) provide a
flexible method of node organization. You can also use node labels in service constraints.
Nodes are automatically assigned labels according to the hostname and group added to the static inventory.
Groups to be used to separate nodes are `docker_swarm_manager` and `docker_swarm_worker`. For example:

    $ cat inventory
    ...
    [docker_swarm_manager]
    dev-swarm-manager0 

    [docker_swarm_worker]
    dev-swarm-worker0 
    ...
    
In this case:

    $ docker inspect --format '{{json .Spec.Labels}}'   dev-swarm-manager0 | jq
    {
       "worker": "dev-swarm-manager0",
    }

    $ docker inspect --format '{{json .Spec.Labels}}'   dev-swarm-worker0 | jq
    {
       "worker": "dev-swarm-worker0",
    }

Apply constraints when you create a service to limit the nodes where the scheduler assigns tasks for the service.
You can define labels by `swarm_labels` variable, e.g:

    $ cat inventory
    ...
    [docker_swarm_manager]
    dev-swarm-manager0 swarm_labels=deploy

    [docker_swarm_worker]
    dev-swarm-worker0 swarm_labels='["libvirt", "docker", "foo", "bar"]'
    ...

In this case:

    $ docker inspect --format '{{json .Spec.Labels}}'   dev-swarm-worker0 | jq
    {
       "worker": "dev-swarm-worker0",
       "bar": "true",
       "docker": "true",
       "foo": "true",
       "libvirt": "true",
    }

You can assign labels to cluster running playbook with `--tags=swarm_labels`

**NB**: Please note, all labels that are not defined in inventory will be removed

## Usage

```sh
ansible-galaxy install git@github.com:stackresilient/ansible-role-docker-swarm.git
```

To update the existing role version

```sh
ansible-galaxy install git@github.com:stackresilient/ansible-role-docker-swarm.git
```

For more information, please visit [installation page](https://docs.ansible.com/ansible/latest/reference_appendices/galaxy.html#installing-multiple-roles-from-a-file).

Example Playbook
----------------

    $ cat inventory
    [all]
    dev-swarm-manager0 ansible_ssh_host=10.85.1.36
    dev-swarm-manager1 ansible_ssh_host=10.85.1.104
    dev-swarm-manager2 ansible_ssh_host=10.85.1.186
    dev-swarm-worker0 ansible_ssh_host=10.85.1.84
    dev-swarm-worker1 ansible_ssh_host=10.85.1.161
    
    [docker_engine]
    dev-swarm-manager0 
    dev-swarm-manager1
    dev-swarm-manager2
    dev-swarm-worker0
    dev-swarm-worker1
    
    [docker_swarm_manager]
    dev-swarm-manager0 
    dev-swarm-manager1
    dev-swarm-manager2
    
    [docker_swarm_worker]
    dev-swarm-worker0
    dev-swarm-worker1

    $ cat playbook.yml
    ---
    - name: "Provision Docker Swarm Cluster"
      gather_facts: true
      hosts: all
      
      roles:
        - { role: ansible-role-docker-swarm }
