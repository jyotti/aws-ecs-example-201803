# AWS ECS Example - 201803

ecs-cliをつかったサンプル。

## Installation

### Prerequisites

- [VirtualBox](https://www.virtualbox.org/wiki/Downloads)
- [Docker](https://www.docker.com/community-edition#/download)
- [ecs-cli](https://github.com/aws/amazon-ecs-cli)

Homebrew でも可能

```bash
brew cask install virtualbox
brew cask install docker
brew install amazon-ecs-cli
```

Show version

```bash
docker --version
Docker version 18.03.0-ce, build 0520e24
ecs-cli --version
ecs-cli version 1.3.0 (*UNKNOWN)
VBoxManage --version
5.2.8r121009
```

### Setup Docker Machine

Create docker-machine `ecs-demo`

```bash
docker-machine create ecs-demo
docker-machine ls
```

machine status

```bash
docker-machine ls
```

IPAddress => `192.168.99.101`

```
$ docker-machine ls
NAME       ACTIVE   DRIVER       STATE     URL                         SWARM   DOCKER        ERRORS
default    *        virtualbox   Running   tcp://192.168.99.100:2376           v18.03.0-ce
ecs-demo   -        virtualbox   Running   tcp://192.168.99.101:2376           v18.03.0-ce
```

`DOCKER_HOST`を確認

```bash
docker-machine ip ecs-demo
192.168.99.101
docker-machine env ecs-demo
export DOCKER_TLS_VERIFY="1"
export DOCKER_HOST="tcp://192.168.99.101:2376"
export DOCKER_CERT_PATH="/Users/jyotti/.docker/machine/machines/ecs-demo"
export DOCKER_MACHINE_NAME="ecs-demo"
# Run this command to configure your shell:
# eval $(docker-machine env ecs-demo)
```

`eval`で環境変数を設定

```bash
eval $(docker-machine env ecs-demo)
```

`DOCKER_HOST`などが反映されている

```bash
env | grep -i docker
DOCKER_MACHINE_NAME=ecs-demo
DOCKER_CERT_PATH=/Users/jyotti/.docker/machine/machines/ecs-demo
DOCKER_TLS_VERIFY=1
DOCKER_HOST=tcp://192.168.99.101:2376
```

## Demo

Sample - https://docs.docker.com/compose/wordpress/

docker-compose構成
- WordPress
- MySQL5.7

注意点

- 2018-03時点でecs-cliはdocker-composeのバージョン2まで対応

docker-compose

- docker-compose.yml: ローカル用
- docker-compose-ecs.yml: AWS ECS 用

### Local

docker-compose up

- ファイル名が`docker-compose.yml`の場合、`-f`はなくても良い

```bash
docker-compose -f docker-compose.yml up
```

Open Browser http://{your_docker_host_ip}:8000/

### AWS ECS

ecs-cliでクラスタ構築から実行まで。

#### ecs-cliの初期設定

クラスタ名 `ecs-demo` でセットアップ。

```bash
ecs-cli configure \
  --cluster ecs-demo \
  --region us-east-1 \
  --default-launch-type EC2 \
  --config-name ecs-demo
```

#### クラスタ立ち上げ

CloudFormationでVPCからクラスタまで作られる。

- t2.medium * 2
- EC2でKeyPair `ecs_demo` を作成しておく

```bash
ecs-cli up \
  --keypair ecs_demo \
  --capability-iam \
  --size 2 \
  --instance-type t2.medium \
  --cluster-config ecs-demo
```

#### Taskの作成と実行

- Taskが未作成の場合は新規に作成して実行
- Taskが既に作成済みの場合、新たなリビジョンが追加されて実行

```bash
ecs-cli compose \
  --file docker-compose-ecs.yml \
  --cluster ecs-demo \
  --project-name wordpress \
  --cluster-config ecs-demo \
  up --create-log-groups
```

#### クラスタ削除

```bash
ecs-cli down \
  --cluster-config ecs-demo \
  --force
```
