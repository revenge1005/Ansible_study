## 6. Handler와 When

#### 1) When

- 조건문, 조건이 참(True)일 경우 Task 실행, 거짓(False)일 경우 Task 미실행

#### 2) Handler

- 조건부 형식 중에 하나 (조건부 형식이란, if 앞의 테스크가 동작하거나 또는 조건에 맞는다면 handler를 실행한다)

- Task에서 notify를 이용하여 handler의 name을 매개변수로 전달하여 알린다.

- **모든 Task가 모두 실행된 후에 Handler가 실행**된다.

- 여러 차레 알람을 받아도 **핸들러는 한 번만 실행**된다.

- 플레이에 여러 handler를 포함하면, 해당 핸들러는 알림 순서가 아니라, **handler 섹션에 정의된 순서대로 항상 실행**된다.

----

#### 3) Handler 예제 - web server install 

```bash
$ tree
.
├── ansible.cfg
├── files
│   └── index.html
├── group_vars
│   └── ALL_LINUX
├── inventory
└── playbook.yml

2 directories, 5 files
```

```bash
$ cat playbook.yml
---
- name: Install Apache and Upload my Home Page
  hosts: all
  become: yes

  vars:
    source_file: ./files/index.html
    destin_file: /var/www/html/

  tasks:

  - name: Install Apache WEB Server
    yum:
      name: httpd
      state: latest

  - name: Copy HomePage file to WEB Server
    copy:
      src: "{{ source_file  }}"
      dest: "{{ destin_file  }}"
      mode: 0555
    notify: Restart Apache

  - name: Start WEB Server service
    service:
      name: httpd
      state: started
      enabled: yes

  handlers:
  - name: Restart Apache
    service:
      name: httpd
      state: restarted
```

![1321321](https://user-images.githubusercontent.com/42735894/152640925-c446e60a-0792-489f-8c64-6701bd2a5624.PNG)

----

#### 4) When 예제 

```bash
$ tree
.
├── ansible.cfg
├── files
│   └── index.html
├── group_vars
│   └── ALL_LINUX
├── inventory
└── playbook.yml

2 directories, 5 files
```

```bash
$ cat inventory
[ALL_LINUX]
node01 ansible_host=192.168.219.101
node02 ansible_host=192.168.219.102
node03 ansible_host=192.168.219.103
node04 ansible_host=192.168.219.104

$ cat playbook.yml
---
- name: Install Apache and Upload my Home Page
  hosts: all
  become: yes

  vars:
    source_file: ./files/index.html
    destin_file: /var/www/html/

  tasks:

  - name: Install Aapache WEB Server for RedHat
    yum:
      name: httpd
      state: latest
    when: ansible_os_family == "RedHat"

  - name: Install Aapache WEB Server for Debian
    apt:
      name: apache2
      state: latest
    when: ansible_os_family == "Debian"

  - name: Copy HomePage file to WEB Server
    copy:
      src: "{{ source_file  }}"
      dest: "{{ destin_file  }}"
      mode: 0555
#   notify: Restart Apache

  - name: Start WEB Server service for RedHat
    service:
      name: httpd
      state: started
      enabled: yes
    when: ansible_os_family == "RedHat"

  - name: Start WEB Server service for Debian
    service:
      name: apache2
      state: started
      enabled: yes
    when: ansible_os_family == "Debian"

  handlers:
  - name: Restart Apache RedHat
    service:
      name: httpd
      state: restarted

  - name: Restart Apache Debian
    service:
      name: apache2
      state: restarted
```

![dawqeqw](https://user-images.githubusercontent.com/42735894/152642068-256d7f8e-2530-4c80-8054-6d4c394773f5.PNG)