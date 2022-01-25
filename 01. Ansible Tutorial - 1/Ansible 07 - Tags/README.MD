## 6. Targeting Specific Nodes

```
cat <<EOF > site2.yml
---

- hosts: all
  become: true
  pre_tasks:
  
  - name: install updates (CentOS)
    tags: always
    dnf:
      update_only: yes
      update_cache: yes
    when: ansible_distribution == "CentOS"

  - name: install updates (Ubuntu)
    tags: always
    apt:
      upgrade: dist
      update_cache: yes
    when: ansible_distribution == "Ubuntu"

- hosts: web_servers
  become: true
  tasks:

  - name: install apache2 and php package for Ubuntu Server
    tags: apache,apache2,ubuntu
    apt:
      name: 
        - apache2
        - libapache2-mod-php
      state: latest
    when: ansible_distribution == "Ubuntu"

  - name: install Httpd and php package for CentOS Server
    tags: apache,centos,httpd
    dnf:
      name: 
        - httpd
        - php
      state: latest
    when: ansible_distribution == "CentOS"

- hosts: db_servers
  become: true
  tasks:
  
  - name: install mariadb package (CentOS)
    tags: centos,db,mariadb
    dnf:
      name: mariadb-server
      state: latest
    when: ansible_distribution == "CentOS"

  - name: install mariadb package (Ubuntu)
    tags: db,mariadb,ubuntu
    apt:
      name: mariadb-server
      state: latest
    when: ansible_distribution == "Ubuntu"

- hosts: file_servers
  become: true
  tasks:
  
  - name: install samba package
    tags: samba
    package:
      name: samba
      state: latest
EOF

ansible-playbook --ask-become-pass site2.yml
```

```
ansible-playbook --list-tags site2.yml

playbook: site2.yml

  play #1 (all): all    TAGS: []
      TASK TAGS: [always]

  play #2 (web_servers): web_servers    TAGS: []
      TASK TAGS: [apache, apache2, centos, httpd, ubuntu]

  play #3 (db_servers): db_servers      TAGS: []
      TASK TAGS: [centos, db, mariadb, ubuntu]

  play #4 (file_servers): file_servers  TAGS: []
      TASK TAGS: [samba]
```

#### (2) 특정 tag를 가진 task만 실행
```
ansible-playbook --tags centos --ask-become-pass site2.yml

ansible-playbook --tags "apache,db" --ask-become-pass site2.yml
```