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
사내 저장소를 쓰거나 Github의 private 저장소를 쓴다면 Github으로 관리합니다.  
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
(아마 대부분이 ```jenkins```일겁니다.)  

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

```bash
echo "jenkins ALL=(ALL) NOPASSWD:ALL" > /etc/sudoers.d/jenkins
```

![5](./images/5.png)

```bash
visudo
```

아래와 같이 requiretty 항목이 있다면 주석 (```#```) 처리 합니다.

```bash
#Defaults requiretty # 주석
```

### 플러그인 설치

젠킨스에서는 
![1](./images/1.png)

![2](./images/2.png)

지금 다운로드하고 재시작 후 설치하기

![3](./images/3.png)

메인페이지로 돌아가기

![4](./images/4.png)

![5](./images/5.png)

재시작 합니다.  

### 설정

## 플레이북 실행

