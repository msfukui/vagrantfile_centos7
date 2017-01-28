# vagrantfile_centos7

msfukui's Vagrantfile CentOS7 for development enviroment.

# Setup

Example by Windows10 + Virtualbox + cygwin64.

Use Vagrant box centos/7 by Atlas. https://atlas.hashicorp.com/centos/boxes/7

## Preparation

```
$ git clone [repogitory] [hostname]

$ cd [hostname]

$ touch hostname.[hostname]

$ cp ~/.ssh/id_rsa.pub ./keys/[account]_authorized_keys

$ chmod 640 ./keys/[account]_authorized_keys

```

## Init & Up

```
$ vagrant init centos/7

$ vagrant up --provider virtualbox --provision

```

# Contents

## Port forwarding

* ssh: 10022 -> 22

* rails server: 3000 -> 3000

## CentOS configuration

* TimeZone setup(Asia/Tokyo)

* Language setup(Japanese/UTF-8)

* SELinux setup(permissive)

* firewalld enabled

* Install packages

* hostname setup

* motd setup

* account setup

* Update packages
