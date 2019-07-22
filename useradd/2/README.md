# 2. Ansible (앤서블) 로 전체 서버 계정 추가하기 - CLI로 계정 추가하기

이번 시간엔 앤서블 CLI를 통해 각 호스트에 **루트 권한을 가진 계정을 추가**해보겠습니다.

## 2-1. user 모듈로 계정 추가

ansible에선 ```user```라는 모듈이 계정 추가/삭제 기능을 지원하고 있습니다.  
이 모듈을 사용해서 다음과 같이 명령어를 수행하면 계정이 생성됩니다.  
(**앤서블 서버에선 root계정 상태**입니다.)  
  
```bash
ansible all -m user -a "name=jojoldu" -u ec2-user
```

명령어를 수행하시면 아래와 같이 **권한 거부** 에러가 발생합니다.

![1](./images/1.png)

이는 현재 호스트 접근 계정인 ec2-user에 ```sudo``` 권한이 없기 때문입니다.  
이를 해결하기 위해 ec2-user가 아닌 **root 계정으로 접근하기엔 부담**스럽습니다.  
  
그래서 명령어를 수행할때마다 ```sudo``` 가 함께 수행될 수 있게 설정값을 추가해보겠습니다.  
  
아래 명령어로 ```ansible.cfg``` 을 열어서

```bash
sudo vim /etc/ansible/ansible.cfg
```

아래와 같이 ```[privilege_escalation]``` 항목을 설정합니다.

```bash
[privilege_escalation]
become=True
become_method=sudo
become_user=root
#become_ask_pass=False
```

![2](./images/2.png)

* ansible.cfg은 앤서블의 설정 파일입니다.
* /etc/ansible/ansible.cfg 은 글로벌 설정 파일입니다.
  * 우선 순위가 가장 낮습니다.
  * 이보다 높은 우선 순위는 현재 디렉토리 (```ansible.cfg```), 현재 사용자의 홈 디렉토리 (```~/.ansible.cfg```) 이 있습니다.
  * 글로벌 설정을 피하고 싶다면 위 설정 위치를 사용하시면 됩니다.

설정이 다 되셨다면 다시 명령어를 수행해봅니다.

![3](./images/3.png)

정상적으로 계정이 생성된 것을 확인할 수 있습니다!  
  
실제로 각 서버에 계정이 추가되었는지 확인도 ansible로 해봅니다.

```bash
ansible all -m shell -a "tail -n 1 /etc/passwd" -u ec2-user
```

그럼 다음과 같이 home 디렉토리에 계정이 추가된 것을 확인할 수 있습니다.

![4](./images/4.png)


## 2-2. 임의 추가된 계정 삭제하기

다음으로 방금 추가한 계정을 삭제해보겠습니다.  
  
삭제는 간단합니다.  
기존 계정 생성 명령어에 ```state=absent remove=yes``` 만 추가하면 됩니다.

```bash
ansible all -m user -a "name=jojoldu state=absent remove=yes" -u ec2-user
```

![5](./images/5.png)

다시 삭제되었는지 확인 해봅니다.

![6](./images/6.png)

더이상 마지막 계정 표기에 jojoldu가 아닌 ```ec2-user```만 노출된다면 정상적으로 삭제 된 것입니다.  
  
테스트로 추가된 계정은 이렇게 삭제하면 되겠죠?

## 2-3. 패스워드 추가하기

자 그럼 계정을 생성하고 해당 계정에 비밀번호를 추가해보겠습니다.  
(기존 계정의 비밀번호 변경도 같은 방법으로 가능합니다.)  
  
먼저 파이썬의 암호화 모듈이 필요합니다.  



```bash
ansible all -m user -a "name=jojoldu update_password=always password={{ '변경하고싶은 비밀번호' | password_hash('sha512') }}" -k --user=dwlee
```


```bash
echo 'jojoldu   ALL=(ALL)   NOPASSWD:ALL' > /etc/sudoers.d/jojoldu
```

```bash
ansible all -m shell -a "echo '$USER_NAME   ALL=(ALL)   NOPASSWD:ALL' > /etc/sudoers.d/$USER_NAME" -e "ansible_user=dwlee ansible_ssh_pass=비밀번호"
```

```bash
[web]
호스트1
호스트2

[db]
호스트3

[all:vars]
ansible_user=접속계정
ansible_ssh_pass=접속비밀번호
```

## 2-5. sudoers.d 추가
