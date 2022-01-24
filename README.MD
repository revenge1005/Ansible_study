## 1. 테스트 환경 구성

## 2. 사전 구성

#### (1) Workstation
```
apt update 

apt -y install ansible

ssh-keygen -t ed25519 -C "ansible" 

ssh-copy-id -i ~/.ssh/ansible.pub <Node01_IP_Address>

eval $(ssh-agent) && ssh-add
```

