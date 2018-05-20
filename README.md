# Open vSwitch with DPDK

Virtio-user with vhost-user backend provide high performance user space container networking. 

Please refer to [here](http://dpdk.org/doc/guides/howto/virtio_user_for_container_networking.html) for more information.

## Setup Open vSwitch
```shell
$ ovsdb-server --remote=punix:/usr/local/var/run/openvswitch/db.sock --remote=db:Open_vSwitch,Open_vSwitch,manager_options --pidfile --detach --log-file

$ ovs-vsctl --no-wait init
$ ovs-vsctl --no-wait set Open_vSwitch . other_config:dpdk-init=true 
$ ovs-vsctl --no-wait set Open_vSwitch . other_config:dpdk-socket-mem="1024" 
$ ovs-vsctl --no-wait set Open_vSwitch . other_config:pmd-cpu-mask=0x2
$ ovs-vsctl --no-wait set Open_vSwitch . other_config:max-idle=30000

$ ovs-vswitchd  unix:/usr/local/var/run/openvswitch/db.sock --pidfile --detach --log-file

$ ovs-vsctl add-br br0 -- set bridge br0 datapath_type=netdev
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
