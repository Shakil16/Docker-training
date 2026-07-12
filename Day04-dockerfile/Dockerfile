FROM ubuntu:latest

LABEL maintainer="SHAKIL"

ARG T_VERSION=1.6.6
ARG P_VERSION=1.8.0

ENV AWS_DEFAULT_REGION=us-east-1

RUN apt-get update && \
    apt-get install -y --no-install-recommends \
        curl \
        wget \
        unzip \
        jq \
        net-tools \
        nginx \
        iputils-ping && \
    rm -rf /var/lib/apt/lists/*

RUN wget -q https://releases.hashicorp.com/terraform/${T_VERSION}/terraform_${T_VERSION}_linux_amd64.zip && \
    wget -q https://releases.hashicorp.com/packer/${P_VERSION}/packer_${P_VERSION}_linux_amd64.zip && \
    unzip terraform_${T_VERSION}_linux_amd64.zip && \
    unzip packer_${P_VERSION}_linux_amd64.zip && \
    mv terraform /usr/local/bin/ && \
    mv packer /usr/local/bin/ && \
    chmod +x /usr/local/bin/terraform /usr/local/bin/packer && \
    terraform version && \
    packer version && \
    rm -f *.zip

EXPOSE 80

CMD ["nginx", "-g", "daemon off;"]
```