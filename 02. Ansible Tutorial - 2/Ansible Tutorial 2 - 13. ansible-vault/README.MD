# 13. ansible-vault

- **Ansible Vault는 패스워드, 키와 같은 보안에 민감한 파일들을 암호화해주는 기능**이다.

- Ansible에서 사용하는 모든 구조화된 데이터 파일을 암호화 할 수 있다.

- **<< ansible-vault >>**

    |설명|명령|
    |----|:-----------:|
    |암호화된 파일을 생성하기|ansible-vault **create** foo.yml|
    |암호화된 파일을 수정하기|ansible-vault **edite** foo.yml|
    |암호화된 파일의 패스워드 변경하기|ansible-vault **rekey** foo.yml|
    |암호화되지 않은 파일을 암호화|ansible-vault **encrypt** foo.yml|
    |암호화된 파일의 복호화|ansible-vault **decrypt** foo.yml|
    |암호화된 파일의 내용 보기|ansible-vault **view** foo.yml|


## 1) 예제

### 암호화된 파일 create/edite/view

```bash
$ tree
.
├── ansible.cfg
├── group_vars
├── inventory
└── playbook_vault.yml

15 directories, 27 files


$ ansible-vault create mysecret.txt
New Vault password:
Confirm New Vault password:


$ cat mysecret.txt
$ANSIBLE_VAULT;1.1;AES256
63613266616531393762373066386332353835626535363761616666353566656432646165386633
6562656136323139333439396538396137333638386535380a303732633663343238386264303261
32303361613332633736373734373336396661336330326636653235613661343638393061353362
6636303636373063320a306633633932393864663536613731663565363766396431323363366563
3564


$ ansible-vault view mysecret.txt
Vault password:
Hello World!!!


$ vim mysecret.txt
$ANSIBLE_VAULT;1.1;AES256
63613266616531393762373066386332353835626535363761616666353566656432646165386633
6562656136323139333439396538396137333638386535380a303732633663343238386264303261
32303361613332633736373734373336396661336330326636653235613661343638393061353362
6636303636373063320a306633633932393864663536613731663565363766396431323363366563
3564


$ ansible-vault edit mysecret.txt
Vault password:
Hello World!!!
```

### playbook 암호화

```bash
$ cat playbook_vault.yml
---
- name: Ansible Vault Test
  hosts: all
  become:

  vars:
    admin_password: PAss0rd@1234

  tasks:
  - name: Install package Tree
    yum:
      name: tree
      state: latest

  - name: Create Config File
    copy:
      dest: "/home/choi/myconfig.conf"
      content: |
        port = 9092
        log = 7days
        home = /opt/kafka/bin
        user = admin
        password = {{ admin_password }}


$ ansible-vault encrypt playbook_vault.yml
New Vault password:
Confirm New Vault password:
Encryption successful


$ cat playbook_vault.yml
$ANSIBLE_VAULT;1.1;AES256
32363239633262613235366636393038373564306661613035333538383738353230623939383837
3738666233323261666336666337343532376637373362610a383531313032303262336231646537
33353936616366353439306137666237633666376336623039653466333264383934313661386437
...(생략)


$ ansible-vault view playbook_vault.yml
Vault password:
---
- name: Ansible Vault Test
  hosts: all
  become:

  vars:
    admin_password: PAss0rd@1234

  tasks:
  - name: Install package Tree
    yum:
      name: tree
      state: latest

  - name: Create Config File
    copy:
      dest: "/home/choi/myconfig.conf"
      content: |
        port = 9092
        log = 7days
        home = /opt/kafka/bin
        user = admin
        password = {{ admin_password }}

# ansible playbook 실행
$ ansible-playbook playbook_vault.yml
ERROR! Attempting to decrypt but no vault secrets found


$ ansible-playbook playbook_vault.yml --ask-become-pass --ask-vault-pass
BECOME password:
Vault password:

PLAY [Ansible Vault Test] *************************************************************************

TASK [Gathering Facts] ****************************************************************************
ok: [node03]
ok: [node04]

TASK [Install package HTTP] ***********************************************************************
ok: [node03]
ok: [node04]

TASK [Create Config File] *************************************************************************
ok: [node03]
ok: [node04]

PLAY RECAP ****************************************************************************************
node03                     : ok=3    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
node04                     : ok=3    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```

### 패스워드 값 암호화

```bash
$ ansible-vault encrypt_string
New Vault password:
Confirm New Vault password:
Reading plaintext input from stdin. (ctrl-d to end input, twice if your content does not already have a newline)
Pass0rd@1234 # <- 암호화할 값 입력, 입력이 끝나면 Ctrl + D 
!vault |
          $ANSIBLE_VAULT;1.1;AES256
          35376239306138373464373461353964623539346434646531373231663934346163373163303362
          3365353064336165626338333135326361646530653833350a393464653634343532386532646632
          36663135383930633564386163353135336336393661366164356637623466393266626433313664
          6138383238313565350a643666613038323532376362353561643331353532336363646465316139
          6138
Encryption successful


$ ansible-vault encrypt_string --stdin-name "Mypassword"
New Vault password:
Confirm New Vault password:
Reading plaintext input from stdin. (ctrl-d to end input, twice if your content does not already have a newline)
Pass@0rd@1234
Mypassword: !vault |
          $ANSIBLE_VAULT;1.1;AES256
          39386138366363353834616239313836386564656435373938626263306230613231656538613162
          3933336464356639383665346639633836383064393639660a313037663664303961663465323339
          30396136356661336661363630316237633863646261383432613035363932656336323933356634
          3839383434373130350a363366366536653535653736656531333264636465373361663131623663
          6535
Encryption successful


$ ansible-vault view playbook_vault.yml
Vault password:
---
- name: Ansible Vault Test
  hosts: ROCKEY
  become: true
  # 암호화한 값을 입력
  vars:
    admin_password: !vault |
          $ANSIBLE_VAULT;1.1;AES256
          39386138366363353834616239313836386564656435373938626263306230613231656538613162
          3933336464356639383665346639633836383064393639660a313037663664303961663465323339
          30396136356661336661363630316237633863646261383432613035363932656336323933356634
          3839383434373130350a363366366536653535653736656531333264636465373361663131623663
          6535

  tasks:
  - name: Install package HTTP
    yum:
      name: httpd
      state: latest

  - name: Create Config File
    copy:
      dest: "/home/choi/myconfig.conf"
      content: |
        port = 9092
        log = 7days
        home = /opt/kafka/bin
        user = choi
        password = {{ admin_password }}


$ ansible-playbook playbook_vault.yml --ask-vault-pass --ask-become-pass
BECOME password:
Vault password:

PLAY [Ansible Vault Test] *************************************************************************

(생략)

PLAY RECAP ****************************************************************************************
node03                     : ok=3    changed=2    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
node04                     : ok=3    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0


$ ssh node03

[choi@node03 ~]$ cat
.ansible/      .bash_logout   .bashrc        myconfig.conf
.bash_history  .bash_profile  .ssh/

[choi@node03 ~]$ cat myconfig.conf
port = 9092
log = 7days
home = /opt/kafka/bin
user = choi
password = Pass@0rd@1234
```