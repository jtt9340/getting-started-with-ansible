# Getting Started with Ansible

This repository is a scratch pad for teaching myself [Ansible][ansible] following [this YouTube tutorial][youtube].
That link will bring you to the first in a series of videos by [Learn Linux TV][learn-linux-tv] for teaching Ansible.

The remainder of this README is really just notes to my (future) self on anything I think is important to know/remember for Ansible.

## Ansible Basics

In Ansible, **servers** are provisioned by a **control host** (this diagram is adapted from one shown in the first video
in the linked Ansible tutorial series):

```
                                        /----------------------\
                               /------> |         srv1         |
                               |        \----------------------/
                               |
/----------------------\       |        /----------------------\
| Ansible Control Host | ------+------> |         srv2         |
\----------------------/       |        \----------------------/
                               |
                               |        /----------------------\
                               \------> |         srv3         | 
                                        \----------------------/
```

For this, my laptop is the **control host** and three Ubuntu VMs, created and managed by Vagrant and VirtualBox,
are the three servers that need to be configured.

The [`Vagrantfile`](./Vagrantfile) in the root of this repository describes this setup and everything should be
able to be brought up with

```bash
vagrant up
```

provided Vagrant and VirtualBox are installed.

## Installing Ansible

Most package managers, including Homebrew, provide Ansible, which can be installed that way. But, to keep things portable,
I've provided a simple [`requirements.txt`](./requirements.txt) which can be used to install Ansible in a Python virtual environment
(or globally if you like).

## [`inventory`](./inventory)

According to the Ansible documentation, ["An inventory is a list of managed nodes, or hosts, that Ansible deploys and configures."][inventory-docs],
so this file lists the IP addresses of the three VMs for this repository.

The hosts are divided into groups whose names are wrapped in [ ].
A host can belong to multiple different groups, but for this example each host only belongs to a single group.
Groups allow Ansible to target a subset of hosts in the inventory, rather than all of them or only one of them.

Note that there is an `ansible-inventory` binary for interacting with the inventory.

## [`ansible.cfg`](./ansible.cfg)

Ansible can be configured, and one of the first places it looks for a configuration file is in the directory it is
being invoked from. `ansible.cfg` is an INI file and [the Ansible Documentation][config-docs] lists all the options
that can be specified.

Note that there is an `ansible-config` binary for interacting with the configuration.

## Ad-Hoc Commands

These are just commands you invoke from the shell that instruct Ansible to do something.

### Ping

Used to determine if the servers are reachable and can be logged in via SSH.

```bash
ansible all -m ping
```

Replace `all` with the IP address or host name of an individual server to run the Ping module on only that server.
Alternatively, `all` can be replaced with a group name (specified in the inventory) and the module will be run on
only the hosts for that particular group.

### `--list-hosts`

Used to see which hosts Ansible will be provisioning. _Question: What is the difference between this and using `ansible-inventory`
to determine this information?_

```bash
ansible all --list-hosts
```

### Gather Facts

Get a whole bunch of info about a server.

```bash
ansible all -m gather_facts
```

You can pass the `--limit` flag to specify only a certain subset of hosts to gather facts about.

```bash
ansible all -m gather_facts --limit 192.168.56.10
```

_Commentary: Not sure what the difference is between `ansible 192.168.56.10 -m gather_facts` and
`ansible all -m gather_facts --limit 192.168.56.10`._

## Elevated Ad-Hoc Commands

Commands that require root (elevated) privileges, such as installing packages. To do this, pass the `--become`
flag to Ansible. If a password is required to become root, also pass the `--ask-become-pass` to have Ansible
prompt for the root password.

### Apt

The Apt module for Ansible is used to interact with Apt on hosts. This module requires arguments, which
are passed to Ansible with the `-a` flag and are given in `key=value` pairs. All the recognized arguments
for this module are documented in [the Ansible documentation][apt-module-docs].

To install a package (e.g. `vim-nox`):

```bash
ansible all -m apt -a name=vim-nox --become --ask-become-pass
```

To ensure a given package is up-to-date:

```bash
# Notice, since we are passing multiple key=value pairs to -a, they must be surrounded in quotes.
ansible all -m apt -a 'name=vim-nox state=latest' --become --ask-become-pass
```

To upgrade every upgradeable package:

```bash
ansible all -m apt -a upgrade=dist --become --ask-become-pass
```

## Playbooks

Playbooks allow you to "script" usage of Ansible. They are YAML files that are "executed" by the
`ansible-playbook` binary.

The examples [`site.yml`](./site.yml) and [`remove_apache.yml`](./remove_apache.yml)
show how to write two playbooks. `site.yml` is commented with some important things to note when
writing playbooks.

To run the `site.yml` playbook, do

```bash
ansible-playbook site.yml --ask-become-pass
```

Replace `site.yml` with `remove_apache.yml` in the above command to run the `remove_apache.yml` playbook.

The [Ansible documentation][playbook-docs] mentions two more binaries: `ansible-pull` and `ansible-lint`.
It seems `ansible-pull` was installed by default upon running `pip install ansible` but `ansible-lint` needed
to be installed separately with `pip install ansible-lint` (see [`requirements.txt`](./requirements.txt)).

Normally, with Ansible, the **control host** _pushes_ configuration to one or more hosts, but `ansible-pull`
inverts this control to have a host instead git clone (or _pull_) a playbook and then run it with `ansible-pull`. I've
seen this technique used in people's dotfiles repositories to automate the setup of a new computer.

`ansible-lint` is a linter for Ansible playbooks.

[ansible]: https://www.ansible.com
[youtube]: https://youtu.be/3RiVKs8GHYQ
[learn-linux-tv]: https://www.learnlinux.tv/
[inventory-docs]: https://docs.ansible.com/ansible/latest/inventory_guide/index.html
[config-docs]: https://docs.ansible.com/ansible/latest/reference_appendices/config.html
[apt-module-docs]: https://docs.ansible.com/ansible/latest/collections/ansible/builtin/apt_module.html
[playbook-docs]: https://docs.ansible.com/ansible/latest/playbook_guide/playbooks_intro.html
