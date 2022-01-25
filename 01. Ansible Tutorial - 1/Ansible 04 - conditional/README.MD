## 4. Conditional 


#### (1) Node OS 정보 확인
```
ansible all -m gather_facts --limit 192.168.219.101 | grep ansible_distribution

        "ansible_distribution": "Ubuntu",
        "ansible_distribution_file_parsed": true,
        "ansible_distribution_file_path": "/etc/os-release",
        "ansible_distribution_file_variety": "Debian",
        "ansible_distribution_major_version": "20",
        "ansible_distribution_release": "focal",
        "ansible_distribution_version": "20.04",
```

#### (3) (centos) Node 추가
```
cat <<EOF >> inventory
192.168.219.103
EOF
```

#### (2) install 
```
cat <<EOF > install_apache2.yml
---

- hosts: all
  become: true
  tasks:

  - name: update repository index (Ubuntu)
    apt:
      update_cache: yes
    when: ansible_distribution == "Ubuntu" and ansible_distribution_version == "20.04"

  - name: install apache2 package (Ubuntu)
    apt:
      name: apache2
      state: latest
    when: ansible_distribution in ["Ubuntu", "Debiain"]
  
  - name: install PHP package (Ubuntu)
    apt:
      name: libapache2-mod-php
      state: latest
    when: ansible_distribution == "Ubuntu"

  - name: update repository index (CentOS)
    dnf:
      update_cache: yes
    when: ansible_distribution == "CentOS"

  - name: install httpd packge (CentOS)
    dnf:
      name: httpd
      state: latest
    when: ansible_distribution == "CentOS"

  - name: install PHP packge (CentOS)
    dnf:
      name: httpd
      state: latest
    when: ansible_distribution == "CentOS"
EOF

ansible-playbook --ask-become-pass install_apache2.yml
```
![dadasdwqeqw](https://user-images.githubusercontent.com/42735894/150982810-5ee53496-9f7c-4fee-9714-cda5c4f226bd.PNG)