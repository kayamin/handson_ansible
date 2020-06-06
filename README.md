
# Hands ON: Ansible x PostgreSQL

###  コンテナを立ち上げる
- ansible コンテナ
    - ansible コマンドをインストールし，実行するコンテナ
    - target コンテナに対して ansible コマンドを実行しDBをセットアップする
- target コンテナ
    - sshd を立ち上げ，ssh 接続可能にされたコンテナ
    - このコンテナに ansible コマンドを用いて postgreSQL を起動し DBを作成する

```
docker-compose -up d
```


### コンテナ間の接続を確認する

```
# ansible docker に入る
docker exec -it ansible /bin/bash

# ansible docker から target01 コンテナに ssh できることを確認
ssh target01

```

### ansible を用いて target01, 02 コンテナに posrgreSQL の DBをセットアップする

```
# ansible docker に入る
docker exec -it ansible /bin/bash

# target01,02 コンテナに対して ansible を実行する
ansible-playbook -i inventory.ini playbook.yml

# posrgresql がインストールされ，サービスとして起動されていることを確認
ssh target01
systemctl list-unit-files | grep postgresql

# posrgresql にログインするため postges ユーザーに切り替え
su postgres

# posrgresql にログイン (ユーザーは現在のシステムユーザーになる）
psql

# exampledb が作成されていることを確認 (ansible で作成した）
\l
```

### ansible コンテナから， target01 コンテナに立ち上げた db に接続する
- ansible でセットアップした db 接続情報
    - db: exampledb
    - user: jdoe
    - password: test

```
psql -h target01 -d exampledb -U jdoe
```


# Docker 設定詳細

- ansible コンテナ
    - centos x python イメージを使用
    - ansible をインストール

- target コンテナ
    - amazonlinux を使用
    - sshd 起動し，外部からssh接続を可能にする
        - openssh-server をインストール
        - root ユーザーに パスワード無しで接続できるようにする
        - ssh 接続のために 22 番port を開ける
    - systemd を実行可能にする
        - `/sbin/init` コマンドを実行する ( + docker-compose での privledge 設定も必要）

- docker-compose
    - ansible コンテナに, このレポジトリのディレクトリをマウント
        - ansible 関連の設定ファイルを配置するため
    - target コンテナに `priviledged: true` を設定
        - target コンテナ内で，systemd を用いた postgres の起動を可能にするため

# Ansible 詳細

- posrgresql セットアップのテンプレートとして[geerlingguy/ansible-role-postgresql](https://github.com/geerlingguy/ansible-role-postgresql) を使用
    - [Ansible-Galaxy](https://galaxy.ansible.com/) に登録されている ansible テンプレート
    - posrgresql のテンプレートの中で利用数が多いものを選択した

Ansible関連のファイル説明（必要なファイルのみ記述）

```
❯ tree ./ -l 3
./
├── README.md
├── ansible.cfg     # ansible コマンドの設定ファイル，roles の配置場所等を指定
├── ansible_log     # ansible コマンドのログファイル
│   └── log
├── inventory.ini   # ansible コマンドを実行する対象の host, ip 等の設定ファイル
├── group_vars      # ansible コマンド実行対象毎に ansible で設定する変数ファイルの置き場
│   └── target          # target コンテナに対して ansible コマンドを実行するときの変数
│       └── dbuser_password_postgres.yml
├── playbook.yml     # ansible-playbook コマンドで用いる，ansible の実行内容を設定するファイル
├── requirements.yml # ansible-galaxy コマンドで用いる，role をansible-galaxy から取得する際に用いる設定ファイル
└── roles            # playbook.yml で指定する role の内容を配置するディレクトリ
    └── geerlingguy.postgresql      # ansible-galaxy で提供されている postgresql をセットアップする role のテンプレート
        ├── LICENSE
        ├── README.md
        ├── defaults             # main.yml で読み込まれる変数を定義するディレクトリ
        │   └── main.yml
        ├─── vars                # main.yml で読み込まれる変数を定義するディレクトリ, defaults を上書きできる
        │   └── RedHat-2.yml        # amazonlinux でこのテンプレートを利用する際に使う定義ファイル，元々のこのファイルが定義されていなく amazonlinux で動かすために作成
        ├── handlers
        │   └── main.yml
        ├── meta
        │   └── main.yml
        ├── molecule
        │   └── default
        │       ├── converge.yml
        │       └── molecule.yml
        ├── tasks               # この role で実行する処理を定義するディレクトリ
        │   ├── configure.yml
        │   ├── databases.yml
        │   ├── initialize.yml
        │   ├── main.yml            # この role が playbook.yml で呼ばれたときに参照され，他の ymlの内容が読み込まれる
        │   ├── setup-Debian.yml
        │   ├── setup-RedHat.yml
        │   ├── users.yml
        │   └── variables.yml
        ├── templates           # templateモジュールでセットアップされるJinja2形式のテキストファイルを配置するディレクトリ
        │   ├── pg_hba.conf.j2      # postgresql 設定ファイルのテンプレート，このテンプレートに変数を代入したファイルが taget に配置される
        │   └── postgres.sh.j2      # postgresql 設定ファイルのテンプレート，このテンプレートに変数を代入したファイルが taget に配置される
        └── vars                # main.yml で読み込まれる変数を定義するディレクトリ
            └── RedHat-2.yml        # amazonlinux でこのテンプレートを利用する際に使う定義ファイル，元々のこのファイルが定義されていなく amazonlinux で動かすために作成
```




