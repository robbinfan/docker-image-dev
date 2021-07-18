# Copyright Verizon Media. Licensed under the terms of the Apache 2.0 license. See LICENSE in the project root.
FROM centos:7

# Needed to build vespa
RUN yum-config-manager --add-repo https://copr.fedorainfracloud.org/coprs/fanyu83/vespa/repo/epel-7/fanyu83-vespa-epel-7.repo && \
    yum -y install epel-release && \
    yum -y install centos-release-scl && \
    yum -y --setopt=skip_missing_names_on_install=False --enablerepo=epel-testing install \
        git \
        jq \
        yum-utils \
        ccache \
        sudo \
        gflags-devel \
        leveldb-devel

ENV GIT_REPO "https://github.com/robbinfan/vespa.git"

# Install vespa build and runtime dependencies
RUN git clone $GIT_REPO && cd vespa && git -c advice.detachedHead=false checkout vespa-build-image && \
    sed -e '/^BuildRequires:/d' -e '/^Requires: %{name}-/d' -e 's/^Requires:/BuildRequires:/' dist/vespa.spec > dist/vesparun.spec && \
    yum-builddep -y --setopt="centos-sclo-rh-source.skip_if_unavailable=true" dist/vespa.spec dist/vesparun.spec && \
    cd .. && rm -r vespa && \
    alternatives --set java java-11-openjdk.x86_64 && \
    alternatives --set javac java-11-openjdk.x86_64 && \
    yum clean all && rm -rf /var/cache/yum && \
    echo -e "#!/bin/bash\nsource /opt/rh/devtoolset-10/enable" >> /etc/profile.d/enable-devtoolset-10.sh && \
    echo -e "#!/bin/bash\nsource /opt/rh/rh-maven35/enable" >> /etc/profile.d/enable-rh-maven35.sh && \
    echo -e "* soft nproc 409600\n* hard nproc 409600" > /etc/security/limits.d/99-nproc.conf && \
    echo -e "* soft nofile 262144\n* hard nofile 262144" > /etc/security/limits.d/99-nofile.conf

ENV BRPC_REPO "https://github.com/apache/incubator-brpc.git"

RUN source scl_source enable devtoolset-10 && git clone $BRPC_REPO && cd incubator-brpc && git -c advice.detachedHead=false checkout tags/0.9.7 && \
    cmake3 . -DCMAKE_PREFIX_PATH=/opt/vespa-deps && make install && \
    cd .. && rm -r incubator-brpc


# Java requires proper locale for unicode
ENV LANG en_US.UTF-8
