# -*- mode: ruby -*-
# vi: set ft=ruby :

# All Vagrant configuration is done below. The "2" in Vagrant.configure
# configures the configuration version (we support older styles for
# backwards compatibility). Please don't change it unless you know what
# you're doing.
Vagrant.configure('2') do |config|
  # The most common configuration options are documented and commented below.
  # For a complete reference, please see the online documentation at
  # https://docs.vagrantup.com.

  # Every Vagrant development environment requires a box. You can search for
  # boxes at https://atlas.hashicorp.com/search.
  config.vm.box = 'centos/7'

  # Disable automatic box update checking. If you disable this, then
  # boxes will only be checked for updates when the user runs
  # `vagrant box outdated`. This is not recommended.
  # config.vm.box_check_update = false

  # Create a forwarded port mapping which allows access to a specific port
  # within the machine from a port on the host machine. In the example below,
  # accessing "localhost:8080" will access port 80 on the guest machine.
  # config.vm.network "forwarded_port", guest: 80, host: 8080
  config.vm.network 'forwarded_port', guest: 22, host: 10_022
  config.vm.network 'forwarded_port', guest: 3000, host: 3000

  # Create a private network, which allows host-only access to the machine
  # using a specific IP.
  config.vm.network 'private_network', ip: '192.168.33.10'

  # Create a public network, which generally matched to bridged network.
  # Bridged networks make the machine appear as another physical device on
  # your network.
  # config.vm.network "public_network"

  # Share an additional folder to the guest VM. The first argument is
  # the path on the host to the actual folder. The second argument is
  # the path on the guest to mount the folder. And the optional third
  # argument is a set of non-required options.
  # config.vm.synced_folder "../data", "/vagrant_data"

  # Provider-specific configuration so you can fine-tune various
  # backing providers for Vagrant. These expose provider-specific options.
  # Example for VirtualBox:
  #
  # config.vm.provider "virtualbox" do |vb|
  #   # Display the VirtualBox GUI when booting the machine
  #   vb.gui = true
  #
  #   # Customize the amount of memory on the VM:
  #   vb.memory = "1024"
  # end
  #
  # View the documentation for the provider you are using for more
  # information on available options.

  # Define a Vagrant Push strategy for pushing to Atlas. Other push strategies
  # such as FTP and Heroku are also available. See the documentation at
  # https://docs.vagrantup.com/v2/push/atlas.html for more information.
  # config.push.define "atlas" do |push|
  #   push.app = "YOUR_ATLAS_USERNAME/YOUR_APPLICATION_NAME"
  # end

  # Enable provisioning with a shell script. Additional provisioners such as
  # Puppet, Chef, Ansible, Salt, and Docker are also available. Please see the
  # documentation for more information about their specific syntax and use.
  config.vm.provision 'shell', inline: <<-SHELL
  # set timezone
  sudo timedatectl set-timezone Asia/Tokyo
  # set locales
  sudo yum install -y ibus-kkc vlgothic-*
  sudo localectl set-locale LANG=ja_JP.utf8
  # set selinux(permissive)
  sudo setenforce 0
  sudo cp /vagrant/conf/selinux.base /etc/selinux/config
  sudo chmod 644 /etc/selinux/config
  # enabled firewalld
  wanted_ports=("3000/tcp")
  enable_ports=$(sudo firewall-cmd --list-ports)
  echo "wanted_ports:[$wanted_ports]"
  echo "enable_ports:[$enable_ports]"
  for port in $wanted_ports; do
    if ! `echo ${enable_ports[@]} | grep -q "${port}"`; then
      sudo firewall-cmd --add-port=${port} --zone=public --permanent
    fi
  done
  enable_ports=$(sudo firewall-cmd --list-ports)
  echo "wanted_ports:[$wanted_ports]"
  echo "enable_ports:[$enable_ports]"
  for port in $enable_ports; do
    if ! `echo ${wanted_ports[@]} | grep -q "${port}"`; then
      sudo firewall-cmd --remove-port=${port} --zone=public --permanent
    fi
  done
  sudo systemctl start firewalld
  sudo systemctl enable firewalld
  sudo systemctl restart firewalld
  # install packages.
  sudo yum install -y gcc make kernel-devel
  sudo yum install -y git tcsh ctags wget
  sudo yum install -y readline-devel openssl-devel libxml2-devel libxslt-devel
  sudo yum install -y sqlite-devel postgresql-devel
  sudo yum install -y java-1.8.0-openjdk gcc-c++
  #keychain_setup=$(yum list installed keychain | awk '{ print $1;}' | tail -1)
  #if [ ! ${keychain_setup} = "keychain.noarch" ]; then
  #  sudo yum install -y https://kojipkgs.fedoraproject.org//packages/keychain/2.8.0/3.fc24/noarch/keychain-2.8.0-3.fc24.noarch.rpm
  #fi
  # setup sudoers
  #sudo cp /vagrant/conf/sudoers.base /etc/sudoers
  #sudo chmod 440 /etc/sudoers
  # setup iptables
  #sudo cp /vagrant/conf/iptables.base /etc/sysconfig/iptables
  #sudo chmod 640 /etc/sysconfig/iptables
  #sudo service iptables restart
  # setup network, hostname
  myhost=$(ls -1 /vagrant/hostname.* | head -1 | cut -d'.' -f2-)
  if [ "${myhost}" = "" ]; then
    myhost="centos7"
  fi
  #sudo cp /vagrant/conf/network.base /etc/sysconfig/network
  #sudo sed -i "s/%myhost%/${myhost}/g" /etc/sysconfig/network
  #sudo chmod 644 /etc/sysconfig/network
  #sudo service network restart
  #sudo cp /vagrant/conf/hosts.base /etc/hosts
  #sudo hostname ${myhost}
  sudo nmcli general hostname ${myhost}
  # setup motd
  sudo cp /vagrant/conf/motd.base /etc/motd
  sudo chmod 644 /etc/motd
  # setup epel
  sudo yum install epel-release
  sudo cp -p /etc/yum.repos.d/epel.repo /etc/yum.repos.d/epel.repo.old
  sudo sed -e "s/enabled=1/enabled=0/g" /etc/yum.repos.d/epel.repo > /etc/yum.repos.d/epel.repo.new
  sudo mv /etc/yum.repos.d/epel.repo.new /etc/yum.repos.d/epel.repo
  # account setup
  for myaccount in $(ls -1 /vagrant/keys | cut -d'_' -f1); do
    if [ ! -d /home/${myaccount} ]; then
      sudo useradd -G wheel ${myaccount}
      sudo mkdir -p /home/${myaccount}/.ssh
      sudo chmod 700 /home/${myaccount}/.ssh
      sudo cp /vagrant/keys/${myaccount}_authorized_keys /home/${myaccount}/.ssh/authorized_keys
      sudo chmod 600 /home/${myaccount}/.ssh/authorized_keys
      sudo chown -R ${myaccount}:${myaccount} /home/${myaccount}/.ssh
    fi
  done
  # setup 'view' command.
  cd /usr/bin; sudo rm view; sudo ln -s vim view
  # setup vim
  if [ ! -f /usr/local/bin/vim ]; then
    sudo yum install -y lua-devel
    sudo mkdir -p /usr/local/src
    cd /usr/local/src
    if [ ! -d /usr/local/src/vim ]; then
      sudo git clone https://github.com/vim/vim.git
    fi
    sudo sh -c "cd vim/; git pull; make distclean; ./configure --prefix=/usr/local --with-features=huge --enable-multibyte --enable-luainterp --enable-cscope --enable-fail-if-missing; make; make install"
  fi
  # update packages.
  sudo yum update -y
  SHELL
end
