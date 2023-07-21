FROM mpioperator/mpi-pi:latest

ENV TENSORFLOW_VERSION=2.8.4
ENV PYTORCH_VERSION=1.12.1
ENV TORCHVISION_VERSION=0.13.1
ENV MXNET_VERSION=1.9.1
ENV HOROVOD_VERSION=0.28.1
ENV NUMPY_VERSION=1.23.5

# Python 2.7 or 3.6 is supported by Ubuntu Bionic out of the box
ARG python=3
ENV PYTHON_VERSION=${python}

# Set default shell to /bin/bash
SHELL ["/bin/bash", "-cu"]

RUN apt-get update && apt-get install -y --allow-downgrades --allow-change-held-packages --no-install-recommends \
        build-essential \
        cmake \
        g++ \
        git \
        curl \
        vim \
        wget \
        ca-certificates \
        libjpeg-dev \
        libpng-dev \
        python${PYTHON_VERSION} \
        python${PYTHON_VERSION}-dev \
        librdmacm1 \
        libibverbs1 \
        ibverbs-providers

RUN if [[ "${PYTHON_VERSION}" == "3" ]]; then \
        apt-get install -y python${PYTHON_VERSION}-distutils; \
    fi
#RUN ln -s /usr/bin/python${PYTHON_VERSION} /usr/bin/python3

RUN curl -O https://bootstrap.pypa.io/pip/3.6/get-pip.py && \
    python3 get-pip.py && \
    rm get-pip.py

# Install TensorFlow, Keras, PyTorch and MXNet
RUN pip install future typing
RUN pip install numpy==${NUMPY_VERSION} \
        tensorflow==${TENSORFLOW_VERSION} \
        keras \
        h5py
RUN pip install torch==${PYTORCH_VERSION} torchvision==${TORCHVISION_VERSION}
RUN pip install mxnet==${MXNET_VERSION}

# Install Open MPI
RUN mkdir /tmp/openmpi && \
    cd /tmp/openmpi && \
    wget https://download.open-mpi.org/release/open-mpi/v4.1/openmpi-4.1.0.tar.gz && \
    tar zxf openmpi-4.1.0.tar.gz && \
    cd openmpi-4.1.0 && \
    ./configure --enable-orterun-prefix-by-default && \
    make -j $(nproc) all && \
    make install && \
    ldconfig && \
    rm -rf /tmp/openmpi

# Install Horovod
RUN HOROVOD_WITH_TENSORFLOW=1 HOROVOD_WITH_PYTORCH=1 HOROVOD_WITH_MXNET=1 \
    pip install --no-cache-dir horovod==${HOROVOD_VERSION}

# Install OpenSSH for MPI to communicate between containers
RUN apt-get install -y --no-install-recommends openssh-client openssh-server && \
    mkdir -p /var/run/sshd

# Allow OpenSSH to talk to containers without asking for confirmation
# by disabling StrictHostKeyChecking.
# mpi-operator mounts the .ssh folder from a Secret. For that to work, we need
# to disable UserKnownHostsFile to avoid write permissions.
# Disabling StrictModes avoids directory and files read permission checks.
RUN sed -i 's/[ #]\(.*StrictHostKeyChecking \).*/ \1no/g' /etc/ssh/ssh_config && \
#    sed -i 's/[ #]\(.*Port \).*/ \12222/g' /etc/ssh/ssh_config && \
    echo "    UserKnownHostsFile /dev/null" >> /etc/ssh/ssh_config && \
    sed -i 's/#\(StrictModes \).*/\1no/g' /etc/ssh/sshd_config && \
#    sed -i 's/#\(Port \).*/\12222/g' /etc/ssh/sshd_config && \
    sed -i 's/#\(PermitRootLogin \).*/\1yes/g' /etc/ssh/sshd_config && \
#    sed -i 's/#\(TCPKeepAlive \).*/\1yes/g' /etc/ssh/sshd_config && \
    sed -i 's/#\(PubkeyAuthentication \).*/\1yes/g' /etc/ssh/sshd_config

WORKDIR "/examples"

# Get example python scripts
COPY tensorflow_mnist.py /examples
