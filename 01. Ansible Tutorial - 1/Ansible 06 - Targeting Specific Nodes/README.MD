## 6. Targeting Specific Nodes

```
cat <<EOF > inventory
[web_servers]
192.168.219.101
192.168.219.103

[db_servers]
192.168.219.104

[file_servers]
192.168.219.104
EOF
```

```
cat <<EOF > site.yml
---

- hosts: all
  become: true
  pre_tasks:
  
  - name: install updates (CentOS)
    dnf:
      update_only: yes
      update_cache: yes
    when: ansible_distribution == "CentOS"

  - name: install updates (Ubuntu)
    apt:
      upgrade: dist
      update_cache: yes
    when: ansible_distribution == "Ubuntu"

- hosts: web_servers
  become: true
  tasks:

  - name: install apache2 and php package for Ubuntu Server
    apt:
      name: 
        - apache2
        - libapache2-mod-php
      state: latest
    when: ansible_distribution == "Ubuntu"

  - name: install Httpd and php package for CentOS Server
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
    dnf:
      name: mariadb-server
      state: latest
    when: ansible_distribution == "CentOS"

  - name: install mariadb package (Ubuntu)
    apt:
      name: mariadb-server
      state: latest
    when: ansible_distribution == "Ubuntu"

- hosts: file_servers
  become: true
  tasks:
  
  - name: install samba package
    package:
      name: samba
      state: latest
EOF

ansible-playbook --ask-become-pass site.yml
```