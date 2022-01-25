## 8. Managing Files

- CentOS httpd 설치 후, 서비스 Start/Enable

- default html 파일, 웹 서버에 배포

- CentOS 웹 서버의 /etc/httpd/conf/httpd.conf 설정 파일 수정

- 설정 변경 후, 서비스 Restart

```
mkdir files

vim files/default_site.html
<html>
  <title>Web-site Test</title>
  <body>
	  <p> Ansible is awesome!</p>
  </body>
</html>
```

```
cat <<EOF > site3.yml
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

  - name: start httpd (CentOS)
    tags: apache,centos,httpd
    service:
      name: httpd
      state: started
      enabled: yes
    when: ansible_distribution == "CentOS"

  - name: change e-mail address for admin
    tags: apache,centos,httpd
    lineinfile:
      path: /etc/httpd/conf/httpd.conf
      regexp: '^ServerAdmin'
      line: ServerAdmin revenge1005@test.com
    when: ansible_distribution == "CentOS"
    register: httpd

  - name: change server name 
    tags: apache,centos,httpd
    lineinfile:
      path: /etc/httpd/conf/httpd.conf
      regexp: '^#ServerName'
      line: ServerName choi.example.com
    when: ansible_distribution == "CentOS"
    register: httpd

  - name: restart httpd (CentOS)
    tags: apache,centos,httpd
    service:
      name: httpd
      state: restarted
    when: httpd.changed

  - name: copy default httml file for site
    tags: apache,apache2,httpd
    copy:
      src: default_site.html
      dest: /var/www/html/index.html
      owner: root
      group: root
      mode: 0644

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
