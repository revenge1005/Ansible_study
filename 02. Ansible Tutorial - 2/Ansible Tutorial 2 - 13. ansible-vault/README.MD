# 13. ansible-vault

- **Ansible Vault는 패스워드, 키와 같은 보안에 민감한 파일들을 암호화해주는 기능**이다.

- Ansible에서 사용하는 모든 구조화된 데이터 파일을 암호화 할 수 있다.

- << ansible-vault >>

    |설명|명령|
    |:----:|-----------|
    |암호화된 파일을 생성하기|ansible-vault create foo.yml|
    |암호화된 파일을 수정하기|ansible-vault edite foo.yml|
    |암호화된 파일의 패스워드 변경하기|ansible-vault rekey foo.yml|
    |암호화되지 않은 파일을 암호화|ansible-vault encrypt foo.yml|
    |암호화된 파일의 복호화|ansible-vault decrypt foo.yml|
    |암호화된 파일의 내용 보기|ansible-vault view foo.yml|