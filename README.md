# Getting Started with Ansible

This repository is a scratch pad for teaching myself [Ansible][ansible] following [this YouTube tutorial][youtube].
That link will bring you to the first in a series of videos by [Learn Linux TV][learn-linux-tv] for teaching Ansible.

The remainder of this README is really just notes to my (future) self on anything I think is important to know/remember for Ansible.

## Ansible Basics

In Ansible, servers are provisioned by a **control host** (this diagram is adapted from one shown in the first video
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

According to the Ansible documentation, ["An **inventory** is a list of managed nodes, or hosts, that Ansible deploys and configures."][inventory-docs],
so this file lists the IP addresses of the three VMs for this repository.

The hosts are divided into **groups** whose names are wrapped in [ ].
A host can belong to multiple different **groups**, but for this example each host only belongs to a single **group**.
**Groups** allow Ansible to target a subset of hosts in the **inventory**, rather than all of them or only one of them.

Note that there is an `ansible-inventory` executable for interacting with the **inventory**.

## [`ansible.cfg`](./ansible.cfg)

Ansible can be configured, and one of the first places it looks for a configuration file is in the directory it is
being invoked from. `ansible.cfg` is an INI file and [the Ansible Documentation][config-docs] lists all the options
that can be specified.

Note that there is an `ansible-config` executable for interacting with the configuration.

## Ad-Hoc Commands

These are just commands you invoke from the shell that instruct Ansible to do something.

### Ping

Used to determine if the servers are reachable and can be logged in via SSH.

```bash
ansible all -m ping
```

Replace `all` with the IP address or host name of an individual server to run the Ping module on only that server.
Alternatively, `all` can be replaced with a group name (specified in the **inventory**) and the module will be run on
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

The Apt module is used to interact with Apt on hosts. This module requires arguments, which
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

**Playbooks** allow you to "script" usage of Ansible. They are YAML files that are "executed" by the
`ansible-playbook` executable. They list a series of **tasks** to be run in a certain order, where each
task is one of the modules that has been passed to `-m` in the ad-hoc commands above.

The examples [`site_before_roles.yml`](./site_before_roles.yml) and [`bootstrap.yml`](./bootstrap.yml)
show how to write two **playbooks**. `site_before_roles.yml` is commented with some important things to note when
writing **playbooks**. [`site.yml`](./site.yml) is `site_before_roles.yml` rewritten to use Ansible **roles**, discussed below.

To run the `site_before_roles.yml` playbook, do

```bash
ansible-playbook site_before_roles.yml --ask-become-pass
```

Replace `site_before_roles.yml` with `bootstrap.yml` in the command above to run the `bootstrap.yml` **playbook**
instead. `bootstrap.yml` is intended to "bootstrap" a host that has never been provisioned by
Ansible before by configuring a user and copying its SSH keys so that the `--ask-become-pass` flag
doesn't need to be passed to future Ansible commands.

Pass the `--check` and `--diff` flags (`-C` and `-D` for short, respectively) to `ansible-playbook` to do a "dry run"
of the playbook before actually running it, to ensure Ansible is going to do what you expect it to do.

### Tags

You'll notice `tags` entries in `site_before_roles.yml`. These can be used to selectively run certain **tasks** in
a **playbook**. By default, Ansible runs all **tasks** (that match the appropriate host and conditions and
all that), but if you pass a comma-separated list to `--tags` then only the specified **tags** are run,
plus any **tasks** tagged `always` (regardless of whether or not you specified `always`).

For example, running

```bash
ansible-playbook --tags ubuntu,samba site_before_roles.yml
```

will run all **tasks** in `site_before_roles.yml` with the `always`, `ubuntu`, and/or `samba` **tags**.

To see which **tags** are defined in a **playbook**, run

```bash
ansible-playbook --list-tags site_before_roles.yml
```

See the [Ansible documentation][tags-docs] for more information on where **tags** can appear in **playbooks**
as well as additional ways for specifying **tags** at the command line.

### `ansible-pull` and `ansible-lint`

The [Ansible documentation][playbook-docs] mentions two more executables: `ansible-pull` and `ansible-lint`.
It seems `ansible-pull` was installed by default upon running `pip install ansible` but `ansible-lint` needed
to be installed separately with `pip install ansible-lint` (see [`requirements.txt`](./requirements.txt)).

Normally, with Ansible, the **control host** _pushes_ configuration to one or more hosts, but `ansible-pull`
inverts this control to have a host instead git clone (or _pull_) a **playbook** and then run it with `ansible-playbook`. I've
seen this technique used in people's dotfiles repositories to automate the setup of a new computer.

`ansible-lint` is a linter for Ansible **playbooks**.

## Roles

The concept of **roles** enables separation of concerns within **playbooks**, so that all the **plays** and **tasks** within a **playbook**
(or **taskbook** as they will come to be referred as when talking about **roles**) are related to a single goal.

[`site.yml`](./site.yml) is [`site_before_roles.yml`](./site_before_roles.yml) rewritten to use **roles**. Now each **play** has
a `roles` entry with a list of **role** names. Ansible expects the directory structure to be the following:

```
.
├── site.yml
└── roles
    ├── base
    │   └── tasks
    │       └── main.yml
    ├── db
    │   └── tasks
    │       └── main.yml
    ├── files
    │   └── tasks
    │       └── main.yml
    ├── webservers
    │   ├── files
    │   │   └── default_site.html
    │   └── tasks
    │       └── main.yml
    └── workstations
        └── tasks
            └── main.yml
```

where each directory under the `roles` directory is the name of a **role** specified in the top-level playbook, `site.yml`.
Ansible then expects a `main.yml` in a `tasks` directory under each **role** directory. Each YAML file in a tasks directory
is a **taskbook**. The difference between a **taskbook** and a **playbook** is a **taskbook** only contains the part that
would appear under a `tasks` or `pre_tasks` entry in a **playbook**.

You'll notice the `webservers` **role** also contains a `files` directory. If a role uses the copy module it must store the
sources of the file(s) to be copied in such a directory. There are also other directories that can be part of a role, and are documented
in the [Ansible documentation][roles-docs].

`site.yml` is executed just as it was before: `ansible-playbook site.yml`.

[ansible]: https://www.ansible.com
[youtube]: https://youtu.be/3RiVKs8GHYQ
[learn-linux-tv]: https://www.learnlinux.tv/
[inventory-docs]: https://docs.ansible.com/ansible/latest/inventory_guide/index.html
[config-docs]: https://docs.ansible.com/ansible/latest/reference_appendices/config.html
[apt-module-docs]: https://docs.ansible.com/ansible/latest/collections/ansible/builtin/apt_module.html
[playbook-docs]: https://docs.ansible.com/ansible/latest/playbook_guide/playbooks_intro.html
[tags-docs]: https://docs.ansible.com/ansible/latest/playbook_guide/playbooks_tags.html
[roles-docs]: https://docs.ansible.com/ansible/latest/playbook_guide/playbooks_reuse_roles.html
