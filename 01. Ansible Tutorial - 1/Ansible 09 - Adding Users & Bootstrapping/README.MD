## 9. Adding Users & Bootstrapping

#### (1) Adding User

```
cat ~/.ssh/ansible.pub
ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIEErAl4/wC0S28kN2BZRJwjldJOX3+qs3p/e8rk6cZS7 ansible

cat <<EOF > files/sudoer_simone
simone ALL=(ALL) NOPASSWD:ALL
EOF
```

```
cat <<EOF > site4.yml
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

  - name: create simone user
    tags: always
    user:
      name: simone
      groups: root

  - name: add ssh key for simone
    tags: always
    authorized_key:
      user: simone
      key: "ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIEErAl4/wC0S28kN2BZRJwjldJOX3+qs3p/e8rk6cZS7 ansible"

  - name: add sudoers file for simone
    tags: always
    copy: 
      src: sudoer_simone
      dest: /etc/sudoers.d/simone
      owner: root
      group: root
  
- hosts: workstations
  become: true
  tasks:
  
  - name: install unzip
    package:
       name: unzip

  - name: install terraform
    unarchive:
       src: https://releases.hashicorp.com/terraform/0.12.28/terraform_0.12.28_linux_amd64.zip
       dest: /usr/local/bin
       remote_src: yes
       mode: 0755
       owner: root
       group: root

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

ansible-playbook --ask-become-pass site4.yml
```

```
ssh -i ~/.ssh/ansible simone@192.168.219.101

cat .ssh/authorized_keys
ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIEErAl4/wC0S28kN2BZRJwjldJOX3+qs3p/e8rk6cZS7 ansible

exit

ssh 192.168.219.101

cat /etc/sudoers.d/simone
simone ALL=(ALL) NOPASSWD:ALL

exit
```

```
cat <<EOF >> ansible.cfg
remote_user = simone
EOF
```

```
ansible-playbook site4.yml
```

#### (2) bootstrap

```
cat <<EOF > bootstrap.yml
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

  - name: create simone user
    tags: always
    user:
      name: simone
      groups: root

  - name: add ssh key for simone
    tags: always
    authorized_key:
      user: simone
      key: "ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIEErAl4/wC0S28kN2BZRJwjldJOX3+qs3p/e8rk6cZS7 ansible"

  - name: add sudoers file for simone
    tags: always
    copy: 
      src: sudoer_simone
      dest: /etc/sudoers.d/simone
      owner: root
      group: root
EOF

ansible-playbook --ask-become-pass bootstrap.yml
```

- https://watch-n-learn.tistory.com/86

- changed_when: false 사용
<br>
-> 이 키워드는 task에서 changed 결과를 보고하는 시점을 제어할 수 있다.
<br>
-> 실제로 changed가 나와야 되는 task에 대해서도 changed_whan: false 지시문을 포함하면 changed 로 보고되지 않는다. (ok로 보고됨)
<br>
-> true로 한다면 changed로 보고된다.

```
cat <<EOF > site5.yml
---

- hosts: all
  become: true
  pre_tasks:
  
  - name: update repo cache (CentOS)
    tags: always
    dnf:
      update_only: yes
    changed_when: false
    when: ansible_distribution == "CentOS"

  - name: update repo cache (Ubuntu)
    tags: always
    apt:
      upgrade: dist
    changed_when: false
    when: ansible_distribution == "Ubuntu"
  
- hosts: workstations
  become: true
  tasks:
  
  - name: install unzip
    package:
       name: unzip

  - name: install terraform
    unarchive:
       src: https://releases.hashicorp.com/terraform/0.12.28/terraform_0.12.28_linux_amd64.zip
       dest: /usr/local/bin
       remote_src: yes
       mode: 0755
       owner: root
       group: root

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

ansible-playbook --ask-become-pass site5.yml
```