# 12. Interception and Error Control

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

- ### **【 changed_when 사용할 경우 】**

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

<br>

## 3) failed_when

- **실패를 명확하게 구분할 수 있는 조건을 명시하여, 어떤 상황을 실패로 판단할지 정의한다.**

- 일반적으로 ansible에서 shell/command를 실행하면 정상인 경우 0을 반환하고, 나머지는 에러코드로 판단하여 실패로 간주한다.

- 하지만 실제 업무에서 활용할때는 0이 아닌 다른 코드도 정상으로 인식해야할 경우가 많은데, 이때 failed_when을 이용하여 실패의 조건을 명시하여 사용할 수 있다.

- ### **【 failed_when 사용하지 않을 경우 】**

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

- ### **【 failed_when 사용 경우 】**
