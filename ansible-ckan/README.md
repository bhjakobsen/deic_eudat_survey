## CKAN Ansible Playbook
[![Build Status](https://travis-ci.org/oguya/ansible-ckan.svg)](https://travis-ci.org/oguya/ansible-ckan)

This is an ansible playbook for building and minimally deploying [CKAN](http://ckan.org/) on Ubuntu 14.04 host. This playbook tries as much as possible to follow [the recommended way of installing CKAN from source](http://docs.ckan.org/en/latest/maintaining/installing/install-from-source.html) while adhering to [Ansible best practices](http://docs.ansible.com/playbooks_best_practices.html).
After successfully running this playbook, there are some additional tasks which must be run manually such as initializing databases, copying site theme, content e.t.c.

By default, this playbook assumes that you're installing CKAN & its dependencies(solr, postgresql, nginx & apache) on one host. And you'll be using apache web server primarily for python WSGI support via nginx web server on the front(reverse proxy) i.e. port 80 as recommended by CKAN docs.

### Ansible Roles
This playbook is broken down the following several parts/roles, each doing a minimal & specific tasks:
- `apache` role: Installs & configures apache2. Apache provides [modwsgi](https://code.google.com/p/modwsgi/) which allows us to run python applications with WSGI interface. This role also configures apache2 to listen & serve requests on port `8080` since we'll have nginx on the front( port `80`).
- `nginx` role: Installs & configures nginx web server. Additional configuration parameters for proxying, X-* security headers, gzip compression, caching, e.t.c. are also deployed on the remote server.
- `postgresql` role: This role installs & configures postgresql-9.3 server. This role doesn't create/manage postgresql users & or databases, that is done in the ckan role.
- `solr` role: Installs & configures `solr-5.2.x`. By default, this role creates one solr core—`ckan_default`—used by CKAN. Though CKAN doesn't insist/recommend using a specific version of solr, I decided to use `solr 5.2.1` and it works fine on test environments.
- `ckan` role: This role builds & installs CKAN from sources using pip. It creates a ckan user who owns all CKAN related files & directories, creates postgresql user & database used by CKAN, configures apache2 vhost for WSGI support & finally creates nginx vhost for proxy passing requests to apache2 WSGI processes.

> **NOTE**<br/>
> `apache`, `nginx` & `ckan` roles can all be grouped in one group—_ckan_—and are usually deployed on one host in production & test environments whereas `postgresql` & `solr` roles are in separate groups—_db_ & _solr_ respectively—and are usually deployed on separate hosts too.

### Usage

- clone this repo:

        $ git clone https://github.com/oguya/ansible-ckan.git
        $ cd ansible-ckan

- Add your `db`, `solr` and `ckan` host(s) in the [inventory](http://docs.ansible.com/intro_inventory.html) `hosts` file, for example: `ckan01` host belongs to the `ckan`, `db` and `solr` groups - for more info about these groups, see sample host variables files in [host_vars](https://github.com/oguya/ansible-ckan/tree/master/host_vars) directory:

        [ckan]
        ckan01

        [db]
        ckan01

        [solr]
        ckan01

- dry run all the playbooks:

        $ ansible-playbook site.yml --ask-become-pass --check

- deploy CKAN & its dependencies(solr, postgresql, nginx & apache) on `ckan01` host:

        $ ansible-playbook site.yml --ask-become-pass -i hosts --limit=ckan01

- install & configure solr 5.2.x on `ckan01` host:

        $ ansible-playbook solr.yml --ask-become-pass -i hosts --limit=ckan01

- install & configure postgresql-9.3 on `ckan01` host:

        $ ansible-playbook db.yml --ask-become-pass -i hosts --limit=ckan01

- deploy CKAN on `ckan01` host:

        $ ansible-playbook ckan.yml --ask-become-pass -i hosts --limit=ckan01

### Assumptions
I took into assumption, a few key items when running this playbook:
- You have a provisioning user account—`provisioning`—with passwordless SSH access to the target host
- The provisioning user has sudo privileges on the remote host
- You're installing CKAN & its dependencies(solr, postgresql, nginx & apache) on one target host - [ckan01](https://github.com/oguya/ansible-ckan/blob/master/host_vars/ckan01)

### References
A few roles in this playbook e.g. postgresql, solr & nginx have been borrowed from [ILRI's rmg-ansible-public repo](https://github.com/ilri/rmg-ansible-public).

### Contribution
[Pull requests](https://github.com/oguya/ansible-ckan/issues) and [Github issues](https://github.com/oguya/ansible-ckan/issues) are all welcome!

## License
Copyright (C) 2015–2016 James Oguya

The contents of this repository are free software: you can redistribute
it and/or modify it under the terms of the GNU General Public License
as published by the Free Software Foundation, either version 3 of the
License, or (at your option) any later version.

This program is distributed in the hope that it will be useful,
but WITHOUT ANY WARRANTY; without even the implied warranty of
MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
GNU General Public License for more details.

You should have received a copy of the GNU General Public License
along with this program.  If not, see <http://www.gnu.org/licenses/>.
