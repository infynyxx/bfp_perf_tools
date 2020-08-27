# -*- mode: ruby -*-
# vi: set ft=ruby :

# All Vagrant configuration is done below. The "2" in Vagrant.configure
# configures the configuration version (we support older styles for
# backwards compatibility). Please don't change it unless you know what
# you're doing.
Vagrant.configure("2") do |config|
  # The most common configuration options are documented and commented below.
  # For a complete reference, please see the online documentation at
  # https://docs.vagrantup.com.

  # Every Vagrant development environment requires a box. You can search for
  # boxes at https://vagrantcloud.com/search.
  config.vm.box = "centos/8"
  config.vm.box_version = "1905.1"

  # Monkey patch for https://github.com/dotless-de/vagrant-vbguest/issues/367
  # https://github.com/dotless-de/vagrant-vbguest/issues/367#issuecomment-648930897
  class Foo < VagrantVbguest::Installers::CentOS
    def has_rel_repo?
      unless instance_variable_defined?(:@has_rel_repo)
        rel = release_version
        @has_rel_repo = communicate.test("yum repolist")
      end
      @has_rel_repo
    end

    def install_kernel_devel(opts=nil, &block)
      cmd = "yum update kernel -y"
      communicate.sudo(cmd, opts, &block)

      cmd = "yum install -y kernel-devel"
      communicate.sudo(cmd, opts, &block)

      cmd = "shutdown -r now"
      communicate.sudo(cmd, opts, &block)

      begin
        sleep 5
      end until @vm.communicate.ready?
    end
  end
  config.vbguest.installer = Foo

  # Disable automatic box update checking. If you disable this, then
  # boxes will only be checked for updates when the user runs
  # `vagrant box outdated`. This is not recommended.
  # config.vm.box_check_update = false

  # Create a forwarded port mapping which allows access to a specific port
  # within the machine from a port on the host machine. In the example below,
  # accessing "localhost:8080" will access port 80 on the guest machine.
  # NOTE: This will enable public access to the opened port
  # config.vm.network "forwarded_port", guest: 80, host: 8080

  # Create a forwarded port mapping which allows access to a specific port
  # within the machine from a port on the host machine and only allow access
  # via 127.0.0.1 to disable public access
  # config.vm.network "forwarded_port", guest: 80, host: 8080, host_ip: "127.0.0.1"

  # Create a private network, which allows host-only access to the machine
  # using a specific IP.
  # config.vm.network "private_network", ip: "192.168.33.10"

  # Create a public network, which generally matched to bridged network.
  # Bridged networks make the machine appear as another physical device on
  # your network.
  # config.vm.network "public_network"

  # Share an additional folder to the guest VM. The first argument is
  # the path on the host to the actual folder. The second argument is
  # the path on the guest to mount the folder. And the optional third
  # argument is a set of non-required options.
  # config.vm.synced_folder "../data", "/vagrant_data"

  config.vm.synced_folder ".", "/vagrant", type: 'virtualbox'

  # Provider-specific configuration so you can fine-tune various
  # backing providers for Vagrant. These expose provider-specific options.
  # Example for VirtualBox:
  #
  config.vm.provider "virtualbox" do |vb|
    # Customize the amount of memory on the VM:
    vb.memory = "2048"
    vb.cpus = 4

  end
  #
  # View the documentation for the provider you are using for more
  # information on available options.

  # Enable provisioning with a shell script. Additional provisioners such as
  # Ansible, Chef, Docker, Puppet and Salt are also available. Please see the
  # documentation for more information about their specific syntax and use.
  config.vm.provision "shell", inline: <<-SHELL
    # https://github.com/iovisor/bcc/blob/v0.15.0/INSTALL.md#centos---source
    sudo yum install -y epel-release
    sudo yum update -y
    sudo yum groupinstall -y "Development tools"
    sudo yum install -y elfutils-libelf-devel cmake3 git bison flex ncurses-devel
    # for Lua support
    sudo yum install -y luajit luajit-devel
    # python is required for compiling LLVM
    sudo yum install -y python3
    sudo alternatives --set python /usr/bin/python3

    curl  -LO  https://releases.llvm.org/7.0.1/llvm-7.0.1.src.tar.xz
    curl  -LO  https://releases.llvm.org/7.0.1/cfe-7.0.1.src.tar.xz

    tar -xf cfe-7.0.1.src.tar.xz
    tar -xf llvm-7.0.1.src.tar.xz

    mkdir clang-build
    mkdir llvm-build

    cd llvm-build
    cmake3 -G "Unix Makefiles" -DLLVM_TARGETS_TO_BUILD="BPF;X86" \
      -DCMAKE_BUILD_TYPE=Release ../llvm-7.0.1.src
    make
    sudo make install

    cd ../clang-build
    cmake3 -G "Unix Makefiles" -DLLVM_TARGETS_TO_BUILD="BPF;X86" \
      -DCMAKE_BUILD_TYPE=Release ../cfe-7.0.1.src
    make
    sudo make install
    cd ..

    # Install and compile BCC
    git clone https://github.com/iovisor/bcc.git
    git checkout v0.15.0
    mkdir bcc/build
    cd bcc/build
    cmake3 ..
    make
    sudo make install

    # Install BCC
    # https://github.com/iovisor/bpftrace/blob/master/INSTALL.md#CentOS-package
    # https://github.com/fbs/el7-bpf-specs/blob/master/README.md#repository
    sudo curl https://repos.baslab.org/bpftools.repo --output /etc/yum.repos.d/bpftools.repo
    sudo yum -y install bpftrace-static bpftrace-tools bcc-static bcc-tools --nobest
    # somehow it's complaining python3 is not best but it does seem to be work
    # and bpftrace-docs could not be found so, skipping it
  SHELL
end
