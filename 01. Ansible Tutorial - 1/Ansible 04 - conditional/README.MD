## 3. Conditional


#### (1) OS information Check 
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

#### (3) (centos)node add
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

  - name: update repository index
    apt:
      update_cache: yes
    when: ansible_distribution == "Ubuntu" 

  - name: install apache2 package
    apt:
      name: apache2
      state: latest
    when: ansible_distribution in ["Ubuntu", "Debiain"]

  - name: install httpd packge
    dnf:
      name: httpd
      state: latest
    when: ansible_distribution == "CentOS"
EOF
```