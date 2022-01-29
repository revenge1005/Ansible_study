## 04. Playbook 작성

#### (1) 테스트 구성

```bash
$ tree
.
├── ansible.cfg
├── files
│   └── default_site.html
├── group_vars
│   └── PROD_SERVERS_WEB
└── inventory

2 directories, 4 files
```

```bash
$ cat ansible.cfg
[defaults]
inventory = inventory

$ cat files/default_site.html
<html>
  <title>Web-site Test</title>
  <body>
          <p> Ansible is awesome!</p>
  </body>
</html>

$ cat group_vars/PROD_SERVERS_WEB
---
ansible_user                    : choi
ansible_ssh_private_key_file    : /home/choi/.ssh/ansible

$ cat inventory
[PROD_SERVERS_WEB]
node01  ansible_host=192.168.219.101
node02  ansible_host=192.168.219.102
```

#### (2) playbook 작성

- (2-1) ping test
```bash
$ cat playbook1.yml
cat playbook1.yml
---
- name: Test Connection to my server
  hosts: all
  become: yes

  tasks:
  - name: Ping my Servers
    ping:

$ ansible-playbook playbook1.yml --ask-become-pass
```

- (2-2) httpd 설치 및 서비스 restart/enable
```bash
$ cat playbook2.yml
---
- name: Install default Apache Web Server
  hosts: all
  become: yes

  tasks:
  - name: Install Apache Web Server
    yum:
     name: httpd
     state: latest

  - name: Start Apache and Enable it on the every boot
    service:
     name: httpd
     state: started
     enabled: yes

$ ansible-playbook playbook2.yml --ask-become-pass
```

- (2-3) httpd 설치 및 서비스 restart/enable, default_site.html 파일을 각 노드에 배포
```bash
$ cat playbook3.yml
---
- name: Install Apache and Upload my Web Page
  hosts: all
  become: yes

  vars:
    source_file: ./files/default_site.html
    destin_file: /var/www/html/index.html

  tasks:
  - name: Install Apache Web Server
    yum:
      name: httpd
      state: latest

  - name: Copy MyHomePage to Servers
    copy:
      src: "{{ source_file }}"
      dest: "{{ destin_file }}"
      mode: 0555
    notify: Restart Apache

  - name: Start Web Server and make it enable on boot
    service:
       name: httpd
       state: started
       enabled: yes

  handlers:
  - name: Restart Apache
    service:
      name: httpd
      state: restarted

$ ansible-playbook playbook3.yml --ask-become-pass
```