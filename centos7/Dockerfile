FROM nvidia/cuda:8.0-devel-centos7
LABEL maintainer "ULSEE inc. <collin@ulsee.com>"

ENV CUDNN_VERSION 5.1.10
LABEL com.nvidia.cudnn.version="${CUDNN_VERSION}"
RUN CUDNN_DOWNLOAD_SUM=c10719b36f2dd6e9ddc63e3189affaa1a94d7d027e63b71c3f64d449ab0645ce && \
    curl -fsSL http://developer.download.nvidia.com/compute/redist/cudnn/v5.1/cudnn-8.0-linux-x64-v5.1.tgz -O && \
    echo "$CUDNN_DOWNLOAD_SUM  cudnn-8.0-linux-x64-v5.1.tgz" | sha256sum -c --strict - && \
    tar -xzf cudnn-8.0-linux-x64-v5.1.tgz -C /usr/local && \
    rm cudnn-8.0-linux-x64-v5.1.tgz && \
    ldconfig

RUN rpm --import /etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-7

RUn yum -y update && yum -y install epel-release && \
    sed -i -e "s/\]$/\]\npriority=5/g" /etc/yum.repos.d/epel.repo && \
    sed -i -e "s/enabled=1/enabled=0/g" /etc/yum.repos.d/epel.repo

RUN yum -y update && yum -y install --enablerepo=epel cmake \
    wget \
    git \
    vim \
    protobuf protobuf-devel \
    leveldb leveldb-devel \
    snappy-devel \
    opencv opencv-devel \
    boost-devel \
    hdf5 hdf5-devel \
    gflags gflags-devel \
    glog glog-devel \
    lmdb lmdb-devel \
    openblas-devel

RUN yum -y update && yum -y install --enablerepo=epel python-devel \
        python-pip

RUN yum -y update && yum install -y gcc g++ kernel-devel && yum groupinstall -y "Development Tools"
RUN yum clean expire-cache
RUN pip install --upgrade pip

## gflags
#RUN wget https://github.com/schuhschuh/gflags/archive/master.zip && \
#    unzip master.zip && \
#    cd gflags-master && \
#    mkdir build && cd build && \
#    export CXXFLAGS="-fPIC" && cmake .. && make VERBOSE=1 && \
#    make && make install
#
## glog
#RUN wget https://storage.googleapis.com/google-code-archive-downloads/v2/code.google.com/google-glog/glog-0.3.3.tar.gz && \
#    tar zxvf glog-0.3.3.tar.gz && \
#    cd glog-0.3.3 && \
#    ./configure && \
#    make && make install
#
## lmdb
#RUN git clone https://github.com/LMDB/lmdb && \
#    cd lmdb/libraries/liblmdb && \
#    make && make install

ENV CAFFE_ROOT=/workspace/caffe
WORKDIR $CAFFE_ROOT
RUN git clone https://github.com/BVLC/caffe.git . && \
    for req in $(cat python/requirements.txt) pydot; do pip install $req; done && \
    mkdir build && cd build && \
    cmake -DUSE_CUDNN=1 -DBLAS=open .. && \
    make -j"$(nproc)"

ENV PYCAFFE_ROOT $CAFFE_ROOT/python
ENV PYTHONPATH $PYCAFFE_ROOT:$PYTHONPATH
ENV PATH $CAFFE_ROOT/build/tools:$PYCAFFE_ROOT:$PATH
RUN echo "$CAFFE_ROOT/build/lib" >> /etc/ld.so.conf.d/caffe.conf && ldconfig

WORKDIR /workspace
