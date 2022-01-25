## 1. 테스트 환경 구성
![001](https://user-images.githubusercontent.com/42735894/150800286-046d32de-f2df-4350-98c6-50a718dd38af.PNG)

## 2. 사전 구성

#### (1) Workstation
```
sudo apt update 

sudo apt -y install ansible
```
```
ssh-keygen -t ed25519 -C "choi default"

ssh-copy-id -i ~/.ssh/id_ed25519.pub <Node_IP_Address>
```
```
ssh-keygen -t ed25519 -C "ansible"
Generating public/private ed25519 key pair.
Enter file in which to save the key (/home/choi/.ssh/id_ed25519): /home/choi/.ssh/ansible

...

ssh-copy-id -i ~/.ssh/ansible.pub <Node_IP_Address>
```
#### (2) (CentOS) Nodes Seting
```
cat <<EOF > /etc/sudoers.d/011_choi-sudoer
choi    ALL=(ALL)   ALL
EOF

chmod 0440 /etc/sudoers.d/011_choi-sudoer
```