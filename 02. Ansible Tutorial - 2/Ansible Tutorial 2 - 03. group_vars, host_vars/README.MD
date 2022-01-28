## 03. group_vars, host_vars

#### (1) inventory 파일에 group_vars, host_vars 모두 작성
```bash
$  cat inventory

[STAGING_SERVERS_WEB]
node01  ansible_host=192.168.219.101    password=1234
node03  ansible_host=192.168.219.103    password=1234

[STAGING_SERVERS_WEB:vars]
ansible_user=choi
ansible_ssh_private_key_file=/home/choi/.ssh/ansible.pub

[PROD_SERVERS_WEB]
node02  ansible_host=192.168.219.102    password=1234
node04  ansible_host=192.168.219.104    password=1234

[PROD_SERVERS_WEB:vars]
ansible_user=choi
ansible_ssh_private_key_file=/home/choi/.ssh/ansible.pub

[STAGING_SERVERS_DB]
192.168.219.101
192.168.219.103

[PROD_SERVERS_DB]
192.168.219.102
192.168.219.104

[ALL_SERVERS_DB:children]
STAGING_SERVERS_DB
PROD_SERVERS_DB

[ALL_SERVERS_DB:vars]
db_endpoint=test.example.com:3306
owner=choi
location="Seoul,KR"
```

#### (1-1) ansible-inventory --list 명령으로 확인
```bash
$ ansible-inventory --list
{
    "ALL_SERVERS_DB": {
        "children": [
            "PROD_SERVERS_DB",
            "STAGING_SERVERS_DB"
        ]
    },
    "PROD_SERVERS_DB": {
        "hosts": [
            "192.168.219.102",
            "192.168.219.104"
        ]
    },
    "PROD_SERVERS_WEB": {
        "hosts": [
            "node02",
            "node04"
        ]
    },
    "STAGING_SERVERS_DB": {
        "hosts": [
            "192.168.219.101",
            "192.168.219.103"
        ]
    },
    "STAGING_SERVERS_WEB": {
        "hosts": [
            "node01",
            "node03"
        ]
    },
    "_meta": {
        "hostvars": {
            "192.168.219.101": {
                "db_endpoint": "test.example.com:3306",
                "location": "Seoul,KR",
                "owner": "choi"
            },
            "192.168.219.102": {
                "db_endpoint": "test.example.com:3306",
                "location": "Seoul,KR",
                "owner": "choi"
            },
            "192.168.219.103": {
                "db_endpoint": "test.example.com:3306",
                "location": "Seoul,KR",
                "owner": "choi"
            },
            "192.168.219.104": {
                "db_endpoint": "test.example.com:3306",
                "location": "Seoul,KR",
                "owner": "choi"
            },
            "node01": {
                "ansible_host": "192.168.219.101",
                "ansible_ssh_private_key_file": "/home/choi/.ssh/ansible.pub",
                "ansible_user": "choi",
                "password": 1234
            },
            "node02": {
                "ansible_host": "192.168.219.102",
                "ansible_ssh_private_key_file": "/home/choi/.ssh/ansible.pub",
                "ansible_user": "choi",
                "password": 1234
            },
            "node03": {
                "ansible_host": "192.168.219.103",
                "ansible_ssh_private_key_file": "/home/choi/.ssh/ansible.pub",
                "ansible_user": "choi",
                "password": 1234
            },
            "node04": {
                "ansible_host": "192.168.219.104",
                "ansible_ssh_private_key_file": "/home/choi/.ssh/ansible.pub",
                "ansible_user": "choi",
                "password": 1234
            }
        }
    },
    "all": {
        "children": [
            "ALL_SERVERS_DB",
            "PROD_SERVERS_WEB",
            "STAGING_SERVERS_WEB",
            "ungrouped"
        ]
    }
}
```

#### (2) group_vars, host_vars 폴더를 나누어서 따로 관리 가능
```bash
$ tree
.
├── ansible.cfg
├── group_vars
│   ├── ALL_SERVERS_DB
│   ├── PROD_SERVERS_WEB
│   └── STAGING_SERVERS_WEB
├── host_vars
│   ├── node01
│   ├── node02
│   ├── node03
│   └── node04
└── inventory

### group_vars
$ cat group_vars/STAGING_SERVERS_WEB
---
ansible_user                    : choi
ansible_ssh_private_key_file    : /home/choi/.ssh/ansible.pub

$ cat group_vars/PROD_SERVERS_WEB
---
ansible_user                    : choi
ansible_ssh_private_key_file    : /home/choi/.ssh/ansible.pub

$ cat group_vars/ALL_SERVERS_DB
---
db_endpoint     : test.example.com:3306
owner           : choi
location        : "Seoul,KR"

### host_vars
$ cat host_vars/node01
---
ansible_host    : 192.168.219.101
password        : 1234

$ cat host_vars/node02
---
ansible_host    : 192.168.219.102
password        : 1234

$ cat host_vars/node03
---
ansible_host    : 192.168.219.103
password        : 1234

$ cat host_vars/node04
---
ansible_host    : 192.168.219.104
password        : 1234

### inventory file
$ cat inventory
[STAGING_SERVERS_WEB]
node01
node03

[PROD_SERVERS_WEB]
node02
node04

[STAGING_SERVERS_DB]
192.168.219.101
192.168.219.103

[PROD_SERVERS_DB]
192.168.219.102
192.168.219.104

[ALL_SERVERS_DB:children]
STAGING_SERVERS_DB
PROD_SERVERS_DB
```

#### (2-1) ansible-inventory --list 명령으로 확인 

- (1-1)과 같은 결과로 나오는 것을 확인할 수 있다.

```bash
$ ansible-inventory --list
{
    "ALL_SERVERS_DB": {
        "children": [
            "PROD_SERVERS_DB",
            "STAGING_SERVERS_DB"
        ]
    },
    "PROD_SERVERS_DB": {
        "hosts": [
            "192.168.219.102",
            "192.168.219.104"
        ]
    },
    "PROD_SERVERS_WEB": {
        "hosts": [
            "node02",
            "node04"
        ]
    },
    "STAGING_SERVERS_DB": {
        "hosts": [
            "192.168.219.101",
            "192.168.219.103"
        ]
    },
    "STAGING_SERVERS_WEB": {
        "hosts": [
            "node01",
            "node03"
        ]
    },
    "_meta": {
        "hostvars": {
            "192.168.219.101": {
                "db_endpoint": "test.example.com:3306",
                "location": "Seoul,KR",
                "owner": "choi"
            },
            "192.168.219.102": {
                "db_endpoint": "test.example.com:3306",
                "location": "Seoul,KR",
                "owner": "choi"
            },
            "192.168.219.103": {
                "db_endpoint": "test.example.com:3306",
                "location": "Seoul,KR",
                "owner": "choi"
            },
            "192.168.219.104": {
                "db_endpoint": "test.example.com:3306",
                "location": "Seoul,KR",
                "owner": "choi"
            },
            "node01": {
                "ansible_host": "192.168.219.101",
                "ansible_ssh_private_key_file": "/home/choi/.ssh/ansible.pub",
                "ansible_user": "choi",
                "password": 1234
            },
            "node02": {
                "ansible_host": "192.168.219.102",
                "ansible_ssh_private_key_file": "/home/choi/.ssh/ansible.pub",
                "ansible_user": "choi",
                "password": 1234
            },
            "node03": {
                "ansible_host": "192.168.219.103",
                "ansible_ssh_private_key_file": "/home/choi/.ssh/ansible.pub",
                "ansible_user": "choi",
                "password": 1234
            },
            "node04": {
                "ansible_host": "192.168.219.104",
                "ansible_ssh_private_key_file": "/home/choi/.ssh/ansible.pub",
                "ansible_user": "choi",
                "password": 1234
            }
        }
    },
    "all": {
        "children": [
            "ALL_SERVERS_DB",
            "PROD_SERVERS_WEB",
            "STAGING_SERVERS_WEB",
            "ungrouped"
        ]
    }
}
```