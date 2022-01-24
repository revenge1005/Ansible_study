## 2. elevated ad-hoc Commands

- https://docs.ansible.com/ansible/latest/collections/ansible/builtin/apt_module.html

#### (1) apt update
- becom : sudo 권한을 실행
- ask-become-pass : pw
```
ansible all -m apt -a update_cache=true --become --ask-become-pass
```

#### (2) apt install 
```
ansible all -m apt -a name=vim-nox --become --ask-become-pass
```
```
ssh 192.168.219.101

tail /var/log/apt/history.log

Start-Date: 2022-01-25  00:36:56
Commandline: /usr/bin/apt-get -y -o Dpkg::Options::=--force-confdef -o Dpkg::Options::=--force-confold install vim-nox
Requested-By: choi (1000)
Install: javascript-common:amd64 (11, automatic), ruby2.7:amd64 (2.7.0-5ubuntu1.6, automatic), rake:amd64 (13.0.1-4, automatic), ruby-net-telnet:amd64 (0.1.1-2, automatic), ....
```

#### (3) apt install - lastest 
```
ansible all -m apt -a "name=snapd state=latest" --become --ask-become-pass
```

#### (4) apt dist-upgrade
```
ansible all -m apt -a "upgrade=dist" --become --ask-become-pass
```