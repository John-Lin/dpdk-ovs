# -*- mode: ruby -*-
# vi: set ft=ruby :

# Open vSwitch 	DPDK
# 2.6.x         16.07.2
# 2.7.x         16.11.6
# 2.8.x         17.05.2
# 2.9.x         17.11.2

$script = <<SCRIPT
set -e -x -u
# Configure hugepages
# You can later check if this change was successful with `cat /proc/meminfo`
# Hugepages setup should be done as early as possible after boot
echo 'vm.nr_hugepages=1024' | sudo tee /etc/sysctl.d/hugepages.conf
sudo mount -t hugetlbfs none /dev/hugepages
sudo sysctl -w vm.nr_hugepages=1024

# Name of network interface provisioned for DPDK to bind
export NET_IF_NAME=enp0s8

sudo apt-get -qq update
sudo apt-get -y -qq install vim git clang doxygen hugepages build-essential libnuma-dev libpcap-dev inux-headers-`uname -r` dh-autoreconf libssl-dev libcap-ng-dev openssl python python-pip htop
sudo pip install six

#### Install Golang
wget --quiet https://storage.googleapis.com/golang/go1.9.1.linux-amd64.tar.gz
sudo tar -zxf go1.9.1.linux-amd64.tar.gz -C /usr/local/
echo 'export GOROOT=/usr/local/go' >> /home/vagrant/.bashrc
echo 'export GOPATH=$HOME/go' >> /home/vagrant/.bashrc
echo 'export PATH=$PATH:$GOROOT/bin:$GOPATH/bin' >> /home/vagrant/.bashrc
source /home/vagrant/.bashrc
mkdir -p /home/vagrant/go/src
rm -rf /home/vagrant/go1.9.1.linux-amd64.tar.gz

#### Install Docker
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
sudo apt-get -qq update
sudo apt-get -qq install -y docker-ce

#### Download DPDK, Open vSwitch and pktgen source
wget --quiet https://fast.dpdk.org/rel/dpdk-17.11.2.tar.xz
sudo tar xf dpdk-17.11.2.tar.xz -C /usr/src/

wget --quiet http://openvswitch.org/releases/openvswitch-2.9.0.tar.gz
sudo tar -zxf openvswitch-2.9.0.tar.gz -C /usr/src/

wget --quiet http://www.dpdk.org/browse/apps/pktgen-dpdk/snapshot/pktgen-3.4.9.tar.gz
sudo tar -zxf pktgen-3.4.9.tar.gz -C /usr/src/

#### Install DPDK
echo 'export DPDK_DIR=/usr/src/dpdk-stable-17.11.2' | sudo tee -a /root/.bashrc
echo 'export LD_LIBRARY_PATH=$DPDK_DIR/x86_64-native-linuxapp-gcc/lib' | sudo tee -a /root/.bashrc
echo 'export DPDK_TARGET=x86_64-native-linuxapp-gcc' | sudo tee -a /root/.bashrc
echo 'export DPDK_BUILD=$DPDK_DIR/$DPDK_TARGET' | sudo tee -a /root/.bashrc
export DPDK_DIR=/usr/src/dpdk-stable-17.11.2
export LD_LIBRARY_PATH=$DPDK_DIR/x86_64-native-linuxapp-gcc/lib
export DPDK_TARGET=x86_64-native-linuxapp-gcc
export DPDK_BUILD=$DPDK_DIR/$DPDK_TARGET

cd $DPDK_DIR
# Build and install the DPDK library
sudo make install T=$DPDK_TARGET DESTDIR=install
# (Optional) Export the DPDK shared library location
sudo sed -i 's/CONFIG_RTE_BUILD_SHARED_LIB=n/CONFIG_RTE_BUILD_SHARED_LIB=y/g' ${DPDK_DIR}/config/common_base

# Install kernel modules
sudo modprobe uio
sudo insmod ${DPDK_DIR}/x86_64-native-linuxapp-gcc/kmod/igb_uio.ko

# Make uio and igb_uio installations persist across reboots
sudo ln -sf ${DPDK_DIR}/x86_64-native-linuxapp-gcc/kmod/igb_uio.ko /lib/modules/`uname -r`
sudo depmod -a
echo "uio" | sudo tee -a /etc/modules
echo "igb_uio" | sudo tee -a /etc/modules

# Bind secondary network adapter
# Note that this NIC setup does not persist across reboots
sudo ifconfig ${NET_IF_NAME} down
sudo ${DPDK_DIR}/usertools/dpdk-devbind.py --bind=igb_uio ${NET_IF_NAME}
sudo ${DPDK_DIR}/usertools/dpdk-devbind.py --status

#### Install Open vSwitch
export OVS_DIR=/usr/src/openvswitch-2.9.0
cd $OVS_DIR
./boot.sh
CFLAGS='-march=native' ./configure --with-dpdk=$DPDK_BUILD
make && sudo make install
sudo mkdir -p /usr/local/etc/openvswitch
sudo mkdir -p /usr/local/var/run/openvswitch
sudo mkdir -p /usr/local/var/log/openvswitch
sudo ovsdb-tool create /usr/local/etc/openvswitch/conf.db vswitchd/vswitch.ovsschema

echo 'export PATH=$PATH:/usr/local/share/openvswitch/scripts' | sudo tee -a /root/.bashrc

#### Cleanup
rm -rf /home/vagrant/openvswitch-2.9.0.tar.gz /home/vagrant/dpdk-17.11.2.tar.xz /home/vagrant/go1.9.1.linux-amd64.tar.gz /home/vagrant/pktgen-3.4.9.tar.gz
SCRIPT

Vagrant.configure("2") do |config|
  config.vm.box = "ubuntu/xenial64"
  config.vm.hostname = "devstack"
  config.vm.provision "shell", privileged: false, inline: $script
  config.vm.network "private_network", ip: "192.168.200.100"

  # Create a private network, which allows host-only access to the machine
  # using a specific IP.
  # This option is needed otherwise the Intel DPDK takes over the entire adapter
  config.vm.provider :virtualbox do |v|
      v.customize ["modifyvm", :id, "--cpus", 2]
      v.customize ["modifyvm", :id, "--memory", 4096]
      v.customize ['modifyvm', :id, '--nicpromisc2', 'allow-all']

      # Configure VirtualBox to enable passthrough of SSE 4.1 and SSE 4.2
      # instructions, according to this:
      # https://www.virtualbox.org/manual/ch09.html#sse412passthrough
      # This step is fundamental otherwise DPDK won't build.
      # It is possible to verify in the guest OS that these changes took effect
      # by running `cat /proc/cpuinfo` and checking that `sse4_1` and `sse4_2`
      # are listed among the CPU flags
      v.customize ["setextradata", :id, "VBoxInternal/CPUM/SSE4.1", "1"]
      v.customize ["setextradata", :id, "VBoxInternal/CPUM/SSE4.2", "1"]
  end # end provider
end
