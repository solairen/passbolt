FROM moleszek/ansible:13.2.0

RUN apt-get update && apt-get -y install git openssh-server openssh-client dos2unix vim

WORKDIR /ansible

COPY . /ansible

ENTRYPOINT [ "bash" ]
