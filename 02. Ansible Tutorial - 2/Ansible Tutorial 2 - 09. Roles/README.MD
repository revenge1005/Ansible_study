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
