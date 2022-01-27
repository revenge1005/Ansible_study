## 02. ad-hoc 명령

- ansible의 ad-hoc 명령은 playbook을 작성하지 않고 command-line에서 직접 ansible 모듈을 호출해서 사용하는 방식

- ad-hoc 명려어 사용법
```bash
$ ansible host-pattern -m module [-a 'module options'] [-i inventory]
```

#### (1) ping module
```bash
$ ansible web_servers -m ping
192.168.219.104 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/libexec/platform-python"
    },
    "changed": false,
    "ping": "pong"
}
192.168.219.103 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/libexec/platform-python"
    },
    "changed": false,
    "ping": "pong"
}
```

#### (2) setup module

- ad-hoc에서 많이 사용하는 모듈중 하나로, ansible fact를 수집하는 setup 모듈이다.

- 실행하면 인벤토리에 있는 서버의 모든 fact를 수집해서 출력한다.

```bash
$ ansible db_servers -m setup | grep "ansible_distribution"
        "ansible_distribution": "Ubuntu",
        "ansible_distribution_file_parsed": true,
        "ansible_distribution_file_path": "/etc/os-release",
        "ansible_distribution_file_variety": "Debian",
        "ansible_distribution_major_version": "20",
        "ansible_distribution_release": "focal",
        "ansible_distribution_version": "20.04",

ansible db_servers -m setup | grep "ansible_user"
        "ansible_user_dir": "/home/choi",
        "ansible_user_gecos": "choi",
        "ansible_user_gid": 1000,
        "ansible_user_id": "choi",
        "ansible_user_shell": "/bin/bash",
        "ansible_user_uid": 1000,
        "ansible_userspace_architecture": "x86_64",
        "ansible_userspace_bits": "64",
```

#### (3) shell module

```bash
$ ansible all -m shell -a "uptime"
192.168.219.103 | CHANGED | rc=0 >>
 00:51:13 up  2:01,  1 user,  load average: 0.00, 0.00, 0.00
192.168.219.104 | CHANGED | rc=0 >>
 00:51:12 up  2:01,  1 user,  load average: 0.00, 0.00, 0.00
192.168.219.101 | CHANGED | rc=0 >>
 00:51:12 up  2:17,  1 user,  load average: 0.00, 0.00, 0.00
192.168.219.102 | CHANGED | rc=0 >>
 00:51:12 up  2:17,  1 user,  load average: 0.00, 0.00, 0.00

ansible all -m shell -a "ls -al .ssh"
192.168.219.102 | CHANGED | rc=0 >>
total 12
drwx------ 2 choi choi 4096 Jan 27 22:47 .
drwxr-xr-x 5 choi choi 4096 Jan 27 22:53 ..
-rw------- 1 choi choi  183 Jan 27 22:48 authorized_keys
192.168.219.101 | CHANGED | rc=0 >>
total 12
drwx------ 2 choi choi 4096 Jan 27 22:47 .
drwxr-xr-x 5 choi choi 4096 Jan 27 22:53 ..
-rw------- 1 choi choi  183 Jan 27 22:47 authorized_keys
192.168.219.103 | CHANGED | rc=0 >>
total 4
drwx------  2 choi choi  29 Jan 27 22:47 .
drwx------. 4 choi choi 111 Jan 27 22:53 ..
-rw-------  1 choi choi 183 Jan 27 22:48 authorized_keys
192.168.219.104 | CHANGED | rc=0 >>
total 4
drwx------  2 choi choi  29 Jan 27 22:47 .
drwx------. 4 choi choi 111 Jan 27 22:53 ..
-rw-------  1 choi choi 183 Jan 27 22:48 authorized_keys
```

#### (4) copy module
```bash
$ ansible all -m copy -a "src=inventory.bak dest=/home/choi/ mode=0777"
192.168.219.101 | CHANGED => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "changed": true,
    "checksum": "5dffa510ae6affeaea8a6366df8f748b5887bf17",
    "dest": "/home/choi/inventory.bak",
    "gid": 1000,
    "group": "choi",
    "md5sum": "1553fcb0aed9bc581e91566267f50531",
    "mode": "0777",
    "owner": "choi",
    "size": 1577,
    "src": "/home/choi/.ansible/tmp/ansible-tmp-1643299156.8577902-10900-145368955608159/source",
    "state": "file",
    "uid": 1000
}

...

$ ansible db_servers -m shell -a "ls -l /home/choi"
192.168.219.101 | CHANGED | rc=0 >>
total 4
-rwxrwxrwx 1 choi choi 1577 Jan 28 00:59 inventory.bak
```

#### (5) file module
```bash
$ ansible all -m file -a "path=/home/choi/inventory.bak state=absent"
192.168.219.103 | CHANGED => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/libexec/platform-python"
    },
    "changed": true,
    "path": "/home/choi/inventory.bak",
    "state": "absent"
}

...

$ ansible all -m shell -a "ls -l /home/choi"
192.168.219.104 | CHANGED | rc=0 >>
total 0
192.168.219.103 | CHANGED | rc=0 >>
total 0
192.168.219.101 | CHANGED | rc=0 >>
total 0
192.168.219.102 | CHANGED | rc=0 >>
total 0
```

#### (6) get_url module
```bash
$ ansible all -m get_url -a "url=https://download.java.net/openjdk/jdk8u41/ri/openjdk-8u41-b04-linux-x64-14_jan_2020.tar.gz dest=/home/choi"
192.168.219.102 | CHANGED => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "changed": true,
    "checksum_dest": null,
    "checksum_src": "3bf29d9bc379f18dab24358e8bd97c4340ce3f1a",
    "dest": "/home/choi/openjdk-8u41-b04-linux-x64-14_jan_2020.tar.gz",
    "elapsed": 6,
    "gid": 1000,
    "group": "choi",
    "md5sum": "35f515e9436f4fefad091db2c1450c5f",
    "mode": "0664",
    "msg": "OK (174287938 bytes)",
    "owner": "choi",
    "size": 174287938,
    "src": "/home/choi/.ansible/tmp/ansible-tmp-1643300466.165444-11834-5022500933289/tmpyct9v784",
    "state": "file",
    "status_code": 200,
    "uid": 1000,
    "url": "https://download.java.net/openjdk/jdk8u41/ri/openjdk-8u41-b04-linux-x64-14_jan_2020.tar.gz"
}

...

ansible all -m shell -a "ls -l /home/choi"
192.168.219.102 | CHANGED | rc=0 >>
total 170204
-rw-rw-r-- 1 choi choi 174287938 Jan 28 01:21 openjdk-8u41-b04-linux-x64-14_jan_2020.tar.gz
```

#### (7) yum module

```bash
$ ansible all -m yum -a "name=httpd state=installed" --ask-become-pass -b

## > Ubuntu 환경일 경우, Error -> module을 apt로 변경하면 설치됨
192.168.219.101 | FAILED! => {
    "ansible_facts": {
        "pkg_mgr": "apt"
    },
    "changed": false,
    "msg": [
        "Could not detect which major revision of yum is in use, which is required to determine module backend.",
        "You should manually specify use_backend to tell the module whether to use the yum (yum3) or dnf (yum4) backend})"
    ]
}

## > CentOS 환경일 경우, 설치되는 것을 확인
192.168.219.104 | CHANGED => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/libexec/platform-python"
    },
    "changed": true,
    "msg": "",
    "rc": 0,
    "results": [
        "Installed: httpd-filesystem-2.4.37-43.module_el8.5.0+1022+b541f3b1.noarch",
        "Installed: apr-util-1.6.1-6.el8.x86_64",
        "Installed: apr-util-bdb-1.6.1-6.el8.x86_64",
        "Installed: httpd-tools-2.4.37-43.module_el8.5.0+1022+b541f3b1.x86_64",
        "Installed: mod_http2-1.15.7-3.module_el8.4.0+778+c970deab.x86_64",
        "Installed: apr-util-openssl-1.6.1-6.el8.x86_64",
        "Installed: centos-logos-httpd-85.8-2.el8.noarch",
        "Installed: httpd-2.4.37-43.module_el8.5.0+1022+b541f3b1.x86_64",
        "Installed: apr-1.6.3-12.el8.x86_64"
    ]
}

## > 최신 버전 설치
$ ansible all -m yum -a "name=httpd state=latest" --ask-become-pass -b

## > 삭제
$ ansible all -m yum -a "name=httpd state=removed" --ask-become-pass -b
```

#### (8) uri module
```bash
$ ansible all -m uri -a "url=http://192.168.219.100"
192.168.219.103 | SUCCESS => {
    "accept_ranges": "bytes",
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/libexec/platform-python"
    },
    "changed": false,
    "connection": "close",
    "content_length": "10918",
    "content_type": "text/html",
    "cookies": {},
    "cookies_string": "",
    "date": "Thu, 27 Jan 2022 16:30:47 GMT",
    "elapsed": 0,
    "etag": "\"2aa6-5d692cdbb0495\"",
    "last_modified": "Thu, 27 Jan 2022 16:28:08 GMT",
    "msg": "OK (10918 bytes)",
    "redirected": false,
    "server": "Apache/2.4.41 (Ubuntu)",
    "status": 200,
    "url": "http://192.168.219.100",
    "vary": "Accept-Encoding"
}
```

#### (9) service module
```bash
$ ansible all -m yum -a "name=httpd state=latest" --ask-become-pass -b

$ ansible all -m service -a "name=http state=started enabled=yes" --ask-become-pass -b
192.168.219.104 | CHANGED => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/libexec/platform-python"
    },
    "changed": true,
    "enabled": true,
    "name": "httpd",
    "state": "started",
    ......
    ......
    ......
```