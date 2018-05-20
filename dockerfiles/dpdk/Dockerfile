FROM ubuntu:16.04

RUN apt-get update -qq && \
    apt-get install -qq -y numactl && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*

ENV DPDK_VERSION 17.11.2
WORKDIR /root/dpdk
COPY dpdk-stable-${DPDK_VERSION} /root/dpdk/.
ENV PATH "$PATH:/root/dpdk/x86_64-native-linuxapp-gcc/app/"

