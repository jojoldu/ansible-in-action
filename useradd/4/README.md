# 4. Ansible (앤서블) 로 전체 서버 사용자 추가하기 - Jenkins&Github 연동하기

이번 시간에는 앤서블로 전체 서버 사용자 추가하기 시리즈의 마지막! Jenkins&Github로 관리하기 입니다.  
  
그간 리눅스 서버의 터미널에서만 관리하던 앤서블을 개선해보겠습니다.  
  
이번 시간에 앞서 진행되야할 것들이 있습니다.  
  
일단 앤서블 호스트 서버에 젠킨스가 설치 되어 있어야 하며, 해당 젠킨스는 작성중인 앤서블 플레이북 코드가 담긴 깃허브 저장소와 연동되어 있는 상태여야 합니다.  
  
안되어 있으신 분들은 아래 링크를 참고하여 진행해주시면 됩니다.

* [젠킨스 설치](https://jojoldu.tistory.com/441)
* [젠킨스와 깃허브 프로젝트 연동](https://jojoldu.tistory.com/442)

환경 설정이 다 되신분들은 아래 내용을 차례로 진행합니다.

## 1. Github 관리로 전환

먼저 플레이북을 Github으로 관리할 수 있도록 이관하겠습니다.  
여기서는 플레이북 파일과 인벤토리 파일의 관리 방법을 다르게 진행합니다.

### 플레이북

3장에서 사용한 플레이북 (site.yml)을 Github 프로젝트에 복사합니다.  

![1](./images/1.png)

코드 그대로 복사해서 만드시면 됩니다.  
꼭 저처럼 프로젝트 최상단에 위치하지 않아도 되며, 본인만의 하위 디렉토리를 만들어서 진행하셔도 무방합니다.

### 인벤토리

인벤토리 파일인 **hosts는 Github으로 관리하지 않겠습니다**.  
사내 저장소를 쓰거나 Github의 private 저장소를 쓴다면 Github으로 관리 합니다.  
다만 여기에서의 예제는 Github **public** 저장소를 기준으로 하기 때문에 **민감한 정보가 외부에 공개**될 수 있습니다.  
마찬가지로 이 예제를 진행하시는 분들도 정보 공개 위험이 있으니 안전하게 진행하겠습니다.  
  
인벤토리 파일은 **위치와 권한**을 수정합니다.  
  
먼저 파일을 다른 디렉토리로 복사하겠습니다.  
복사될 위치는 ```/var/lib/jenkins/ansible/useradd``` 입니다.  
이 위치는 **젠킨스가 실행되는 디렉토리** 입니다.  
결국 젠킨스가 앤서블을 실행해야하니, **젠킨스가 접근 가능한 디렉토리**에 관련된 파일들이 모두 위치하는게 좋습니다.  
  
해당 디렉토리는 현재 생성되어있지 않으니 차례로 생성합니다.

```bash
mkdir /var/lib/jenkins/ansible/
mkdir /var/lib/jenkins/ansible/useradd
```

그리고 hosts 파일을 복사합니다.

```bash
cp /root/ansible-useradd/hosts /var/lib/jenkins/ansible/useradd/
```

복사가 완료 되서 **디렉토리 이동 후** 파일을 확인해보면 권한이 **root**인것을 알 수 있습니다.

![2](./images/2.png)

젠킨스의 서비스 실행자를 확인합니다.  
(대부분은 ```jenkins``` 입니다.)  

![3](./images/3.png)

인벤토리 파일 역시 젠킨스 사용자가 실행할 수 있게 사용자 권한을 ```jenkins``` 로 변경합니다.  

```bash
chown jenkins:jenkins /var/lib/jenkins/ansible/useradd/hosts
```

사용자 권한도 변경이 잘 된 것을 확인할 수 있습니다.

![4](./images/4.png)

Github의 설정이 모두 끝났으니 젠킨스 설정을 진행해보겠습니다.

## 2. 젠킨스 설정


### 젠킨스 사용자 sudo 권한 추가

젠킨스가 결국 앤서블을 실행할 수 있어야 합니다.  
그럼 **비밀번호 입력 없이 sudo**를 실행할 수 있어야 합니다.  
아래와 같이 /etc/sudoers.d/jenkins 에 jenkins 사용자를 등록합니다.  

```bash
echo "jenkins ALL=(ALL) NOPASSWD:ALL" > /etc/sudoers.d/jenkins
```

![5](./images/5.png)

그리고 tty 없이 sudo를 실행할 수 있도록 /etc/sudoers 파일을 수정합니다.

> 참고: [tty는 무엇인가요?](https://kldp.org/node/155210)

visudo를 입력하시면 /etc/sudoers 파일을 안전하게 편집할 수 있습니다.  
(```visudo```만 입력하셔야 합니다.)

```bash
visudo
```

아래와 같이 requiretty 항목이 있다면 주석 (```#```) 처리 합니다.

```bash
#Defaults requiretty # 주석
```

설정이 다 되셨다면 젠킨스에서 앤서블 명령어를 실행할 수 있는 준비가 되었습니다.

### 플러그인 설치

젠킨스에서는 앤서블을 편하게 사용할 수 있도록 플러그인들을 제공합니다.  
해당 플러그인들을 설치하겠습니다.  
  
Jenkins 관리 -> 플러그인 관리로 이동합니다.

![6](./images/6.png)

설치 가능 항목에서 [Ansible](https://wiki.jenkins.io/display/JENKINS/Ansible+Plugin) 과 [AnsiColor](https://wiki.jenkins.io/display/JENKINS/AnsiColor+Plugin) 플러그인을 검색하여 체크합니다.

* Ansible
  * 앤서블 실행에 필요한 여러 파라미터를 개별적으로 지정할 수 있는 플러그인
* AnsiColor
  * 출력 결과를 컬러가 있는 채로 나타낼 수 있는 플러그인


![7](./images/7.png)

**지금 다운로드하고 재시작 후 설치하기** 를 클릭합니다.  
  
아래와 같이 플러그인 설치가 진행됩니다.

![8](./images/8.png)

설치가 끝나고 설치된 플러그인 목록에 위 2개 플러그인이 존재하는지 확인합니다.

![9](./images/9.png)

잘 설치되었다면 젠킨스 Job을 생성합니다.  
Freestyle project를 선택해서 개설합니다.

![10](./images/10.png)

매개변수에는 2개가 필요합니다.  
String Parameter, Password Parameter  

![11](./images/11.png)

앤서블 파일들이 있는 깃헙 저장소와 연동 합니다.  

![12](./images/12.png)

![13](./images/13.png)

Invoke Ansible Playbook 을 선택하여 아래와 같이 본인 환경에 맞게 등록합니다.

![14](./images/14.png)



### 설정

## 플레이북 실행

