# Copyright 2020 Intel Corporation
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#     http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.
# ------------------------------------------------------------------------------

# -------------=== listener build ===-------------

FROM ubuntu:bionic

# Ignore timezone prompt in apt
ENV DEBIAN_FRONTEND=noninteractive

# Add necessary packages
RUN apt-get update \
 && apt-get install -y -q \
    build-essential \
    ca-certificates \
    cmake \
    make \
    pkg-config \
    cmake \
    libprotobuf-dev \
    python3-dev \
    python3-pip \
    python3-toml \
    python3-requests \
    python3-colorlog \
    python3-twisted \
    swig \
    software-properties-common \
    curl \
    dh-autoreconf \
    wget \
    xxd \
    tar \
    python3-tk \
    python3-pycodestyle \
    pycodestyle \
    pylint3 \
    cppcheck \
 && apt-get -y -q upgrade \
 && apt-get clean \
 && rm -rf /var/lib/apt/lists/*

# Install setuptools packages using pip because
# these are not available in apt repository.
RUN pip3 install --upgrade setuptools json-rpc

# Make Python3 default
RUN ln -sf /usr/bin/python3 /usr/bin/python

WORKDIR /tmp

# Build ("Untrusted") OpenSSL
RUN OPENSSL_VER=1.1.1d \
 && wget https://www.openssl.org/source/openssl-$OPENSSL_VER.tar.gz \
 && tar -zxf openssl-$OPENSSL_VER.tar.gz \
 && cd openssl-$OPENSSL_VER/ \
 && ./config \
 && THREADS=8 \
 && make -j$THREADS \
 && make test \
 && make install -j$THREADS \
 && ldconfig \
 && ln -s /etc/ssl/certs/* /usr/local/ssl/certs/ \
 && cd .. \
 && rm -rf openssl-$OPENSSL_VER

COPY . /project/avalon

ENV TCF_HOME=/project/avalon

WORKDIR /project/avalon/common/cpp

RUN mkdir -p build \
  && cd build \
  && cmake .. -DUNTRUSTED_ONLY=1 \
  && make

WORKDIR /project/avalon/common/python

RUN make \
  && make install

#install Avalon SDK  package.
RUN cd /project/avalon/sdk \
   && python setup.py bdist_wheel \
   && pip3 install dist/*.whl

# Build Avalon Listener module next.

WORKDIR /project/avalon/listener/

RUN echo "Building avalon listener\n" \
  && make \
  && make install
