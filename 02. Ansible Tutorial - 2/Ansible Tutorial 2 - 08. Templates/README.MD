## 8. Templates

#### 1) 개요

- **Ansible에서는 기존 파일의 내용을 수정하는데에는 여러 모듈을 제공**하는데 대표적으로 lineinfile, blockinfile 등이 있지만, 이것들은 단순 수정하는데는 좋지만 어떤 규칙에 의해 체계적으로 관리하는 파일에서 쓰기에는 부족했다.

- **파일을 관리하는데 더 강력한 방법은 템플릿(Tempate)을 지정**하는 것이다.

- **템플릿은 파일이 배포될 때 Ansible Client에 대해 Ansible 변수 및 팩트 변수를 사용해 자동으로 지정되는 템플릿 파일을 작성할 수 있다. (이것은 제어하기 더 쉽고 오류 발생률이 낮다.)**

- Ansible에서는 템플릿 파일을 **jinja2**라는 템플릿 시스템을 사용하며, jinja2는 위에서 말한 템플릿을 사용하는데 쓰기도 하지만, Playbook에서 지정한 변수를 참조하고 여러가지 확장된 사용을 가능하게 한다.

<br>

#### 2) jinja2 템플릿 구문 사용하기

- jinja2 템플릿은 데이터, 변수, 식 등 여러 요소로 구성되는데, **이러한 변수와 식은 jinja2 템플릿이 랜더링되면(jinja2 문법이 변역되어 실해되면) 해당 값으로 변경된다.**

- **템플릿에 사용하는 변수는 playbook의 vars 섹션에서 지정할 수 있다.**

- **템플릿에 해당하는 ansible client의 팩트변수를 사용할 수 있다.**

- << 사용 방법 >>

    - {{ }} : 식/변수 결과를 최종 사용자에게 출력

    - {% %} : 식 또는 논리(반복문 등)에 사용

    - {# #} : 주석

- **jinja2 템플릿이 들어있는 파일에는 따로 확장자명은 필요 없지만, j2 이런식으로 확장명을 제공하면 템플릿인지 쉽게 알 수 있다.**

<br>

#### 3) template 모듈

- **기능은 ansible host에 있는 파일을 ansible client에 배포하는 것**이지만, 같은 기능을 하는 모듈이 많으므로 **일반적으로 jinja2 템플릿 파일을 배포하는데 사용**된다.

#### <참고>

- https://watch-n-learn.tistory.com/88

----

#### 4) 간단 예시 - 1

- << 템플릿 파일 >>
```bash
$ cat <<EOF > test.j2
Welcome to {{ ansible_facts.hostname }}.
Today's date is : {{ ansible_facts.date_time.date }}
EOF
```
- << 플레이북 파일 >>
```bash
---
- name: jinja2 test
  hosts: all
  tasks:
    - name: jinja2 template
      template:
        src: ./test.j2
        dest: /tmp/jinja-test-result
        owner: root
        group: root
        mode: 0644
```
- << 결과 >> 
```
choi@node01:~$ cat /tmp/jinja-test-result
Welcome to node01.
Today's date is : 2022-02-07

choi@node02:~$ cat /tmp/jinja-test-result
Welcome to node02.
Today's date is : 2022-02-07

[choi@localhost ~]$ cat /tmp/jinja-test-result
Welcome to localhost.
Today's date is : 2022-02-07
```

<br>

#### 5) 간단 예시 - 2, 템플릿 파일에 관리자용 주석 달기

- 관리자가 template으로 만든 결과파일에 대해 수정하지 않도록 요구하거나 특별한 주석을 달고 싶을때, 여러 방법이 있지만 **가장 간단한 방법은 ansible.cfg의 [defaults] 섹션에 "ansible_managed=" 라는 부분을 추가하는 것**이다

```bash
$ cat ansible.cfg
[defaults]
inventory = inventory
ansible_managed = NOTICE : This file is ansible managed : {file} modified on %Y-%m-d %H:%M:%S by {uid} on {host}
```

- 이렇게 구성해둔 후, 템플릿 파일에 아래와 같이 {{ ansible_managed }} 라는 값을 맨 위에 추가

```bash
$ cat test.j2
{{ ansible_managed  }}
Welcome to {{ ansible_facts.hostname }}.
Today's date is : {{ ansible_facts.date_time.date }}
```

```bash
$ cat /tmp/jinja-test-result
NOTICE : This file is ansible managed : ./test.j2 modified on 2022-02-d 00:10:39 by choi on ansible
Welcome to node01.
Today's date is : 2022-02-07
```

<br>

#### 6) jinja2 제어구조 - 반복문

- << 사용법 >>

```bash
{% for user in users %}
    {{ user }}
{% endfor %}
```

- << 예시 1 >> : 직접 jinja2 템플릿에 변수를 정의하여 사용
```bash
cat <<EOF > jinja2-template1.j2
{% set numbers = [1,2,3,4,5,6,7,8,9,10] %}
{% for number in numbers %}
The number is {{ number }}
{% endfor %}
EOF

cat <<EOF > jinja2-template1.yml
---
- hosts: all
  become: true
  tasks:
  - template:
      src: jinja2-template1.j2
      dest: /tmp/jinja2
      mode: 0644
EOF
```

- << 예시 2 >> : 플레이북에서 직접 정의한 변수 사용

```bash
cat <<EOF > jinja2-template2.j2
{% for number in numbers %}
The number is {{ number }}
{% endfor %}
EOF

cat <<EOF > jinja2-template2.yml
---
- hosts: all
  vars:
    numbers: 
      [1,2,3,4,5,6,7,8,9,10]
  become: true
  tasks:
  - template:
      src: jinja2-template2.j2
      dest: /tmp/jinja2
      mode: 0644
EOF
```

- << 예시 결과 1,2 >>
```bash
choi@node01:~$ cat /tmp/jinja2
The number is 1
The number is 2
The number is 3
The number is 4
The number is 5
The number is 6
The number is 7
The number is 8
The number is 9
The number is 10
```

- << 예시 3 >> : 매직변수 & 팩트 변수 사용

**이렇게 여러개의 ansible host를 대상으로 팩트변수를 얻는 구문을 쓸 때는 ansible-playbook에도 해당 호스트를 동일하게 명시해야 한다.**

**"loop.index"는 반복 횟수를 보여주며, 반복 횟수에 따라 특정 구문을 실행하도록 할 때 사용할 수 있고, 여러 방면에서 활용 가능하다**

```bash
cat <<EOF > jinja2-template3.j2
{% for host in groups['all'] %}
HOST {{ hostvars[host]['ansible_facts']['hostname'] }} IPADDR : {{ hostvars[host]['ansible_facts']['default_ipv4']['address'] }}
{{ loop.index }}
{% endfor %}
EOF

cat <<EOF > jinja2-template3.yml
---
- hosts: all
  become: true
  tasks:
  - template:
      src: jinja2-template3.j2
      dest: /tmp/jinja2
      mode: 0644
EOF
```

- << 예시 결과 3 >>
```bash
$ cat /tmp/jinja2
HOST node01 IPADDR : 192.168.219.101
1
HOST node02 IPADDR : 192.168.219.102
2
HOST localhost IPADDR : 192.168.219.103
3
HOST localhost IPADDR : 192.168.219.104
4
```

<br>

#### 7) jinja2 제어구조 - 조건문

- << 사용법 >>

```bash
{% if 조건문 %}
조건이 true인 경우 실행할 내용
{% endif %}
```

<br>

- << 예시 1 >> 

**finished 변수의 값이 true 일 때만 result 변수의 값을 찍어낸다.**
```bash
{% if finished %}
{{ result }}
{% endif %}
```

<br>

- << 예시 2 >> 
```bash
{# for statement #}
{% for myuser in users if not myuser == "root" %}
User number {{loop.index}} - {{myuser }}
{% endfor %}
```