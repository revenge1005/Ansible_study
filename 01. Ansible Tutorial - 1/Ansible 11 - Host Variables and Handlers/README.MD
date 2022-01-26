## 11. Host Variables and Handlers

- https://gruuuuu.github.io/ansible/ansible-role/

#### (1) Host Variable

```
mkdir host_vars

cd host_vars

cat <<EOF > 192.168.219.101.yml
apache_package_name: apache2
apache_service: apache2
php_package_name: libapache2-mod-php
EOF

cp 192.168.219.101.yml 192.168.219.102.yml

cat <<EOF > 192.168.219.103.yml
apache_package_name: httpd
apache_service: httpd
php_package_name: php
EOF
```

```
tree
.
├── ansible.cfg
├── host_vars
│   ├── 192.168.219.101.yml
│   ├── 192.168.219.102.yml
│   └── 192.168.219.103.yml
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

```
cat <<EOF > roles/web_servers/tasks/main.yml
- name: install apache and php packages
  tags: apache,httpd,php
  package:
    name:
      - "{{ apache_package_name }}"
      - "{{ php_package_name }}"
    state: latest

- name: start and enable apache service
  tags: apache,httpd
  service:
    name: "{{ apache_service }}"
    state: started
    enabled: yes

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
  tags: apache,httpd
  service:
    name: "{{ apache_service }}"
    state: restarted
  when: httpd.changed

- name: copy default html file for site
  tags: apache,apache2,httpd
  copy:
    src: default_site.html
    dest: /var/www/html/index.html
    owner: root
    group: root
    mode: 0644
EOF
```

```
ansible-playbook site6.yml
```

#### (2) Handlers
```
cat <<EOF > roles/web_servers/tasks/main.yml
- name: install apache and php packages
  tags: apache,httpd,php
  package:
    name:
      - "{{ apache_package_name }}"
      - "{{ php_package_name }}"
    state: latest

- name: start and enable apache service
  tags: apache,httpd
  service:
    name: "{{ apache_service }}"
    state: started
    enabled: yes

- name: change e-mail address for admin
  tags: apache,centos,httpd
  lineinfile:
    path: /etc/httpd/conf/httpd.conf
    regexp: '^ServerAdmin'
    line: ServerAdmin revenge1005@test.com
  when: ansible_distribution == "CentOS"
  notify: restart_apache

- name: copy default html file for site
  tags: apache,apache2,httpd
  copy:
    src: default_site.html
    dest: /var/www/html/index.html
    owner: root
    group: root
    mode: 0644
EOF
```

```
mkdir roles/web_servers/handlers

cat <<EOF > roles/web_servers/handlers/main.yml
- name: restart_apache
  service:
    name: "{{ apache_service }}"
    state: restarted
EOF
```

```
tree
.
├── ansible.cfg
├── host_vars
│   ├── 192.168.219.101.yml
│   ├── 192.168.219.102.yml
│   └── 192.168.219.103.yml
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
│   │   ├── handlers
│   │   │   └── main.yml
│   │   └── tasks
│   │       └── main.yml
│   └── workstations
│       └── tasks
│           └── main.yml
└── site6.yml
```
![01212dad](https://user-images.githubusercontent.com/42735894/151129575-697eadaa-8e55-442c-9a1a-3e276f3cf248.PNG)

