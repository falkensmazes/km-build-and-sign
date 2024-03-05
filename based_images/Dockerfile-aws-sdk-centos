FROM registry.access.redhat.com/ubi8/ubi AS sdk-builder
MAINTAINER Enrique Belarte Luque <ebelarte@redhat.com>
# Install packages
RUN INSTALL_PKGS="clang unzip cmake zlib-devel openssl-devel libcurl-devel git p11-kit-devel json-c-devel" && \
    yum install -y --setopt=tsflags=nodocs $INSTALL_PKGS && \
    rpm -V $INSTALL_PKGS && \
    yum -y clean all 
# Install AWS SDK C++ 
RUN curl -o ~/aws-sdk-cpp.tar.gz -L https://github.com/aws/aws-sdk-cpp/archive/1.9.332.tar.gz && \
    mkdir ~/aws-sdk-cpp-src && \
    tar -C ~/aws-sdk-cpp-src --strip-components=1 -zxf ~/aws-sdk-cpp.tar.gz && \
    cd ~/aws-sdk-cpp-src && ./prefetch_crt_dependency.sh && \
    mkdir ~/aws-sdk-cpp-src/sdk_build && \
    cd ~/aws-sdk-cpp-src/sdk_build && \
    cmake .. -DCMAKE_BUILD_TYPE=Release -DBUILD_ONLY="kms;acm-pca" -DENABLE_TESTING=OFF -DCMAKE_INSTALL_PREFIX=$HOME/aws-sdk-cpp -DBUILD_SHARED_LIBS=OFF && \
    make && make install 
# Clone PKCS11 implementation to use AWS KMS as backend
RUN git clone https://github.com/JackOfMostTrades/aws-kms-pkcs11.git && \
    cd aws-kms-pkcs11 && \
    AWS_SDK_PATH=~/aws-sdk-cpp make

FROM registry.access.redhat.com/ubi8/ubi
# Set enviroment variables for AWS auth
ENV AWS_ACCESS_KEY_ID=your_access_key_id
ENV AWS_SECRET_ACCESS_KEY=your_secret_access_key
ENV AWS_DEFAULT_REGION=eu-west-3
ENV AWS_KMS_TOKEN=xxxxxxx-xxxx-xxxxxx-xxxxx
# Install packages
RUN INSTALL_PKGS="openssl openssl-pkcs11 kernel-devel unzip less" && \
    yum install -y --setopt=tsflags=nodocs $INSTALL_PKGS && \
    rpm -V $INSTALL_PKGS && \
    yum -y clean all
# Install AWS CLI
RUN curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip" && \
    unzip awscliv2.zip && \
    ./aws/install
# Copy the library from previous build step
COPY --from=sdk-builder /aws-kms-pkcs11/aws_kms_pkcs11.so /usr/lib64/pkcs11/
# Copy configuration files for aws-kms-pkcs11 library
COPY config.json x509.genkey openssl-pkcs11.conf /etc/aws-kms-pkcs11/
# Copy shell script to update config
COPY configure_pkcs.sh /bin/
