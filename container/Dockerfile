FROM docker.io/library/centos:7 AS zeromq_builder

ARG ZEROMQ_VER=4.3.3

RUN yum -y update && yum -y install gcc-c++ make

ADD https://github.com/zeromq/libzmq/releases/download/v${ZEROMQ_VER}/zeromq-${ZEROMQ_VER}.tar.gz /

RUN tar xf zeromq-${ZEROMQ_VER}.tar.gz

WORKDIR /zeromq-${ZEROMQ_VER}

RUN ./configure --prefix=/opt/zeromq --disable-shared --enable-static --without-libsodium --disable-libunwind
RUN make
RUN make install

FROM docker.io/library/centos:7

# Kept as 1.13 as Go 1.14 does not play well with the pinned version of zmq4
# https://github.com/pebbe/zmq4/issues/161
ARG GO_VER=1.13.8
ARG CUDA_VER=8.0.61

ADD https://dl.google.com/go/go${GO_VER}.linux-amd64.tar.gz \
    https://developer.download.nvidia.com/compute/cuda/repos/rhel7/x86_64/cuda-repo-rhel7-${CUDA_VER}-1.x86_64.rpm /

RUN yum -y update && yum -y install gcc-c++ make && \
    tar xf go${GO_VER}.linux-amd64.tar.gz && \
    rpm -i cuda-repo-rhel7-${CUDA_VER}-1.x86_64.rpm && \
    bash -c 'TMP=${CUDA_VER%.*} ; yum -y install cuda-nvml-dev-${TMP/./-}'

COPY --from=zeromq_builder /opt/zeromq /opt/zeromq

ENV PATH="/go/bin:${PATH}" GOPATH=/gopath PKG_CONFIG_PATH=/opt/zeromq/lib/pkgconfig

WORKDIR /gopath/src/github.com/patwie/cluster-smi

CMD ["bash"]
