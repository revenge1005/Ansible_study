## 12. Interception and Error Control

### 1) "failed_when"과 "changed_when"

- **changed_when/failed_when은 정의한 조건에 만족/불만족 했을 경우에 task가 failed 또는 success(changed)로 표시**된다, 이 구문의 가장 큰 목적은 task가 실제로 성공 또는 실패했는지를 결정하는 것이다.

<br>

### 2) changed_when

- **changed 상태값은 해딩 task가 어떤것을 변경했는지 여부를 표시한다.**

- 일반적으로 shell/command/script 등과 같은 명령어 계열의 모듈은 상태변경을 감지하는 기능이 없다.

- 별도로 설정하지 않는 한 이 명령어 모듈의 결과는 항상 failed/changed 또는 skipped를 리턴한다.

- << changed_when 사용하지 않을 경우 >>

    - **정의한 task가 실제로 실행되었는지를 확인하는 것이 필요할 경우가 있다.**

    - 예를 들어 apache http server를 실행하려 했는데, 이미 실행중이라면 실제로 실행하지 못했다고 판다해야 하며, 이러한 실행 여부를 확인하는 값이 changed라는 파라미터이고 각 task가 실행되면 노란색으로 표시된다.

    ```bash
    $ vi test-changed_when.yml
    - name: TEST changed_when
      hosts: "Agents"
      gather_facts: no
      tasks:
      - name: "Start the Apache HTTPD Server"
        become: true
        become_user: root
        register: starthttpdout
        shell: "httpd -k start"
      - debug:
          msg: "{{starthttpdout.stdout}}"

    -----------------------------------------------------------------------------------

    TASK [Start the Apache HTTPD Server] **********************************************************************************
    changed: [agent1]

    TASK [debug] **********************************************************************************************************
    ok: [agent1] => {
        "changed": false,
        "msg": "httpd (pid 2734) already running"
    }

    PLAY RECAP ************************************************************************************************************
    agent1                     : ok=2    changed=1    unreachable=0    failed=0
    ```

- << changed_when 사용할 경우 >>

    - **이미 http server가 구동중이라면 "start apache http server" task를 실행하지 않은것이므로, change 상태가 변경되지 않았다고 정의한다. (changed가 없고 ok만 표시됨)**

    - 이를 changed_when에서 "httpd -k start"를 실행했을 때 출력된 결과를 이용하여 상태를 정의한다.

    ```bash
    - name: "Start the Apache HTTPD Server"
      become: true
      become_user: root
      register: starthttpdout
      shell: "httpd -k start"
      changed_when: " 'already running' not in starthttpdout.stdout "
    
    -----------------------------------------------------------------------------------
    
    TASK [Start the Apache HTTPD Server] **********************************************************************************
    ok: [agent1]

    TASK [debug] **********************************************************************************************************
    ok: [agent1] => {
        "changed": false,
        "msg": "httpd (pid 2734) already running"
    }

    PLAY RECAP ************************************************************************************************************
    agent1                     : ok=2    changed=0    unreachable=0    failed=0
    ```

### 1) ignore_errors

```bash
$ cat playbook_errorhandling.yml
---
- name: Ansible Tutorial 12
  hosts: ROCKEY
  become: yes

  tasks:
  # 에러 발생
  - name: Task Number1
    yum:
      name: treeeeee
      state: latest

  - name: Task Number2
    shell: echo HellWorld!

  - name: Task Number3
    shell: echo Privet Mir!!!

$ ansible-playbook --ask-become-pass playbook_errorhandling.yml
BECOME password:

PLAY [Ansible Tutorial 12] ************************************************************************

TASK [Gathering Facts] ****************************************************************************
ok: [node04]
ok: [node03]

# 에러 발생
TASK [Task Number1] *******************************************************************************
fatal: [node03]: FAILED! => {"changed": false, "failures": ["No package treeeeee available."], "msg": "Failed to install some of the specified packages", "rc": 1, "results": []}
fatal: [node04]: FAILED! => {"changed": false, "failures": ["No package treeeeee available."], "msg": "Failed to install some of the specified packages", "rc": 1, "results": []}

# 에러 발생으로 나머지 task를 실행하지 않고 play를 중단
PLAY RECAP ****************************************************************************************
node03                     : ok=1    changed=0    unreachable=0    failed=1    skipped=0    rescued=0    ignored=0
node04                     : ok=1    changed=0    unreachable=0    failed=1    skipped=0    rescued=0    ignored=0
```

```bash
$ cat playbook_errorhandling.yml
---
- name: Ansible Tutorial 12
  hosts: ROCKEY
  become: yes

  tasks:
  # "ignore_errors: yes" 추가
  - name: Task Number1
    yum:
      name: treeeeee
      state: latest
    ignore_errors: yes

  - name: Task Number2
    shell: echo HellWorld!

  - name: Task Number3
    shell: echo Privet Mir!!!


$ ansible-playbook --ask-become-pass playbook_errorhandling.yml
BECOME password:

PLAY [Ansible Tutorial 12] ************************************************************************

TASK [Gathering Facts] ****************************************************************************
ok: [node03]
ok: [node04]

TASK [Task Number1] *******************************************************************************
fatal: [node03]: FAILED! => {"changed": false, "failures": ["No package treeeeee available."], "msg": "Failed to install some of the specified packages", "rc": 1, "results": []}
...ignoring
fatal: [node04]: FAILED! => {"changed": false, "failures": ["No package treeeeee available."], "msg": "Failed to install some of the specified packages", "rc": 1, "results": []}
...ignoring

TASK [Task Number2] *******************************************************************************
changed: [node03]
changed: [node04]

TASK [Task Number3] *******************************************************************************
changed: [node04]
changed: [node03]

PLAY RECAP ****************************************************************************************
node03                     : ok=4    changed=2    unreachable=0    failed=0    skipped=0    rescued=0    ignored=1
node04                     : ok=4    changed=2    unreachable=0    failed=0    skipped=0    rescued=0    ignored=1
```

---

### 2) failed_when 

```bash
$ cat playbook_errorhandling.yml
---
- name: Ansible Tutorial 12
  hosts: ROCKEY
  become: yes

  tasks:
  - name: Task Number1
    yum:
      name: treeeeee
      state: latest
    ignore_errors: yes

  - name: Task Number2
    shell: echo HellWorld!
    register: results

  - debug:
      var: results

  - name: Task Number3
    shell: echo Privet Mir!!! 

$ ansible-playbook --ask-become-pass playbook_errorhandling.yml
BECOME password:

PLAY [Ansible Tutorial 12] ************************************************************************

TASK [Gathering Facts] ****************************************************************************
ok: [node03]
ok: [node04]

TASK [Task Number1] *******************************************************************************
fatal: [node03]: FAILED! => {"changed": false, "failures": ["No package treeeeee available."], "msg": "Failed to install some of the specified packages", "rc": 1, "results": []}
...ignoring
fatal: [node04]: FAILED! => {"changed": false, "failures": ["No package treeeeee available."], "msg": "Failed to install some of the specified packages", "rc": 1, "results": []}
...ignoring

TASK [Task Number2] *******************************************************************************
changed: [node04]
changed: [node03]

TASK [debug] **************************************************************************************
ok: [node03] => {
    "results": {
        "changed": true,
        "cmd": "echo Hellorld!",
        "delta": "0:00:00.004675",
        "end": "2022-02-09 13:39:51.460972",
        "failed": false,
        "msg": "",
        "rc": 0,
        "start": "2022-02-09 13:39:51.456297",
        "stderr": "",
        "stderr_lines": [],
        "stdout": "HellWorld!",
        "stdout_lines": [
            "HellWorld!"
        ]
    }
}
ok: [node04] => {
    "results": {
        "changed": true,
        "cmd": "echo Hellorld!",
        "delta": "0:00:00.004548",
        "end": "2022-02-09 13:39:50.617684",
        "failed": false,
        "msg": "",
        "rc": 0,
        "start": "2022-02-09 13:39:50.613136",
        "stderr": "",
        "stderr_lines": [],
        "stdout": "HellWorld!",
        "stdout_lines": [
            "HellWorld!"
        ]
    }
}

TASK [Task Number3] *******************************************************************************
changed: [node04]
changed: [node03]

PLAY RECAP ****************************************************************************************
node03                     : ok=5    changed=2    unreachable=0    failed=0    skipped=0    rescued=0    ignored=1
node04                     : ok=5    changed=2    unreachable=0    failed=0    skipped=0    rescued=0    ignored=1


$ cat playbook_errorhandling.yml 
---
- name: Ansible Tutorial 12
  hosts: ROCKEY
  become: yes

  tasks:
  - name: Task Number1
    yum:
      name: treeeeee
      state: latest
    ignore_errors: yes

  - name: Task Number2
    shell: echo HellWorld!
    register: results
    failed_when: "'World' in results.stdout"
#   failed_when: results.rc == 0   

  - debug:
      var: results

  - name: Task Number3
    shell: echo Privet Mir!!!


$ ansible-playbook --ask-become-pass playbook_errorhandling.yml
BECOME password:

PLAY [Ansible Tutorial 12] ************************************************************************

TASK [Gathering Facts] ****************************************************************************
ok: [node03]
ok: [node04]

TASK [Task Number1] *******************************************************************************
fatal: [node03]: FAILED! => {"changed": false, "failures": ["No package treeeeee available."], "msg": "Failed to install some of the specified packages", "rc": 1, "results": []}
...ignoring
fatal: [node04]: FAILED! => {"changed": false, "failures": ["No package treeeeee available."], "msg": "Failed to install some of the specified packages", "rc": 1, "results": []}
...ignoring

TASK [Task Number2] *******************************************************************************
fatal: [node04]: FAILED! => {"changed": true, "cmd": "echo HellWorld!", "delta": "0:00:00.004375", "end": "2022-02-09 13:54:52.459316", "failed_when_result": true, "msg": "",
"rc": 0, "start": "2022-02-09 13:54:52.454941", "stderr": "", "stderr_lines": [], "stdout": "HellWorld!", "stdout_lines": ["HellWorld!"]}
fatal: [node03]: FAILED! => {"changed": true, "cmd": "echo HellWorld!", "delta": "0:00:00.004967", "end": "2022-02-09 13:54:53.283126", "failed_when_result": true, "msg": "", 
rc": 0, "start": "2022-02-09 13:54:53.278159", "stderr": "", "stderr_lines": [], "stdout": "HellWorld!", "stdout_lines": ["HellWorld!"]}

PLAY RECAP ****************************************************************************************
node03                     : ok=2    changed=0    unreachable=0    failed=1    skipped=0    rescued=0    ignored=1
node04                     : ok=2    changed=0    unreachable=0    failed=1    skipped=0    rescued=0    ignored=1
```

