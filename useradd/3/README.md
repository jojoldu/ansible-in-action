# 3. Ansible (앤서블) 로 전체 서버 계정 추가하기 - 플레이북으로 개선하기

이번 시간에는 앤서블의 플레이북 (Playbook) 을 통해 그동안 CLI를 하던 방식을 개선해보겠습니다.

## 3-1. 플레이북?

앤서블 공식 홈페이지의 [플레이북 소개](https://docs.ansible.com/ansible/latest/user_guide/playbooks.html)를 한번 보겠습니다.

> 플레이북은 앤서블의 설정/배포/오케스트레이션 (역주. 통합자동화/통합솔루션등) 언어 입니다.
호스트서버에서 시행할 정책이나 일반적인 시스템 프로세스의 단계를 설정할 수 있습니다.
만약 앤서블의 모듈이 작업장의 **도구**라면, 플레이북은 **지침서**, 호스트들의 인벤토리는 원재료입니다.

소개글을 살짝만 보더라도 플레이북이란 **앤서블의 명령을 모아놓은 설계서**와 같다는 것을 알 수 있습니다.  

플레이북은 쉘이나 DSL (Domain Specific Language, 특정 도메인에 특화된 언어) 과 같은 프로그래밍 언어로 구성되어있지 않습니다.  

> 다른 Infrastructure as code 도구들은 본인만의 DSL을 사용합니다.

프로그래밍 언어 대신 **YAML**을 사용합니다.  
스프링부트를 사용하시던 분들은 YAML에 친숙하실텐데, 처음 들어보신 분들이더라도 쉽게 배울 수 있습니다.  
YAML은 기본적으로 **기계가 파싱하기 쉽게, 사람이 다루기 쉽게**라는 컨셉으로 나온 기술이다보니 그 자체가 어렵지 않습니다.  
  
들여쓰기와 대시 (```-```) 로만 이루어져있기 때문에 러닝커브가 굉장히 낮습니다.  
  
이런 플레이북에 대한 소개는 글로는 한계가 있으니 바로 실습을 통해서 배워보겠습니다.

## 3-2. 플레이북 IDE

일단 플레이북을 편집하기에 좋은 도구가 필요합니다.  

* [](https://marketplace.visualstudio.com/items?itemName=vscoss.vscode-ansible)

## 3-3. 플레이북으로 개선



```yaml
---
- name: 사용자 추가
  hosts: all
  become: true
  tasks:
    - name: 계정 생성
      user:
        name: "{{ USER_NAME }}"
    - name: 패스워드 변경
      user:
        name: "{{ USER_NAME }}"
        password: "{{ PASSWORD | password_hash('sha512') }}"
    - name: sudoers.d 추가
      shell: "echo '{{ USER_NAME }}   ALL=(ALL)   NOPASSWD:ALL' > /etc/sudoers.d/{{ USER_NAME }}"
      validate: "/usr/sbin/visudo -c -f '%s'"
```

* 
```bash
ansible-playbook useradd.yml --extra-vars "USER_NAME=jojoldu2 PASSWORD=패스워드" 
```
