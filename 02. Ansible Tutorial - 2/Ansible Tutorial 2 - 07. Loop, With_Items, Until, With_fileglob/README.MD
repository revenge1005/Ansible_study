## 6. Loop, With_Items, Until, With_fileglob

#### 1) 반복문

- Ansible 2.4 까지는 with_* 키워드를 사용하여 작업 하였는데, 2.5 부터는 loop 키워드를 사용한다.

- with_* 키워드에 대한 여러가지 반복문을 완전히 대체하는 것은 아니지만, 거의 대부분 loop에서도 유효하다.

- **패키지 관련된 모듈은 반복문을 쓰지 않을것을 권장**한다.

#### <참고>

- https://docs.ansible.com/ansible/latest/user_guide/playbooks_loops.html

- https://moonstrike.github.io/ansible/2016/10/17/Ansible-Loops.html

----

#### 2) with_items, loop 예제

```bash
$ cat playbook_loop.yml
---
- name: Loops Playbook
  hosts: node03
  become: yes

  tasks:

  - name: Say Hello to ALL
    debug:
      msg: "Hello {{ item }}"
    with_items:
        - "Vasya"
        - "Petya"
        - "Masha"
        - "Olya"
```

```bash
$ cat playbook_loop.yml
---
- name: Loops Playbook
  hosts: node03
  become: yes

  tasks:

  - name: Say Hello to ALL
    debug:
      msg: "Hello {{ item }}"
    loop:
        - "Vasya"
        - "Petya"
        - "Masha"
        - "Olya"
```

- 결과는 같음

```bash
$ ansible-playbook --ask-become-pass playbook_loop.yml
BECOME password:

PLAY [Loops Playbook] ********************************************************************************************

TASK [Gathering Facts] *******************************************************************************************
ok: [node03]

TASK [Say Hello to ALL] ******************************************************************************************
ok: [node03] => (item=Vasya) => {
    "msg": "Hello Vasya"
}
ok: [node03] => (item=Petya) => {
    "msg": "Hello Petya"
}
ok: [node03] => (item=Masha) => {
    "msg": "Hello Masha"
}
ok: [node03] => (item=Olya) => {
    "msg": "Hello Olya"
}

PLAY RECAP *******************************************************************************************************
node03                     : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```

----

#### 3) until 예제

```bash
$ cat playbook_until.yml
---
- name: Loops Playbook2
  hosts: node03
  become: yes

  tasks:
  - name: Loop Until example
    shell: echo -n Z >> myfile.txt && cat myfile.txt
    register: output
    delay: 2    # 매 실행 시마다 delay
    retries: 10 # 반복 횟수
    until: output.stdout.find("ZZZZ") == false # exit 조건

  - name: Print Final Output
    debug:
      var: output.stdout
```

```bash
$ ansible-playbook --ask-become-pass playbook_until.yml
BECOME password:

PLAY [Loops Playbook2] ****************************************************************************

TASK [Gathering Facts] ****************************************************************************
ok: [node03]

TASK [Loop Until example] *************************************************************************
FAILED - RETRYING: [node03]: Loop Until example (10 retries left).
FAILED - RETRYING: [node03]: Loop Until example (9 retries left).
FAILED - RETRYING: [node03]: Loop Until example (8 retries left).
changed: [node03]

TASK [Print Final Output] *************************************************************************
ok: [node03] => {
    "output.stdout": "ZZZZ"
}

PLAY RECAP ****************************************************************************************
node03                     : ok=3    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```

----

#### 4) loop 예제

```bash
$ $ tree
.
├── ansible.cfg
├── inventory
├── MyWebSite
│   └── index.html
├── MyWebSite2
│   ├── assets
│   │   └── favicon.ico
│   ├── css
│   │   └── styles.css
│   ├── index.html
│   └── js
│       └── scripts.js
├── playbook_loop.yml
├── playbook_until.yml
└── playbook.yml
```

```bash
$ cat playbook.yml
---
- name: Install Apache and Upload my Home Page
  hosts: all
  become: yes

  vars:
    source_folder: ./MyWebSite2
    destin_folder: /var/www/html

  tasks:

  - name: Check and Print Linux Version
    debug:
      var: ansible_os_family

  - block:   # ======Block for RedHat======

      - name: Install Apache WEB Server for RedHat
        yum:
          name: httpd
          state: latest

      - name: Start WEB Server service for RedHat
        service:
          name: httpd
          state: started
          enabled: yes

    when: ansible_os_family == "RedHat"

  - block:   # ======Block for Debian======

      - name: Install Apache WEB Server for Debian
        apt:
          name: apache2
          state: latest

      - name: Start WEB Server service for Debian
        service:
          name: apache2
          state: started
          enabled: yes

    when: ansible_os_family == "Debian"

  - name: Copy HomePage file to WEB Server
    copy:
      src: "{{ source_folder  }}/{{ item }}"
      dest: "{{ destin_folder }}"
      mode: 0555
    loop:
      - "index.html"
      - "assets"
      - "css"
    notify:
      - Restart Apache RedHat
      - Restart Apache Debian

  handlers:
  - name: Restart Apache RedHat
    service:
      name: httpd
      state: restarted
    when: ansible_os_family == "RedHat"

  - name: Restart Apache Debian
    service:
      name: apache2
      state: restarted
    when: ansible_os_family == "Debian"
```

----

#### 5) With_fileglob 예제

```bash
$ $ tree
.
├── ansible.cfg
├── inventory
├── MyWebSite
│   └── index.html
├── MyWebSite2
│   ├── assets
│   │   └── favicon.ico
│   ├── css
│   │   └── styles.css
│   ├── index.html
│   └── js
│       └── scripts.js
├── playbook_loop.yml
├── playbook_until.yml
├── playbook.yml
└── playbook2.yml
```

```bash
$ cat playbook2.yml
---
- name: Install Apache and Upload my Home Page
  hosts: all
  become: yes

  vars:
    source_folder: ./MyWebSite2
    destin_folder: /var/www/html

  tasks:

  - name: Check and Print Linux Version
    debug:
      var: ansible_os_family

  - block:   # ======Block for RedHat======

      - name: Install Apache WEB Server for RedHat
        yum:
          name: httpd
          state: latest

      - name: Start WEB Server service for RedHat
        service:
          name: httpd
          state: started
          enabled: yes

    when: ansible_os_family == "RedHat"

  - block:   # ======Block for Debian======

      - name: Install Apache WEB Server for Debian
        apt:
          name: apache2
          state: latest

      - name: Start WEB Server service for Debian
        service:
          name: apache2
          state: started
          enabled: yes

    when: ansible_os_family == "Debian"

  - name: Copy HomePage file to WEB Server
    copy:
      src: "{{ item }}"
      dest: "{{ destin_folder }}"
      mode: 0555
    with_fileglob: "{{ source_folder  }}/*"
#   loop:
#     - "index.html"
#     - "assets"
#     - "css"
    notify:
      - Restart Apache RedHat
      - Restart Apache Debian

  handlers:
  - name: Restart Apache RedHat
    service:
      name: httpd
      state: restarted
    when: ansible_os_family == "RedHat"

  - name: Restart Apache Debian
    service:
      name: apache2
      state: restarted
    when: ansible_os_family == "Debian"
```