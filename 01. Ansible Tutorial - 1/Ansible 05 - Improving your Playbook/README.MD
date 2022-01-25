## 5. Improving your Playbook 

#### (1)

```
cat <<EOF > install_apache3.yml
---

- hosts: all
  become: true
  tasks:

  - name: install apache2 and php package for Ubuntu
    apt:
      name: 
        - apache2
        - libapache2-mod-php
      state: latest
      update_cache: yes
    when: ansible_distribution == "Ubuntu"
  
  - name: install httpd and php packge for CentOS
    dnf:
      name: 
        - httpd
        - php
      state: latest
      update_cache: yes
    when: ansible_distribution == "CentOS"
EOF

ansible-playbook --ask-become-pass install_apache3.yml
```

#### (2)

- https://docs.ansible.com/ansible/latest/collections/ansible/builtin/package_module.html

```
cat <<EOF > install_apache4.yml
---

- hosts: all
  become: true
  tasks:

  - name: install apache2 and php package 
    package:
      name: 
        - "{{ apache_package }}"
        - "{{ php_package }}"
      state: latest
      update_cache: yes
EOF

cat <<EOF > inventory
192.168.219.101 apache_package=apache2 php_package=libapache2-mod-php
192.168.219.102 apache_package=apache2 php_package=libapache2-mod-php
192.168.219.103 apache_package=httpd php_package=php
EOF

ansible-playbook --ask-become-pass install_apache4.yml
```