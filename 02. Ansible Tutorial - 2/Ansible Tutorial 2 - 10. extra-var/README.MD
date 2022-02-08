## 10. extra-var

#### 1) Command Line로 변수 전달 

- "--extra-var" == "--extra-vars" == "-e"

- 해당 옵션을 이용하여 실행시간에 CLI로 변수 값을 전달이 가능하다.

- **단, 변수의 type을 살리고 싶으면 반드시 JSON 형식으로 전달할 것, 기본 key=valus 형식의 모든 value가 String으로 인식**된다.

- << 예시 - 1 >>

```bash
ansible-playbook --ask-become-pass playbook.yml --extra-vars "hosts=vipers user=starbuck"

# 또는 JSON 형식으로
ansible-playbook --ask-become-pass playbook.yml --extra-vars \
    '{"hosts: "vipers", "user=starbuck", "ghosts":["inky","pinky","clyde","sue"], release: 1}'
```

- **@를 사용하여 JSON 파일로부터 바로 읽을 수도 있다.**

- << 예시 - 2 >>

```bash
ansible-playbook --ask-become-pass playbook.yml --extra-var "@some_file.json"
```

<br>

---

#### 2) 예제

```bash
$ tree
.
├── ansible.cfg
├── inventory
├── playbook.yml
└── roles
    └── web_servers
        ├── defaults
        │   └── main.yml
        ├── files
        │   └── MyWebSite
        │       ├── 1.png
        │       ├── 2.png
        │       ├── 3.png
        │       └── 4.png
        ├── handlers
        │   └── main.yml
        ├── meta
        │   └── main.yml
        ├── tasks
        │   └── main.yml
        ├── templates
        │   └── index.j2
        ├── tests
        │   ├── inventory
        │   └── test.yml
        └── vars
            └── main.yml

11 directories, 15 files

# inventory 파일 
$ cat inventory
[UBUNTU]
node01 ansible_host=192.168.219.101
node02 ansible_host=192.168.219.102

[ROCKEY]
node03 ansible_host=192.168.219.103
node04 ansible_host=192.168.219.104

[All_LINUX:children]
UBUNTU
ROCKEY

# playbook 파일 
$ cat playbook.yml
---
- name: Install Apache and Upload my Home Page
  hosts: "{{ MYHOSTS }}"
  become: true

  roles:
    - web_servers
```

#### << 결과 >>

```bash
$ ansible-playbook playbook.yml --ask-become-pass --extra-vars "MYHOSTS=ROCKEY"
BECOME password:

PLAY [Install Apache and Upload my Home Page] ************************************************************

TASK [Gathering Facts] ***********************************************************************************
ok: [node04]
ok: [node03]

TASK [web_servers : Check and Print Linux Version] *******************************************************
ok: [node03] => {
    "ansible_os_family": "RedHat"
}
ok: [node04] => {
    "ansible_os_family": "RedHat"
}

TASK [web_servers : install Httpd package for Rocky Linux] ***********************************************
changed: [node04]
changed: [node03]

TASK [web_servers : Start WEB Server service for Rocky Linux] ********************************************
changed: [node04]
changed: [node03]

TASK [web_servers : Install Apache WEB Server for Debian] ************************************************
skipping: [node03]
skipping: [node04]

TASK [web_servers : Start WEB Server service for Debian] *************************************************
skipping: [node03]
skipping: [node04]

TASK [web_servers : Generate index.html file] ************************************************************
changed: [node03]
changed: [node04]

TASK [web_servers : Copy Homepage file to WEB Server] ****************************************************
changed: [node03] => (item=/home/choi/ansible/roles/web_servers/files/./MyWebSite/2.png)
changed: [node04] => (item=/home/choi/ansible/roles/web_servers/files/./MyWebSite/2.png)
changed: [node03] => (item=/home/choi/ansible/roles/web_servers/files/./MyWebSite/3.png)
changed: [node04] => (item=/home/choi/ansible/roles/web_servers/files/./MyWebSite/3.png)
changed: [node03] => (item=/home/choi/ansible/roles/web_servers/files/./MyWebSite/1.png)
changed: [node04] => (item=/home/choi/ansible/roles/web_servers/files/./MyWebSite/1.png)
changed: [node03] => (item=/home/choi/ansible/roles/web_servers/files/./MyWebSite/4.png)
changed: [node04] => (item=/home/choi/ansible/roles/web_servers/files/./MyWebSite/4.png)

RUNNING HANDLER [web_servers : Restart Apache RedHat] ****************************************************
changed: [node03]
changed: [node04]

RUNNING HANDLER [web_servers : Restart Apache Debian] ****************************************************
skipping: [node03]
skipping: [node04]

PLAY RECAP ***********************************************************************************************
node03                     : ok=7    changed=5    unreachable=0    failed=0    skipped=3    rescued=0    ignored=0
node04                     : ok=7    changed=5    unreachable=0    failed=0    skipped=3    rescued=0    ignored=0
```