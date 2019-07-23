# 3. Ansible (앤서블) 로 전체 서버 계정 추가하기 - 플레이북으로 개선하기

이번 시간에는 앤서블의 플레이북 (Playbook) 을 통해 그동안 CLI를 하던 방식을 개선해보겠습니다.

## 3-1. 플레이북?

앤서블 공식 홈페이지의 [플레이북 소개](https://docs.ansible.com/ansible/latest/user_guide/playbooks.html)를 한번 보겠습니다.

> 플레이북은 앤서블의 설정/배포/오케스트레이션 (역주. 통합자동화/통합솔루션등) 언어 입니다.


## 3-2. 플레이북으로 개선

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
