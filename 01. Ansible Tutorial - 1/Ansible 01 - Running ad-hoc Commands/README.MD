## 1. Running ad-hoc Commands


#### (1) Ping Test
```
cat <<EOF > inventory
192.168.219.101
192.168.219.101
EOF
```

```
ansible all --key-file ~/.ssh/id_ed25519 -m ping

192.168.219.102 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "changed": false,
    "ping": "pong"
}
192.168.219.101 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "changed": false,
    "ping": "pong"
}
```

```
cat <<EOF > ansible.cfg
[defaults]
inventory = inventory
private_key_file = ~/.ssh/ansible
EOF
```

```
ansible all -m ping

192.168.219.102 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "changed": false,
    "ping": "pong"
}
192.168.219.101 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "changed": false,
    "ping": "pong"
}
```

#### (3) Nodes list
```
ansible all --list-hosts
  hosts (2):
    192.168.219.101
    192.168.219.102
```

#### (4) Detail information
```
ansible all -m gather_facts
```
```
ansible all -m gather_facts --limit <node_ip_addrees>
```