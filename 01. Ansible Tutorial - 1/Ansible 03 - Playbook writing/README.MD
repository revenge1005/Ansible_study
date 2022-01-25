## 3. Playbook writing


#### (1) install
```
cat <<EOF > install_apache.yml
---

- hosts: all
  become: true
  tasks:

  - name: update repository index
    apt:
      update_cache: yes

  - name: install apache2 package
    apt:
      name: apache2
      state: latest

  - name: add php support for apache
    apt:
      name: libapache2-mod-php
      state: latest
EOF

ansible-playbook --ask-become-pass install_apache.yml
```
![003](https://user-images.githubusercontent.com/42735894/150816977-6842f8d8-08b0-4c4b-8476-fce73b2eb84b.PNG)

#### (2) remove
```
cat <<EOF > remove_apache.yml
---

- hosts: all
  become: true
  tasks:

  - name: update repository index
    apt:
      update_cache: yes

  - name: install apache2 package
    apt:
      name: apache2
      state: absent

  - name: add php support for apache
    apt:
      name: libapache2-mod-php
      state: absent
EOF

ansible-playbook --ask-become-pass remove_apache.yml
```