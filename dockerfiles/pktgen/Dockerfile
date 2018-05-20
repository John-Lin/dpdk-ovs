FROM ubuntu:16.04

RUN apt-get update -qq && \
    apt-get install -qq -y numactl libpcap-dev && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*

ENV DPDK_VERSION 17.11.2
ENV PKTGEN_VERSION 3.4.9
WORKDIR /root/dpdk
COPY dpdk-stable-${DPDK_VERSION} /root/dpdk/.
COPY pktgen-${PKTGEN_VERSION} /root/pktgen/.

RUN ln -s /root/pktgen/app/x86_64-native-linuxapp-gcc/pktgen /usr/bin/pktgen
RUN ln -s /usr/lib/x86_64-linux-gnu/libpcap.so /usr/lib/x86_64-linux-gnu/libpcap.so.1
ENV PATH "$PATH:/root/dpdk/x86_64-native-linuxapp-gcc/app/"
