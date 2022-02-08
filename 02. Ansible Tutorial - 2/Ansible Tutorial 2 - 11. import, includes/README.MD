## 11. import, includes

- https://watch-n-learn.tistory.com/91?category=841096

### 1) import: static re-use

    - role, task, playbook 등을 플레이북에 정적으로(statically) 삽입한다.

    - ansible은 playbook을 처음 구문 분석할 때 읽어들인 파일이나 role 등을 최상위 playbook에서 작업을 실행하기 전에 사 처리한다

    - 따라서 imported 된 내용들은 최상위 playbook의 다른 task에 영향을 받지 않는다.

### 2) includes: dynamic ru-use

    - role, task, 변수 등을 Playbook에 동적으로(dynamically) 삽입한다.

    - ansible은 file, role을 playbook을 읽어들일 때 그것들을 include 한다.

    - included 된 컨텐츠는 최상위 플레이북의 이전 작업의 결과에 영향을 받는다.

    - included 된 role 이나 task는 핸들러와 비슷하다.

    - 그것들은 최상위 플레이북의 다른 task의 결과에 따라 실행될수도 있고 실행되지 않을수도 있다.

---

### 3) playbook 가져오기 - import_playbook

#### << 예시 - 1 >>

```bash
---
- name: prepare the web server
  import_playbook: web.yml
  
- name: Prepare the database server
  import_playbook: db.yml
```

#### << 예시 - 2 >>

```bash
- name: play 1
  hosts: localhost
  tasks:
    - debug:
        msg: play 1
        
- name: import playbook
  import_playbook: play2.yml
```

---

### 4) task 가져오기 (1) - import_tasks

- task 내용이 있는 파일을 import 방식으로 가져올 수 있다.

- 반복문은 import_task와 함께 사용할 수 없으며, 또한 변수를 사용하여 가져올 파일 이름을 지정할 때, 호스트/그룹 인벤토리 변수를 사용할 수 없다.

```bash
- name: install web server
  hosts: webservers
  tasks:
    - import_tasks: webserver_tasks.yml
```

### 5) task 가져오기 (2) - include_tasks

```bash
---
- name: install web server
  hosts: webservers
  tasks:
    - include_tasks: webserver_tasks.yml
```

---

### 6) 재사용 극대화 예시

- 재사용 가능성을 극대화하려면, 이러한 task와 play 파일을 최대한 일반화 해야 한다.

- 변수를 사용하여 play 및 task의 요소를 매개변수화하여 유연하게 사용할 수 있다.

#### << import_tasks 관련 예시 >>

- 예시 1: 어떤 패키지든 명시해서 최신으로 설치, 서비스 시작, enable 까지 수행하는 일반화 된 task 파일

```bash
$ package_task.yml
---
- name: install the {{ package }} packge.
  yum:
    name: "{{ package }}"
    state: latest

- name: start the {{ service }} service.
  service:
    name: "{{ service }}"
    enabled: true
    state: started
```

- 예시 2: 위 task 파일을 메인 playbook에 포함하여 사용하기 

```bash
---
- name: work
- hosts: all
- tasks: 
    - name: import task file ans set variables
      import_tasks: package_task.yml
      vars:
        package: httpd
        service: httpd
```

#### << import_playbook 관련 예시 >>

- 예시 : 일반화된 플레이북을 포함해서 사용하는 플레이북

- 위의 import_tasks의 예시 2와 같은 맥락으로, 일반화되어 잘 만들어진 플레이북에 변수 mariadb만 적용하는 플레이북이다.

```
- name: import play file and set the variable
  import_playbook: play.yml
  vars:
    package: mariadb
```

---

### 간단한 예시

```bash
$ tree
.
├── ansible.cfg
├── environment.yml
├── firewall.yml
├── inventory
├── placeholder.yml
├── playbook.yml
└── test.yml
```

```bash
# 전체 마스터 파일
$ cat playbook.yml
---
- name: configure web server
  hosts: ROCKEY
  become: yes
  tasks:

    # 패키지 설치 및 서비스를 시작, enable을 수행
    - name: import the environment task file ans set the variables
      import_tasks: environment.yml
      vars:
        package: httpd
        service: httpd

    # 방화벽 소프트웨어를 설치 및 구성
    - name: import the firewall task file and set the variable.
      import_tasks: firewall.yml
      vars:
        firewall_pkg: firewalld
        firewall_svc: firewalld
        rule: http

    # 웹 페이지 파일을 생성
    - name: import the placeholder task file and set the variable
      import_tasks: placeholder.yml
      vars:
        file: /var/www/html/index.html

# 모든 작업 수행 후, 제대로 작동하는지 테스트
- name: import test play file and set the variable
  import_playbook: test.yml
  vars:
    url: 'http://node03'


# 패키지 설치 및 서비스 시작, enable 수행
$ cat environment.yml
---
  - name: install the {{ package }} packge.
    yum:
      name: "{{ package }}"
      state: latest

  - name: start the {{ service }} service.
    service:
      name: "{{ service }}"
      enabled: true
      state: started


# 방화벽 소프트웨어 설치 및 구성
$ cat firewall.yml
---
  - name: install the firewall
    yum:
      name: "{{ firewall_pkg }}"
      state: latest

  - name: start the firewall
    service:
      name: "{{ firewall_svc }}"
      enabled: true
      state: started

  - name: open the port for {{ rule }}
    firewalld:
      service: "{{ rule }}"
      immediate: true
      permanent: true
      state: enabled


# html 파일 생성
$ cat placeholder.yml
---
  - name: create placeholder file
    copy:
      content: "{{ ansible_facts['fqdn'] }} has been customized using ansible. \n"
      dest: "{{ file }}"


# 연결 테스트
$ cat test.yml
---
  - name: test web service
    hosts: localhost 
    become: no
    tasks:
      - name: connect to internet web server
        uri:
          url: "{{ url }}"
          status_code: 200
```