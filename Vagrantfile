# -*- mode: ruby -*-
# vi: set ft=ruby :

# All Vagrant configuration is done below. The "2" in Vagrant.configure
# configures the configuration version (we support older styles for
# backwards compatibility). Please don't change it unless you know what
# you're doing.

$ubuntu_deps = <<EOF
wget https://apt.llvm.org/llvm.sh
bash ./llvm.sh 12
apt-get -qq update
apt-get -qq install linux-headers-$(uname -r) binutils-dev python
apt-get -qq install bison clang llvm cmake flex g++ git libelf-dev zlib1g-dev libfl-dev systemtap-sdt-dev libclang-12-dev
apt-get -qq install --no-install-recommends pkg-config
EOF

$fedora_deps = <<EOF
dnf builddep -q -y bpftrace
dnf install -q -y git
EOF

$build_bcc = <<EOF
if [ -e /usr/local/lib/libbcc.so ]; then
   echo "libbcc already built, skipping"
   exit 0
fi
git clone https://github.com/iovisor/bcc.git
mkdir -p bcc/build
cd bcc/build
git checkout v0.19.0

cmake .. -DCMAKE_BUILD_TYPE=Debug -DCMAKE_INSTALL_PREFIX=/usr/local \
  -DENABLE_EXAMPLES=0 -DENABLE_TESTS=0 -DENABLE_MAN=0 \
  -DENABLE_LLVM_SHARED=1
make && sudo make install && sudo ldconfig
EOF

$build_libbpf = <<EOF
if [ -e /usr/local/lib/libbpf.so ]; then
   echo "libbpf already built, skipping"
   exit 0
fi
git clone https://github.com/libbpf/libbpf.git
cd libbpf/src
make
sudo PREFIX=/usr/local/ LIBDIR=/usr/local/lib make install install_uapi_headers
EOF

Vagrant.configure("2") do |config|
  boxes = {
    'ubuntu-16.04'     => {
      'image'          => 'bento/ubuntu-18.04',
      'scripts'        => [ $ubuntu_deps, ],
    }
}
  boxes.each do | name, params |
    config.vm.define name do |box|
      box.vm.box = params['image']
      box.vm.provider "parallels" do |v|
        v.memory = 2048
        v.cpus = 2
        if params['fix_console'] == 1
          v.customize ["modifyvm", :id, "--uart1", "0x3F8", "4"]
          v.customize ["modifyvm", :id, "--uartmode1", "file", "./#{name}_ttyS0.log"]
        end
      end
      box.vm.provider :libvirt do |v|
        v.memory = 2048
        v.cpus = 2
      end
      (params['scripts'] || []).each do |script|
        box.vm.provision :shell, inline: script
      end
      unless ENV['SKIP_BCC_BUILD'] || (params['skip_bcc_build'] == 1)
        box.vm.provision :shell, privileged: false, inline: $build_bcc
      end
      unless ENV['SKIP_LIBBPF_BUILD'] || (params['skip_libbpf_build'] == 1)
        box.vm.provision :shell, privileged: false, inline: $build_libbpf
      end
      # Install golang and setup its environment
      box.vm.provision "shell", privileged: false, inline: <<-SHELL
        curl -O https://storage.googleapis.com/golang/go1.16.4.linux-amd64.tar.gz
        mkdir -p go/src go/bin go/pkg
        echo "
        export GOPATH=$HOME/go
        export GOROOT=/usr/local/go
        export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin
        " >> .bashrc
      SHELL

      box.vm.provision "shell", inline: <<-SHELL
        tar -C /usr/local -xzf go1.16.4.linux-amd64.tar.gz
      SHELL
      config.vm.post_up_message = <<-HEREDOC

      HEREDOC
    end
  end
end

  # The most common configuration options are documented and commented below.
  # For a complete reference, please see the online documentation at
  # https://docs.vagrantup.com.



  # Every Vagrant development environment requires a box. You can search for
  # boxes at https://vagrantcloud.com/search.
  
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

  # Enable provisioning with a shell script. Additional provisioners such as
  # Ansible, Chef, Docker, Puppet and Salt are also available. Please see the
  # documentation for more information about their specific syntax and use.
  # config.vm.provision "shell", inline: <<-SHELL
  #   apt-get update
  #   apt-get install -y apache2
  # SHELL
