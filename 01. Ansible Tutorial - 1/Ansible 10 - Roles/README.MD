## 10. Roles

```
cat <<EOF > site6.yml
---

- hosts: all
  become: true
  pre_tasks:
  
  - name: update repository index (CentOS)
    tags: always
    dnf:
      update_cache: yes
    changed_when: false
    when: ansible_distribution == "CentOS"

  - name: update repository index (Ubuntu)
    tags: always
    apt:
      update_cache: yes
    changed_when: false
    when: ansible_distribution == "Ubuntu"

- hosts: all
  become: true
  roles:
    - base

- hosts: workstations
  become: true
  roles:
    - workstations

- hosts: web_servers
  become: true
  roles:
    - web_servers

- hosts: db_servers
  become: true
  roles:
    - db_servers

- hosts: file_servers
  become: true
  roles:
    - file_servers
EOF
```

```
for name in base db_servers file_servers web_servers workstations
do
  mkdir -p roles/$name/tasks
  touch roles/$name/tasks/main.yml
done

mkdir roles/base/files
mv files/sudoer_simone roles/base/files
mv files roles/web_servers/
```

#### (1) roles/base/tasks/main.yml
```
cat << EOF > roles/base/tasks/main.yml
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
```

#### (2) roles/workstations/tasks/main.yml
```
cat << EOF > roles/workstations/tasks/main.yml
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
EOF 
```

#### (3) roles/web_servers/tasks/main.yml
```
cat << EOF > roles/web_servers/tasks/main.yml
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
EOF
```

#### (4) roles/db_servers/tasks/main.yml
```
cat << EOF > roles/db_servers/tasks/main.yml
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
EOF
```

#### (5) roles/file_servers/tasks/main.yml
```
cat << EOF > roles/file_servers/tasks/main.yml
- name: install samba package
  tags: samba
  package:
    name: samba
    state: latest
EOF
```

#### (6) 결과
```
tree
.
├── ansible.cfg
├── inventory
├── roles
│   ├── base
│   │   ├── files
│   │   │   └── sudoer_simone
│   │   └── tasks
│   │       └── main.yml
│   ├── db_servers
│   │   └── tasks
│   │       └── main.yml
│   ├── file_servers
│   │   └── tasks
│   │       └── main.yml
│   ├── web_servers
│   │   ├── files
│   │   │   └── default_site.html
│   │   └── tasks
│   │       └── main.yml
│   └── workstations
│       └── tasks
│           └── main.yml
└── site6.yml
```