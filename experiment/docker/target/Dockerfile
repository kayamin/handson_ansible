# Amazon EC2
FROM amazonlinux:latest

# yumで必要なライブラリをインストールする
RUN yum update -y && \
    # python3 をインストール， このdockerコンテナ内でansibleを python3 で走らせるために必要
    yum -y install python3 && \
    # OpenSSｈサーバをインストール
    yum -y install openssh-server && \
    # sudo コマンドをインストール
    yum install -y sudo

# sshd起動時に公開鍵が必要なため、作成
RUN ssh-keygen -f /etc/ssh/ssh_host_rsa_key -N '' -t rsa
RUN ssh-keygen -f /etc/ssh/ssh_host_ecdsa_key -N '' -t ecdsa
RUN ssh-keygen -f /etc/ssh/ssh_host_ed25519_key -N '' -t ed25519

# rootでログインできるようにする
RUN sed -ri 's/^#PermitEmptyPasswords no/PermitEmptyPasswords yes/' /etc/ssh/sshd_config

# rootのパスワードを指定 ansible実行時に指定するのが手間なので一旦なしに
RUN echo "root:" | chpasswd

# sshdを起動
RUN /bin/systemctl enable sshd


EXPOSE 22

CMD ["/sbin/init"]