## 1. 테스트 환경 구성
![001](https://user-images.githubusercontent.com/42735894/150800286-046d32de-f2df-4350-98c6-50a718dd38af.PNG)

## 2. 사전 구성

#### (1) Workstation
```
apt update 

apt -y install ansible

ssh-keygen -t ed25519 -C "ansible" 

ssh-copy-id -i ~/.ssh/ansible.pub <Node01_IP_Address>

eval $(ssh-agent) && ssh-add
```

