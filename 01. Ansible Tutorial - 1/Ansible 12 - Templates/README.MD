## 12. Templates

```
mkdir roles/base/templates

cd roles/base/templates

cp /etc/ssh/sshd_config sshd_config_ubuntu.j2

scp root@192.168.219.104:/etc/ssh/sshd_config sshd_config_centos.j2

cat <<EOF >> sshd_config_ubuntu.j2

AllowUsers {{ ssh_users }}
EOF

cat <<EOF >> sshd_config_centos.j2

AllowUsers {{ ssh_users }}
EOF
```

```
cd ../../../host_vars/

cat <<EOF >> 192.168.219.101.yml
ssh_users: "choi simone"
ssh_template_file: sshd_config_ubuntu.j2
EOF

cp 192.168.219.101.yml 192.168.219.102.yml

cat <<EOF >> 192.168.219.103.yml
ssh_users: "choi simone"
ssh_template_file: sshd_config_centos.j2
EOF

cp 192.168.219.103.yml 192.168.219.104.yml
```

```
cd ../roles/base/tasks/

cat <<EOF > main.yml
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

- name: sshd_config file from template
  tags: ssh
  template:
    src: "{{ ssh_template_file }}"
    dest: /etc/ssh/sshd_config
    owner: root
    group: root
    mode: 0644
  notify: restart_sshd
EOF

cd roles/base/

mkdir handlers

cat <<EOF > handlers/main.yml
- name: restart_sshd
  service:
    name: sshd
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
│   ├── 192.168.219.103.yml
│   └── 192.168.219.104.yml
├── inventory
├── roles
│   ├── base
│   │   ├── files
│   │   │   └── sudoer_simone
│   │   ├── handlers
│   │   │   └── main.yml
│   │   ├── tasks
│   │   │   └── main.yml
│   │   └── templates
│   │       ├── sshd_config_centos.j2
│   │       └── sshd_config_ubuntu.j2
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

```
ansible-playbook site6.yml
```