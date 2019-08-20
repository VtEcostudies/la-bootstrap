## Vermont Atlas of Life: Ansible Inventories

These are some generated inventories to use to set up some machines on EC2 or other cloud provider with LA software.

### Initial Setup

To use this, add the following into your `/etc/hosts` (of your working machine, and new service machine/s) and/or in your vtatlasoflife.org `DNS`. So these hostname should be accessible from your local working machine but also remotely between each machine/s so the hostname should resolve correctly.

```
12.12.12.12  vtatlasoflife.org
12.12.12.13  collectory.vtatlasoflife.org
12.12.12.14  biocache.vtatlasoflife.org
12.12.12.15  biocache-ws.vtatlasoflife.org
12.12.12.16  bie.vtatlasoflife.org
12.12.12.17  bie-ws.vtatlasoflife.org
12.12.12.18  images.vtatlasoflife.org
12.12.12.19  lists.vtatlasoflife.org
12.12.12.20  regions.vtatlasoflife.org
12.12.12.21  logger.vtatlasoflife.org
12.12.12.22  index.vtatlasoflife.org
12.12.12.23  auth.vtatlasoflife.org
12.12.12.24  spatial.vtatlasoflife.org
```

You'll need to replace `12.12.12.1` etc with the IP address of some new Ubuntu 16 instance in your provider.

These machines should have an user `ubuntu` with `sudo` permissions.

You should generate and use some ssh key and copy `~/.ssh/MyKey.pub` in thouse machines under `~ubuntu/.ssh/authorized_keys` (via `ssh-copy-id` for avoid issues).

You can test your initial setup with some `ssh` command like:
```
ssh -i ~/.ssh/MyKey.pem ubuntu@12.12.12.1 sudo ls /root
```
that should work.

### Run ansible

With access to this machine/s you can run ansible:

```
export AI=<location-of-your-cloned-ala-install-repo>

#  For this demo to run well, we recommend a machine of 16GB RAM, 4 CPUs.

ansible-playbook --private-key ~/.ssh/MyKey.pem -u ubuntu -i vtatlasoflife-inventory.yml -i vtatlasoflife-local-extras.yml -i vtatlasoflife-local-passwords.yml $AI/ansible/ala-demo.yml --limit vtatlasoflife.org

ansible-playbook --private-key ~/.ssh/MyKey.pem -u ubuntu -i vtatlasoflife-inventory.yml -i vtatlasoflife-local-extras.yml -i vtatlasoflife-local-passwords.yml $AI/ansible/collectory-standalone.yml --limit collectory.vtatlasoflife.org
ansible-playbook --private-key ~/.ssh/MyKey.pem -u ubuntu -i vtatlasoflife-inventory.yml -i vtatlasoflife-local-extras.yml -i vtatlasoflife-local-passwords.yml $AI/ansible/biocache-hub-standalone.yml --limit biocache.vtatlasoflife.org
ansible-playbook --private-key ~/.ssh/MyKey.pem -u ubuntu -i vtatlasoflife-inventory.yml -i vtatlasoflife-local-extras.yml -i vtatlasoflife-local-passwords.yml $AI/ansible/biocache-service-clusterdb.yml --limit biocache-ws.vtatlasoflife.org
ansible-playbook --private-key ~/.ssh/MyKey.pem -u ubuntu -i vtatlasoflife-inventory.yml -i vtatlasoflife-local-extras.yml -i vtatlasoflife-local-passwords.yml $AI/ansible/bie-hub.yml --limit bie.vtatlasoflife.org
ansible-playbook --private-key ~/.ssh/MyKey.pem -u ubuntu -i vtatlasoflife-inventory.yml -i vtatlasoflife-local-extras.yml -i vtatlasoflife-local-passwords.yml $AI/ansible/bie-index.yml --limit bie-ws.vtatlasoflife.org
ansible-playbook --private-key ~/.ssh/MyKey.pem -u ubuntu -i vtatlasoflife-inventory.yml -i vtatlasoflife-local-extras.yml -i vtatlasoflife-local-passwords.yml $AI/ansible/image-service.yml --limit images.vtatlasoflife.org
ansible-playbook --private-key ~/.ssh/MyKey.pem -u ubuntu -i vtatlasoflife-inventory.yml -i vtatlasoflife-local-extras.yml -i vtatlasoflife-local-passwords.yml $AI/ansible/species-list-standalone.yml --limit lists.vtatlasoflife.org
ansible-playbook --private-key ~/.ssh/MyKey.pem -u ubuntu -i vtatlasoflife-inventory.yml -i vtatlasoflife-local-extras.yml -i vtatlasoflife-local-passwords.yml $AI/ansible/regions-standalone.yml --limit regions.vtatlasoflife.org
ansible-playbook --private-key ~/.ssh/MyKey.pem -u ubuntu -i vtatlasoflife-inventory.yml -i vtatlasoflife-local-extras.yml -i vtatlasoflife-local-passwords.yml $AI/ansible/logger-standalone.yml --limit logger.vtatlasoflife.org
ansible-playbook --private-key ~/.ssh/MyKey.pem -u ubuntu -i vtatlasoflife-inventory.yml -i vtatlasoflife-local-extras.yml -i vtatlasoflife-local-passwords.yml $AI/ansible/solr7-standalone.yml --limit index.vtatlasoflife.org
ansible-playbook --private-key ~/.ssh/MyKey.pem -u ubuntu -i vtatlasoflife-inventory.yml -i vtatlasoflife-local-extras.yml -i vtatlasoflife-cas-inventory.yml -i vtatlasoflife-cas-local-extras.yml -i vtatlasoflife-local-passwords.yml --extra-vars "ala_install_repo=$AI" $AI/ansible/cas5-standalone.yml --limit auth.vtatlasoflife.org
ansible-playbook --private-key ~/.ssh/MyKey.pem -u ubuntu -i vtatlasoflife-inventory.yml -i vtatlasoflife-local-extras.yml -i vtatlasoflife-local-passwords.yml $AI/ansible/biocache-backend.yml --limit vtatlasoflife.org
ansible-playbook --private-key ~/.ssh/MyKey.pem -u ubuntu -i vtatlasoflife-inventory.yml -i vtatlasoflife-local-extras.yml -i vtatlasoflife-local-passwords.yml $AI/ansible/biocache-cli.yml --limit vtatlasoflife.org
ansible-playbook --private-key ~/.ssh/MyKey.pem -u ubuntu -i vtatlasoflife-inventory.yml -i vtatlasoflife-local-extras.yml -i vtatlasoflife-spatial-inventory.yml -i vtatlasoflife-spatial-local-extras.yml -i vtatlasoflife-local-passwords.yml $AI/ansible/spatial.yml --limit spatial.vtatlasoflife.org
```
#### ansible-playbook wrapper

Also there is the utility `ansiblew` an `ansible-playbook` wrapper that can help you to exec this commands and can be easily modificable by you to your needs. It depends on `python-docopt` package. Help output:

```
$ ./ansiblew --help

This is an ansible wrapper to help you to exec the different playbooks with your
inventories.

By default don't exec nothing only show the commands. With --nodryrun you can exec
the real commands.

With 'main' only operates over your main host.

Usage:
   ansiblew --alainstall=<dir_of_ala_install_repo> [options] [ main | collectory | ala_hub | biocache_service | ala_bie | bie_index | images | lists | regions | logger | solr | cas | biocache_backend | biocache_cli | spatial |  all ]
   ansiblew -h | --help
   ansiblew -v | --version

Options:
  --nodryrun             Exec the ansible-playbook commands
  -p --properties        Only update properties
  -l --limit=<hosts>     Limit to some inventories hosts
  -s --skip=<tags>       Skip tags
  -h --help              Show help options.
  -d --debug             Show debug info.
  -v --version           Show ansiblew version.
----
ansiblew 0.1.0
Copyright (C) 2019 living-atlases.gbif.org
Apache 2.0 License
```
So you can install the CAS service or the spatial service with commands like:

```bash
./ansiblew --alainstall=../ala-install cas --nodryrun
```

and

```bash
./ansiblew --alainstall=../ala-install spatial --nodryrun
```

or all the services with something like:

```bash
./ansiblew --alainstall=../ala-install all --nodryrun
```

### Rerunning the generator

You can rerun the generator with the option `--replay` to use all the previous responses and regenerate the inventories with some modification (if for instance you want to add a new service, or using a new version of this generator with improvements).

We recommend to override and set variables adding then to `vtatlasoflife-local-extras.yml` and `vtatlasoflife-spatial-local-extras.yml` without modify the generated `vtatlasoflife-inventory.yml` and `vtatlasoflife-spatial-inventory.yml`, so you can rerun the generator in the future without lost local changes. The `*-local-extras.sample` files will be updated with future versions of this generator, so you can compare from time to time these samples with your `*-local-extras.yml` files to add new vars, etc.

### Urls of your LA node

- Main landing page: http://vtatlasoflife.org
- Collections: http://collectory.vtatlasoflife.org
- Collections administration: http://collectory.vtatlasoflife.org/admin
- Biocache (occurrences): http://biocache.vtatlasoflife.org
- Biocache administration: http://biocache.vtatlasoflife.org/admin
- Biocache webservice: http://biocache-ws.vtatlasoflife.org
- Species: http://bie.vtatlasoflife.org
- Species webservice: http://bie-ws.vtatlasoflife.org
- Species webservice administration: http://bie-ws.vtatlasoflife.org/admin
- SOLR non-public web interface: http://index.vtatlasoflife.org:8983 (You should use ssh port redirection to access this)
- CAS Auth system: https://auth.vtatlasoflife.org/cas
- User details: https://auth.vtatlasoflife.org/userdetails
- User details administration: https://auth.vtatlasoflife.org/userdetails/admin
- Apikey management: https://auth.vtatlasoflife.org/apikey/
- CAS management administration: https://auth.vtatlasoflife.org/cas-management/
- Logger: http://logger.vtatlasoflife.org/
- Logger administration: http://logger.vtatlasoflife.org/admin
- Species list: http://lists.vtatlasoflife.org
- Species list administration: http://lists.vtatlasoflife.org/admin
- Regions: http://regions.vtatlasoflife.org
- Regions administration: http://regions.vtatlasoflife.org/alaAdmin
- Spatial: http://spatial.vtatlasoflife.org
- Spatial Webservice: http://spatial.vtatlasoflife.org/ws
- Spatial Geoserver: http://spatial.vtatlasoflife.org/geoserver/

### TODO

[ ] Whitelist your IPs in the logger admin interface to allow correct log recollection
