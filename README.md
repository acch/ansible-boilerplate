# Ansible-Boilerplate

[![GitHub Issues](https://img.shields.io/github/issues/acch/ansible-boilerplate.svg)](https://github.com/acch/ansible-boilerplate/issues) [![GitHub Stars](https://img.shields.io/github/stars/acch/ansible-boilerplate.svg?label=github%20%E2%98%85)](https://github.com/acch/ansible-boilerplate/) [![License](https://img.shields.io/github/license/acch/ansible-boilerplate.svg)](LICENSE)

[Ansible](https://www.ansible.com/) is a configuration management tool, similar to [Chef](https://www.chef.io/) and [Puppet](https://puppet.com/). It allows for performing logical configuration of infrastructure components, such as servers and network switches. The configuration files in this repository can act as a template for your own Ansible projects, in order to get you started quickly. Once you've customized the configuration files then new servers can be configured quickly &mdash; excluding their network configuration. This means that adding new servers is as simple as:

- Base OS installation of new server
- Network configuration of new server (including bond, bridge, DNS and routing)
- Configuration of password-less (public key) SSH authentication from the Ansible host (your laptop) to the new server

The remaining configuration (installing packages, configuring services, etc.) can then be achieved using Ansible. In addition, Ansible ensures that configuration of all servers is and remains consistent.

## Using this repository

Simply download (clone) the repository and start modifying files according to your needs.

```
git clone https://github.com/acch/ansible-boilerplate.git myAnsibleProject/
```

Ideally, you'll want to use [Git](https://git-scm.com/) to manage your Ansible configuration files. For that purpose simply [fork](https://help.github.com/articles/fork-a-repo/) this repository into your own Git repository before cloning and customizing it. Alternatively, create your own repository [from the template](https://help.github.com/en/github/creating-cloning-and-archiving-repositories/creating-a-repository-from-a-template). Git will allow you to version and roll-back changes with ease.

Specifically, you'll want to customize the following files:
- Add your own hosts and groups to file `hosts`. You'll want to replace `[anygroup]` with a more meaningful group name, and add your own groups as required.
- Define roles by adding subdirectories underneath directory `roles/`. You'll want to rename `anyrole/` to a more meaningful role name, and add your own roles as required.
- Associate your hosts (groups) with your roles by adding appropriate playbooks in the root directory. Rename `anygroup.yml` to a more meaningful playbook name.
- Import all your playbooks in the main `site.yml` playbook.

## Using Ansible

Install `ansible` on your laptop and link the `hosts` file from `/etc/ansible/hosts` to the file in your repository. Now you're all set.

To run a single (ad-hoc) task on multiple servers:

```
# Check connectivity
ansible all -m ping -u root

# Run single command on all servers
ansible all -m command -a "cat /etc/hosts" -u root

# Run single command only on servers in specific group
ansible anygroup -m command -a "cat /etc/hosts" -u root

# Run single command on individual server
ansible server1 -m command -a "cat /etc/hosts" -u root
```

As the `command` module is the default, it can also be omitted:

```
ansible server1 -a "cat /etc/hosts" -u root
```

To use shell variables on the remote server, use the `shell` module instead of `command`, and use single quotes for the argument:

```
ansible server1 -m shell -a 'echo $HOSTNAME' -u root
```

The true power of ansible comes with so called *playbooks* &mdash; think of them as scripts, but they're declarative. Playbooks allow for running multiple tasks on any number of servers, as defined in the configuration files (`*.yml`):

```
# Run all tasks on all servers
ansible-playbook site.yml -v

# Run all tasks only on group of servers
ansible-playbook anygroup.yml -v

# Run all tasks only on individual server
ansible-playbook site.yml -v -l server1
```

Note that `-v` produces verbose output. `-vv` and `-vvv` are also available for even more (debug) output.

To verify what tasks would do without changing the actual configuration, use the `--list-hosts` and `--check` parameters:

```
# Show hosts that would be affected by playbook
ansible-playbook site.yml --list-hosts

# Perform dry-run to see what tasks would do
ansible-playbook site.yml -v --check
```

Running all tasks in a playbook may take a long time. *Tags* are available to organize tasks so one can only run specific tasks to configure a certain component:

```
# Show list of available tags
ansible-playbook site.yml --list-tags

# Only run tasks required to configure DNS
ansible-playbook site.yml -v -t dns
```

Note that the above command requires you to have tasks defined with the `tags: dns` attribute.

## Configuration files

The `hosts` file defines all hosts and groups which they belong to. Note that a single host can be member of multiple groups. Define groups for each rack, for each network, or for each environment (e.g. production vs. test).

### Playbooks

Playbooks associate hosts (groups) with roles. Define a separate playbook for each of your groups, and then import all playbooks in the main `site.yml` playbook.

File | Description
---- | -----------
`site.yml` | Main playbook - runs all tasks on all servers
`anygroup.yml` | Group playbook - runs all tasks on servers in group *anygroup*

### Roles

The group playbooks (e.g. `anygroup.yml`) simply associate hosts with roles. Actual tasks are defined in these roles:

```
roles/
├── common/             Applied to all servers
│   ├── handlers/
│   ├── tasks/
│   │   └ main.yml      Tasks for all servers
│   └── templates/
└── anyrole/            Applied to servers in specific group(s)
    ├── handlers/
    ├── tasks/
    │   └ main.yml      Tasks for specific group(s)
    └── templates/
```

Consider adding separate roles for different applications (e.g. webservers, dbservers, hypervisors, etc.), or for different responsibilities which servers fulfill (e.g. infra_server vs. infra_client).

### Tags

Use the following command to show a list of available tags:

```
ansible-playbook site.yml --list-tags
```

Consider adding tags for individual components (e.g. DNS, NTP, HTTP, etc.).

Role | Tags
--- | ---
Common | all,check

## Copyright and license

Copyright 2017 Achim Christ, released under the [MIT license](LICENSE)
