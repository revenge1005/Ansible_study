## 1. 테스트 환경 구성
![001](https://user-images.githubusercontent.com/42735894/150800286-046d32de-f2df-4350-98c6-50a718dd38af.PNG)

## 2. 사전 구성

#### (1) Workstation - Ansible install
```
sudo apt update

sudo apt install software-properties-common

sudo add-apt-repository --yes --update ppa:ansible/ansible

sudo apt install ansible
```
```
ssh-keygen -t ed25519 -C "choi default"

for i in 192.168.219.101 192.168.219.102 192.168.219.103 192.168.219.104;
do
    ssh-copy-id -i ~/.ssh/id_ed25519.pub $i
done
```
```
ssh-keygen -t ed25519 -C "ansible"
Generating public/private ed25519 key pair.
Enter file in which to save the key (/home/choi/.ssh/id_ed25519): /home/choi/.ssh/ansible

...

for i in 192.168.219.101 192.168.219.102 192.168.219.103 192.168.219.104;
do
    ssh-copy-id -i ~/.ssh/ansible.pub $i
done
```
#### (2) (CentOS) Nodes Seting
```
su

cat <<EOF > /etc/sudoers.d/011_choi-sudoer
choi    ALL=(ALL)   ALL
EOF

chmod 0440 /etc/sudoers.d/011_choi-sudoer
```

