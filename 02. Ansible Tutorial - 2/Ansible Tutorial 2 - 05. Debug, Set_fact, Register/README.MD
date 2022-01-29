## 5. Debug, Set_fact, Register

- Debug : task 단위로 debug 출력이 가능하며, 변수 출력을 {{}} 할 수 있다.

- Set_fact : ansible 변수 설정, 변수를 ansible_facts안에 저장한다.

-> ansible_facts는 Ansible에서 각 노드에 맞게 동적으로 할당되는 변수들을 말하며, 노드들의 시스템 정보등이 들어가 있다.

- Register : Playbook에서의 결과를 변수로 정하기 위해 사용되는 모듈

#### (1) 테스트 구성

```bash
$ tree .
.
├── ansible.cfg
├── group_vars
│   └── PROD_SERVERS_WEB
├── inventory
└── playbook.yml

1 directory, 4 files

$ cat ansible.cfg
[defaults]
inventory = inventory

$ cat group_vars/PROD_SERVERS_WEB
---
ansible_user                    : choi
ansible_ssh_private_key_file    : /home/choi/.ssh/ansible

$ cat inventory
[PROD_SERVERS_WEB]
node03 ansible_host=192.168.219.103 owner=choi
node04 ansible_host=192.168.219.104 owner=choi
```

#### (2) playbook 작성 
```bash
$ cat playbook.yml
---
- name: My Super Puper Playbook for Variables Lesson
  hosts: all
  become: yes

  vars:
    message1: Privet
    message2: World
    secret: XCVBXHXIJ0IJ0IJ0IDJ0IDJ0JD

  tasks:
  - name: Print Secret variable
    debug:
      var: secret

  - debug:
      msg: "Sekretnoe slovo: {{ secret }}"

  - debug:
      msg: "Vladelec Etogo Servera -->{{ owner }}<--"

  - set_fact: full_message="{{ message1 }} {{ message2 }} from {{ owner }}"

  - debug:
      var: full_message

  - debug:
      var: ansible_distribution

  - shell: uptime
    register: results

  - debug:
      var: results
```

#### (3) 결과 확인
```bash
$ ansible-playbook playbook.yml --ask-become-pass
BECOME password:

PLAY [My Super Puper Playbook for Variables Lesson] *****************************************************************************************************************************************************

TASK [Gathering Facts] **********************************************************************************************************************************************************************************
ok: [node04]
ok: [node03]

TASK [Print Secret variable] ****************************************************************************************************************************************************************************
ok: [node03] => {
    "secret": "XCVBXHXIJ0IJ0IJ0IDJ0IDJ0JD"
}
ok: [node04] => {
    "secret": "XCVBXHXIJ0IJ0IJ0IDJ0IDJ0JD"
}

TASK [debug] ********************************************************************************************************************************************************************************************
ok: [node03] => {
    "msg": "Sekretnoe slovo: XCVBXHXIJ0IJ0IJ0IDJ0IDJ0JD"
}
ok: [node04] => {
    "msg": "Sekretnoe slovo: XCVBXHXIJ0IJ0IJ0IDJ0IDJ0JD"
}

TASK [debug] ********************************************************************************************************************************************************************************************
ok: [node03] => {
    "msg": "Vladelec Etogo Servera -->choi<--"
}
ok: [node04] => {
    "msg": "Vladelec Etogo Servera -->choi<--"
}

TASK [set_fact] *****************************************************************************************************************************************************************************************
ok: [node03]
ok: [node04]

TASK [debug] ********************************************************************************************************************************************************************************************
ok: [node03] => {
    "full_message": "Privet World from choi"
}
ok: [node04] => {
    "full_message": "Privet World from choi"
}

TASK [debug] ********************************************************************************************************************************************************************************************
ok: [node03] => {
    "ansible_distribution": "CentOS"
}
ok: [node04] => {
    "ansible_distribution": "CentOS"
}

TASK [shell] ********************************************************************************************************************************************************************************************
changed: [node04]
changed: [node03]

TASK [debug] ********************************************************************************************************************************************************************************************
ok: [node03] => {
    "results": {
        "changed": true,
        "cmd": "uptime",
        "delta": "0:00:00.005615",
        "end": "2022-01-29 22:03:27.545646",
        "failed": false,
        "msg": "",
        "rc": 0,
        "start": "2022-01-29 22:03:27.540031",
        "stderr": "",
        "stderr_lines": [],
        "stdout": " 22:03:27 up  1:46,  1 user,  load average: 0.12, 0.03, 0.01",
        "stdout_lines": [
            " 22:03:27 up  1:46,  1 user,  load average: 0.12, 0.03, 0.01"
        ]
    }
}
ok: [node04] => {
    "results": {
        "changed": true,
        "cmd": "uptime",
        "delta": "0:00:00.005540",
        "end": "2022-01-29 22:03:26.698548",
        "failed": false,
        "msg": "",
        "rc": 0,
        "start": "2022-01-29 22:03:26.693008",
        "stderr": "",
        "stderr_lines": [],
        "stdout": " 22:03:26 up  1:45,  1 user,  load average: 0.00, 0.00, 0.00",
        "stdout_lines": [
            " 22:03:26 up  1:45,  1 user,  load average: 0.00, 0.00, 0.00"
        ]
    }
}

PLAY RECAP **********************************************************************************************************************************************************************************************
node03                     : ok=9    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
node04                     : ok=9    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```