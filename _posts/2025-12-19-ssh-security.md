---
title: "SSH key로 보안 세팅"
date: 2025-12-19
last_modified_at: 2025-12-20
categories:
  - Server
tags:
  - Server
  - Ubuntu
  - SSH
sidebar:
  nav: "sidebar-category"
permalink: /server/ssh-security/
---

## 0. 이제는 보안을 챙길때

기나긴 포트포워딩 에러를 잡는 여정이 완료된 후, 나는 도메인을 새로 부여받았다.  
이제 도메인을 부여받았고, 이는 이제 보안상의 이슈가 생길 가능성이 생긴다는 뜻이다.  
기존 서버에 root나 계정 비밀번호를 치는 대신, ssh 키 방식으로만 서버를 접근하게 만들거다.

## 1. 서버 포트 변경

일단 첫번째로 변경해야 하는 건 기본 포트 번호 변경이였다.
너무나 알기 쉬운 기본포트 22번을 사용중이여서, 해킹을 시도하기 너무나 쉬웠다.
이젠 포트번호를 나만의 번호로 변경해보자.

    - ssh 포트 번호 변경
        1. 설정 수정: sudo nano /etc/ssh/sshd_config 실행.

        2. 포트 변경: #Port 22를 찾아서 주석(##)을 지우고 Port 22345처럼 10000번 이상의 숫자로 바꾼다.

        3. sudo ufw delete allow {포트번호} 로, 기존에 등록했던 포트 번호 삭제 후 sudo ufw allow {변경포트번호} 로 우분투 방화벽 규칙 등록

        4. 서비스 재시작: sudo systemctl restart ssh

        5. 공유기 설정: 공유기 포트포워딩 설정에서도 외부/내부 포트를 바뀐 번호로 수정.


    - 보안 조치 추가 - Fail2Ban 설치
        
        비밀번호를 게속 틀리며 무작위 공격을 하는 ip를 시스템이 자동으로 차단하는 프로그램

        1. sudo apt update && sudo apt install fail2ban -y

## 2. ssh 키 도입

포트를 변경 후, 제대로 접속되는걸 확인 한 후에 ssh 키를 도입하였다.
방법을 알아보기 전에 ssh란 무엇인가 알아보고 가자.

- SSH란?
    - Secure Socket Shell의 약자로, 다른 네트워크에서 동작하는 서버에 접근할 떄 보안성을 높이기 위해 사용되는 네트워크 프로토콜이다.
    - 비밀번호 대신 서버 접속 시 인증을 위해 사용하는 암호화된 키 쌍(private key/public key)이다.
    - private key로 암호화된 메세지를 public key로 복호화 하는 방식으로 작동한다.

패스워드 방식보다 보안수준이 더 높아 적용하는게 유리하다.  

이제 적용방법을 알아보자.  
    
    1. 내 pc에서 SSH 키 생성 : 터미널을 열어 명령어를 입력한다.  
        ```jsx
            ssh-keygen -t rsa -b 4096
            해당 키는 기본경로(~/.ssh/id_rsa)에 저장됩니다.  
            * 비밀번호는 선택사항입니다.
        ``` 
    2. 서버(우분투)로 공개키 복사
        - 자동복사 : 내 pc에서 서버로 자동으로 공개키를 복사해 준다.
        ```jsx
            ssh-copy-id -p [포트번호] [서버계정]@[도메인]
        ```
        만약 이 방법이 작동하지 않을 경우  
        - 수동복사 : 내 pc의 id_rsa.pub 내용을 복사 해 서버의 ~/.ssh/authorized_keys 파일 안에 그 내용을 붙여넣고 저장합니다.  

        이 작업 후에 파일 권한을 보안에 맞게 수정해 줘야한다.
        ```jsx
            chmod 600 ~/.ssh/authorized_keys
        ```

    3. SSH 키로 서버 접속 확인
        ```jsx
            # 기본 명령어
            ssh -p [포트번호] [서버계정]@[도메인]

            # 만약 키 파일이 여러 개라 특정 키를 지정해야 한다면
            ssh -i ~/.ssh/id_rsa -p [포트번호] [서버계정]@[도메인]
        ```
    
    4. SSH 보안 설정
        키 접속이 성공했다면, 이제 비밀번호를 입력하여 서버에 접속하는 방식을 아에 막아버릴거다.
        이렇게 하면 키파일을 해킹하지 않는 이상 절대로 들어올 수 없다.
        
        1. 서버 설정 파일을 연다.
        ```jsx
            sudo nano /etc/ssh/sshd_config
        ```

        2. 항목을 찾아 변경
            - PasswordAuthentication no (비밀번호 접속 금지)

            - PubkeyAuthentication yes (키 접속 허용)

            - PermitRootLogin no (root 계정 접속 금지 - 보안상 권장)

        3. SSH 재시작
        ```jsx
            sudo systemctl restart ssh
        ```

    **편하게 접속하는법**
        - 매번 주소를 입력하는 방식이 매우 귀찮다면 config를 설정해 해당하는 이름으로 바로 서버에 접속할 수 있다.

            1. PC의 ~/.ssh/ 폴더에 config라는 이름의 파일을 만듭니다 (**확장자 없음**).

            2. 메모장 등으로 열어 아래 내용을 넣습니다.
            ```Plaintext
                Host my-server
                HostName [도메인]
                User [서버계정명]
                Port [포트번호]
                IdentityFile ~/.ssh/id_rsa
            ```

            3. 이제 터미널에서 ssh my-server 라고만 치면 접속가능하다!

## 3. 후기

보안 설정을 마치고 터미널로 서버 컴퓨터에 접속하여 이것저것 만져봣다.
또한 vs코드에 extension을 깔아서 IDE 환경에서 직접 서버 컴퓨터의 설정을 만져도보니  
신기하고 재밌다. 그동안 막연하게 터미널 환경에 대해 두려움을 가지고 있었는데  
이번 기회에 오히려 흥미를 가지게 되는 게기가 되었다.