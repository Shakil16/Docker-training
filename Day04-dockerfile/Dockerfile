FROM ubuntu:latest
WORKDIR /app
LABEL name="shakil1602"

ENV AWS_ACCESS_KEY_ID=AGSSHHSUUHHHH \
AWS_SECRET_KEY_ID=SJDHDHDHHDH \
AWS_DEFAULT_REGION=US-WEST-1

ARG T_VERSION='1.6.6' \
    P_VERSION='1.8.0'

RUN apt update && apt install -y jq net-tools curl wget unzip iputils-ping \
    && apt install -y nginx \
    && wget https://releases.hashicorp.com/terraform/${T_VERSION}/terraform_${T_VERSION}_linux_amd64.zip \
    && wget https://releases.hashicorp.com/packer/${P_VERSION}/packer_${P_VERSION}_linux_amd64.zip \
    && unzip terraform_${T_VERSION}_linux_amd64.zip  && unzip packer_${P_VERSION}_linux_amd64.zip \
    && chmod 777 terraform && chmod 777 packer \
    && ./terraform version && ./packer version 
CMD [ "nginx","-g","daemon off;"]