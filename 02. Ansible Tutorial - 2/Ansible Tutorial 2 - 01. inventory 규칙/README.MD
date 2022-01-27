## 01. inventory 규칙

```bash

## > 호스트 변수: Playbook에서 사용될 호스트에 변수를 지정하는 것
## > host_vars 디렉토리를 생성하여 각각의 호스트에 대한 파일로 지정이 가능하다.
[staging_servers]
node01	ansible_host=192.168.219.101	ansible_user=choi	ansible_ssh_private_key_file=/home/choi/.ssh/ansible
node02	ansible_host=192.168.219.102	ansible_user=choi	ansible_ssh_private_key_file=/home/choi/.ssh/ansible

[prod_servers]
node03	ansible_host=192.168.219.103
node04	ansible_host=192.168.219.104

[prod_servers:vars]
ansible_user=choi
ansible_ssh_private_key_file=/home/choi/.ssh/ansible

## > 그룹 변수: 변수들을 하나의 그룹에 한번에 적용하는것도 가능
[atlanta]
node01
node02

[atlanta:vars]
ntp_server=ntp.atlanta.example.com
proxy=proxy.atlanta.example.com

## > 그룹의 그룹, 그리고 그룹 변수
## > ":children" 접미사를 사용하여 그룹의 하위 그룹을 만드는것도 가능
## > 또한, 이렇게 정의한 그룹에 ":vars"를 이용하여 변수를 지정해주는것 역시 가능하다.
[staging_DB]
192.168.219.101

[staging_WEB]
192.168.219.102

[staging_APP]
192.168.219.103
192.168.219.104

[staging_ALL:children]
staging_DB
staging_WEB
staging_APP

[staging_ALL:vars]
some_server=foo.southeast.example.com
halon_system_timeout=30
self_destruct_countdown=60
escape_pods=2

[prod_DB]
node01

[prod_WEB]
node02

[prod_APP]
node03
node04

[prod_ALL:children]
prod_DB
prod_WEB
prod_APP

[DB_ALL:children]
staging_DB
prod_DB

[APP_ALL:children]
staging_APP
prod_APP

[REGIONS:children]
DB_ALL
APP_ALL
```