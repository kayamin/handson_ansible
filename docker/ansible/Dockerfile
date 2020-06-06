FROM centos/python-36-centos7:latest

USER root

ENV ANSIBLE_HOST_KEY_CHECKING False

RUN pip install --upgrade pip && \
    pip install ansible

# ssh でパスワード認証を用いる際に必要なライブラリ
RUN yum install -y sshpass

CMD /bin/bash