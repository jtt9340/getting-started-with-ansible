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

## [`inventiory.ini`](./inventory.ini)

According to the Ansible documentation, ["An inventory is a list of managed nodes, or hosts, that Ansible deploys and configures."][inventory-docs],
so this file lists the IP addresses of the three VMs for this repository.

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

[ansible]: https://www.ansible.com
[youtube]: https://youtu.be/3RiVKs8GHYQ
[learn-linux-tv]: https://www.learnlinux.tv/
[inventory-docs]: https://docs.ansible.com/ansible/latest/inventory_guide/index.html
[config-docs]: https://docs.ansible.com/ansible/latest/reference_appendices/config.html