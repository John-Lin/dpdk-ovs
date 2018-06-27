# Open vSwitch with DPDK

Virtio-user with vhost-user backend provide high performance user space container networking.

Please refer to [here](http://dpdk.org/doc/guides/howto/virtio_user_for_container_networking.html) for more information.

## Setup Open vSwitch
```shell
sudo systemctl start ovsdb-server
sudo systemctl start ovs-vswitchd

$ ovs-vsctl --no-wait init
$ ovs-vsctl --no-wait set Open_vSwitch . other_config:dpdk-init=true
$ ovs-vsctl --no-wait set Open_vSwitch . other_config:dpdk-socket-mem="1024"
$ ovs-vsctl --no-wait set Open_vSwitch . other_config:pmd-cpu-mask=0x2
$ ovs-vsctl --no-wait set Open_vSwitch . other_config:max-idle=30000

# userspace datapath with dpdk
$ ovs-vsctl add-br br0 -- set bridge br0 datapath_type=netdev
$ ovs-vsctl add-port br0 dpdk0 -- set Interface dpdk0 type=dpdk options:dpdk-devargs=0000:00:08.0 

# kernel datapath
$ sudo ovs-vsctl add-br br1 -- set bridge br1 datapath_type=system
```

## Install pktgen
```shell
$ sudo -i

$ echo 'export RTE_SDK=$DPDK_DIR' | tee -a /root/.bashrc
$ echo 'export RTE_TARGET=$DPDK_TARGET' | tee -a /root/.bashrc
$ source /root/.bashrc

$ export PKTGEN_DIR=/usr/src/pktgen-3.4.9
$ cd $PKTGEN_DIR
$ RTE_SDK=/usr/src/dpdk-stable-17.11.2 RTE_TARGET=x86_64-native-linuxapp-gcc make
$ ln -s $(pwd)/app/x86_64-native-linuxapp-gcc/pktgen /usr/bin/pktgen
```

## Download vhost-user-net-plugin CNI
```shell
$ git clone https://github.com/intel/vhost-user-net-plugin.git /home/vagrant/go/src/github.com/intel/vhost-user-net-plugin/
```

## Docker images

#### [pktgen-3.4.9](https://hub.docker.com/r/johnlin/pktgen-docker/)
```shell
$ docker pull johnlin/pktgen-docker:3.4.9
```
#### [dpdk-17.11.2 with app testpmd & l2fwd](https://hub.docker.com/r/johnlin/dpdk-docker/)

```shell
$ docker pull johnlin/dpdk-docker:17.11.2
```
