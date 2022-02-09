# 12. Interception and Error Control

<br>

## 1) "failed_when"과 "changed_when"

- **changed_when/failed_when은 정의한 조건에 만족/불만족 했을 경우에 task가 failed 또는 success(changed)로 표시**된다, 이 구문의 가장 큰 목적은 task가 실제로 성공 또는 실패했는지를 결정하는 것이다.

<br>

## 2) changed_when

- **changed 상태값은 해딩 task가 어떤것을 변경했는지 여부를 표시한다.**

- 일반적으로 shell/command/script 등과 같은 명령어 계열의 모듈은 상태변경을 감지하는 기능이 없다.

- 별도로 설정하지 않는 한 이 명령어 모듈의 결과는 항상 failed/changed 또는 skipped를 리턴한다.

- ### **【 changed_when 사용하지 않을 경우 】**

    - **정의한 task가 실제로 실행되었는지를 확인하는 것이 필요할 경우가 있다.**

    - 예를 들어 apache http server를 실행하려 했는데, 이미 실행중이라면 실제로 실행하지 못했다고 판다해야 하며, 이러한 실행 여부를 확인하는 값이 changed라는 파라미터이고 각 task가 실행되면 노란색으로 표시된다.

    ```bash
    $ cat test-changed_when.yml
    ---
    - name: TEST changed_when
      hosts: ROCKEY
      become: true
      gather_facts: no

      tasks:
      - name: "Apache install"
        yum:    
          name: httpd
          state: latest

      - name: "Start the Apache HTTPD Server"
        register: start_httpd_out
        shell: "httpd -k start"

      - debug:  
          msg: "{{ start_httpd_out.stdout }}"


    -----------------------------------------------------------------------------------

    $ ansible-playbook --ask-become-pass test-changed_when.yml
    BECOME password:

    PLAY [TEST changed_when] **************************************************************************

    TASK [Apache install] *****************************************************************************
    ok: [node03]
    changed: [node04]

    TASK [Start the Apache HTTPD Server]    **************************************************************
    changed: [node03]   
    changed: [node04]

    TASK [debug] ************************************************************************************** 
    ok: [node03] => {
        "msg": "httpd (pid 11717) already running"
    }
    ok: [node04] => {
        "msg": "httpd (pid 2973) already running"
    }

    PLAY RECAP ****************************************************************************************
    node03                     : ok=3    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
    node04                     : ok=3    changed=2    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
    ```

- ### **【 changed_when 사용할 경우 】**

    - **이미 http server가 구동중이라면 "start apache http server" task를 실행하지 않은것이므로, change 상태가 변경되지 않았다고 정의한다. (changed가 없고 ok만 표시됨)**

    - 이를 changed_when에서 "httpd -k start"를 실행했을 때 출력된 결과를 이용하여 상태를 정의한다.

    ```bash
    $ cat test-changed_when2.yml
    ---
    - name: TEST changed_when
      hosts: ROCKEY
      become: true
      gather_facts: no

      tasks:
      - name: "Apache install"
        yum:
          name: httpd
          state: latest

      - name: "Start the Apache HTTPD Server"
        register: start_httpd_out
        shell: "httpd -k start"
        changed_when: " 'already running' not in start_httpd_out.stdout "

      - debug:
          msg: "{{ start_httpd_out.stdout }}"

    
    -----------------------------------------------------------------------------------
    
    $ ansible-playbook --ask-become-pass test-changed_when2.yml
    BECOME password:

    PLAY [TEST changed_when] **************************************************************************

    TASK [Apache install] *****************************************************************************
    ok: [node03]
    ok: [node04]

    TASK [Start the Apache HTTPD Server] **************************************************************
    ok: [node04]
    ok: [node03]

    TASK [debug] **************************************************************************************
    ok: [node03] => {
        "msg": "httpd (pid 11717) already running"
    }
    ok: [node04] => {
        "msg": "httpd (pid 2973) already running"
    }

    PLAY RECAP ****************************************************************************************
    node03                     : ok=3    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
    node04                     : ok=3    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
    ```

<br>

## 3) failed_when

- **실패를 명확하게 구분할 수 있는 조건을 명시하여, 어떤 상황을 실패로 판단할지 정의한다.**

- 일반적으로 ansible에서 shell/command를 실행하면 정상인 경우 0을 반환하고, 나머지는 에러코드로 판단하여 실패로 간주한다.

- 하지만 실제 업무에서 활용할때는 0이 아닌 다른 코드도 정상으로 인식해야할 경우가 많은데, 이때 failed_when을 이용하여 실패의 조건을 명시하여 사용할 수 있다.

- ### **【 failed_when 사용하지 않을 경우 】**

    - 아래 task는 단순히 /tmp 디렉토리가 속한 디스크의 가용한 사이즈를 확인하는 것이다.

    - 아래 playbook을 실행하면 최종결과에 failed=0(성공)으로 표시된다.

    ```bash
    > vi test-failed_when.yml
    - name: Test failed_when
      hosts: localhost
      gather_facts: no
      tasks:
    - name: (STEP1) Making sure the /tmp has more than 1gb
      shell: "df -h /tmp|grep -v Filesystem|awk '{print $4}'|cut -d G -f1"

    > ansible-playbook test-failed_when.yml
    TASK [(STEP1) Making sure the /tmp has more than 1gb] *****************************************************************
    changed: [localhost]

    PLAY RECAP ************************************************************************************************************
    localhost                  : ok=1    changed=1    unreachable=0    failed=0
    ```

- ### **【 failed_when 조건 사용 】**

    - disk 공간과 가용 메모리를 확인하여, 최소사양 보다 적은 task를 실패하도록 정의해 보자

    - 아래 코드에서 STEP1은 1G 이상의 디스크 공간이 있어서 실패하지 않았고, STEP2는 정의한 공간보도 적어서 task가 실패하게 된다.

    - 아래의 코드를 실행하면 "failed_when_result: true"라는 키/벨류가 생성되고 최종결과에 failed=1로 표시된다.

    ```bash
    vi test-failed_when.yml
    - name: Test failed_when
      hosts: localhost
      gather_facts: no
      tasks:
    - name: (STEP1) Making sure the /tmp has more than 1gb
      shell: "df -h /tmp|grep -v Filesystem|awk '{print $4}'|cut -d G -f1"
      register: tmpspace
      failed_when: "tmpspace.stdout|float < 1"

    - name: (STEP2) Making sure the Physical Memory more than 2gb
      shell: "cat /proc/meminfo|grep -i memtotal|awk '{print $2/1024/1024}'"
      register: memory
      failed_when: "memory.stdout|float < 2"

    >ansible-playbook test-failed_when.yml
    TASK [(STEP1) Making sure the /tmp has more than 1gb] *****************************************************************
    changed: [localhost]

    TASK [(STEP2) Making sure the Physical Memory more than 2gb] **********************************************************
    fatal: [agent1]: FAILED! => {
        "changed": true, 
        "cmd": "cat /proc/meminfo|grep -i memtotal|awk '{print $2/1024/1024}'", 
        "delta": "0:00:00.007386", 
        "end": "2018-11-07 23:56:39.958154", 
        "failed": true, 
        "failed_when_result": true, 
        "rc": 0, 
        "start": "2018-11-07 23:56:39.950768", 
        "stderr": "", 
        "stderr_lines": [], 
        "stdout": "7.14656", 
        "stdout_lines": ["7.14656"]
    }
	to retry, use: --limit @/Users/skiper/work/DevTools/github/cs-install-automation/test-failed_when.retry

    PLAY RECAP ************************************************************************************************************
    agent1                     : ok=1    changed=1    unreachable=0    failed=1
    ```

<br>

## 4) ignore_error

- **ansibledms task를 실행하다 오류가 발생하면, 나머지 task를 실행하지 않고 중지한다.**

- **ignore_error를 이용하면 오류를 무시하고 다음 단계를 진행할 수 있다.**

- ### **【 ignore_errors: false (default) 】**

    - "Task Number1"에서 오류가 발생하면서 "Task Number2"로 진행도지 않고 task를 중지할 것이다.

    ```bash
    $ cat playbook_errorhandling.yml
    ---
    - name: Ansible Tutorial 12
      hosts: ROCKEY
      any_errors_fatal: true
      become: yes

      tasks:
      - name: Task Number1
        yum:
          name: treeee
          state: latest
        ignore_errors: false

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
    fatal: [node03]: FAILED! => {
        "changed": false, 
        "failures": ["No package treeee available."], 
        "msg": "Failed to install some of the specified packages", 
        "rc": 1, 
        "results": []
    }
    fatal: [node04]: FAILED! => {
        "changed": false, 
        "failures": ["No package treeee available."], 
        "msg": "Failed to install some of the specified packages", 
        "rc": 1, 
        "results": []
    }

    NO MORE HOSTS LEFT ********************************************************************************

    PLAY RECAP ****************************************************************************************
    node03                     : ok=4    changed=2    unreachable=0    failed=0    skipped=0    rescued=0    ignored=1
    node04                     : ok=4    changed=2    unreachable=0    failed=0    skipped=0    rescued=0    ignored=1
    ```

- ### **【 ignore_errors: true 】**

    - "Task Number1"에서 오류가 발생하다러도 "...ignoring"으로 표시되면서, "Task Number2"를 실행하는 것을 볼 수 있다.

    ```bash
    cat playbook_errorhandling.yml
    ---
    - name: Ansible Tutorial 12
      hosts: ROCKEY
      any_errors_fatal: true
      become: yes

      tasks:
      - name: Task Number1
        yum:
          name: treeee
          state: latest
        ignore_errors: true

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
    fatal: [node03]: FAILED! => {
        "changed": false, 
        "failures": ["No package treeee available."], 
        "msg": "Failed to install some of the specified packages", 
        "rc": 1, 
        "results": []
    }
    ...ignoring
    fatal: [node04]: FAILED! => {
        "changed": false, 
        "failures": ["No package treeee available."], 
        "msg": "Failed to install some of the specified packages", 
        "rc": 1, 
        "results": []
    }
    ...ignoring

    TASK [Task Number2] *******************************************************************************
    changed: [node03]
    changed: [node04]

    TASK [Task Number3] *******************************************************************************
    changed: [node03]
    changed: [node04]

    PLAY RECAP ****************************************************************************************
    node03                     : ok=4    changed=2    unreachable=0    failed=0    skipped=0    rescued=0    ignored=1
    node04                     : ok=4    changed=2    unreachable=0    failed=0    skipped=0    rescued=0    ignored=1
    ```