## 9. Roles 

- Ansible에서는 "Role"을 사용하여 Ansible 코드를 더 쉽게 재사용하는 방법을 제공한다.

- Role은 인프라를 프로비저닝하고, 애플리케이션을 배포하는 등의 모든 작업에 대해 변수, 파일, 템플릿 등의 기타 **리소스들을 표준화된 디렉토리 구조로 패키징**한다.

- 즉, Role은 쉬운 공유, 휴대성 및 재사용을 위해 **특정 구조로 배열된 yaml 작업파일 및 지원 항목들의 모음**이라고 할 수 있다.

- << 참고 >>

    - https://watch-n-learn.tistory.com/92?category=841096


#### 1) ansible role의 구성요소

- Role은 하위 디렉토리 및 파일의 표준화된 구조에 의해 정의된다.

- **최상위 디렉토리는 role 자체의 이름으로 정의**하며, 그 아래의 파일들은 tasks, handler 등 **role에서 각 파일의 목적에 따라 이름이 지정된 하위 디렉토리로 구성**된다.

![dada](https://user-images.githubusercontent.com/42735894/152728225-0add9b6f-7727-43a0-befe-60c3b721af37.PNG)

![2dasd](https://user-images.githubusercontent.com/42735894/152733677-b99fcd64-7136-41c0-8545-412434334322.PNG)

#### 2) ansible role 사용법

- 아래 예시대로 **"roles 지시자"**를 사용하면 된다.

```bash
---
- host: all
  roles:
    - role01
    - role02
```

- 이렇게 지정한 role01, role02의 디렉토리에서 tasks, heandlers, vars 등 모든 내용들을 불러와서 해당 role을 실행한다.

- role에서 cp, script, template 또는 import_task, include_task 등은 절대경로/상대경로, 이름이 없는 관련 파일, 템플릿 또는 작업 파일을 참조할 수 있다.

- ansible은 files, templates, tasks 하위 디렉토리에서 이러한 각각의 항목들을 찾는다.

#### 3) role이 포함된 playbook에서 실행 순서 제어하기

- role이 포함된 playbook에서 실행 순서를 제어하기 위해 2가지 지시자를 사용한다 ("pre_tasks", "post_tasks")

- **pre_tasks에 나열된 작업은 모든 role, 일반 tasks가 실행되기 전에 제일 먼저 실행된다.**

- **post_tasks는 이러한 작업은 pre_tasks, tasks, roles가 모두 실행된 후에 마지막에 실행된다.**

#### 4) playbook에 role 포함하기

- include_role   : role을 동적으로 포함 (ansible 2.3 version에서 추가된 기능)

- import_role    : role을 동적으로 포함 (ansible 2.3 version에서 추가된 기능)

```
---
- name: execute a role as a task
  hosts: web.example.com
  tasks:
    - name: a normal task
      debug:
        msg: 'first task'
    - name: a task to include role2 here
      include_role: role2
```