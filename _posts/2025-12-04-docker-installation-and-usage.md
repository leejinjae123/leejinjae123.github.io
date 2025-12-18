---
title: "Docker 입문기"
date: 2025-12-05
last_modified_at: 2025-12-05
categories:
  - Server
tags:
  - Docker
  - Ubuntu
sidebar:
  nav: "sidebar-category"
---

## 0. 도커를 선택한 이유

이때까지 도커를 사용해본 적이 없었지만, 항상 개발 환경 세팅에 대한 이슈는 있었다.
이전 직장들에서는 도커를 사용하지않고 패키징을 해서 다녔는데, 개인적으로 매우 비효율적이라고 느꼇다.
이번 내 사이드 프로젝트는 안쓰는 노트북으로 서버를 만들어 개발서버로 사용하고, AWS로는 배포 서버를 만들어 CI/CD와 서버세팅을 딥하게 공부해 보려고 한다. 
무엇을 해볼까 고민하던 찰나 내 눈에 들어온것이 바로 DOCKER였다.

## 1. 도커란 무엇인가

![도커1](/assets/png/도커1.png)

도커란 컨테이너 생성 및 관리 도구이다.
이게 무슨소리일까? 설명을 위해서는 컨테이너의 개념을 먼저 알아야 한다.

```jsx
   컨테이너란?
   - 하나의 운영체제 내에서 여러개의 격리된 사용자 공간을 생성하여 애플리케이션을 격리된 사용자 공간에 배치 & 실행하는 기술이다.
```
기존에 사용되던 VM(Virtual Machine)의 경우, 독립된 게스트 OS를 지니는 만큼 보안성이 좋지만 무겁다는 단점이 있다.  
이 단점을 보완하기 위하여 나온 기술이 컨테이너이다.  
VM보다 자원 낭비가 적고 빠른 속도를 가지고있다. 또한, 설정 파일이나 환경이 포함되어 관리되기 때문에 일관성을 유지하기 쉽다.  
물론 도커 이전에도 리눅스 컨테이너의 개념이 존재했었지만, 사용의 불편함과 세팅이 난이도로 인하여 사용화 되지 못했습니다.  
이 리눅스 컨테이너의 단점을 보완하여 나온것이 Docker 되시겠다.  

Docker의 장점을 뽑아보자면
  1. 직관적이고 쉬운 생성,배포 방법
  2. 쉬운 이미지 생성 및 DockerHub를 통해 공유
  3. Docker compose를 통하여 여러가지 어플리케이션을 관리할 수 있다
  4. 독립 OS를 지닌것이 아닌 호스트 OS의 자원을 공유해 라이트하다
  5. 한번 만들어놓은 Docker는 어느 환경에서도 동일하게 작동한다
  6. 독립성을 가지고있어 서로 영향을 주지 않는다.

밑의 이미지는 도커 아키텍쳐이다.  

![도커-구조](/assets/png/도커-구조.png)

도커 아키텍쳐를 설명하자면
1. Docker Client  =>  도커를 설치하면 그것이 Client이며 build, pull, run 등의 도커 명령어를 수행합니다.
2. Docker_HOST  =>  도커가 띄어져 있는 서버를 의미합니다. DOCKER_HOST에서 컨테이너와 이미지를 관리하게 됩니다.
3. Docker daemon  =>  도커 엔진의 핵심 구성 요소 중 하나로 도커 호스트에서 실행되는 백그라운드 프로세스입니다.
4. Registry  =>  외부(remote) 이미지 저장소로 도커 이미지를 저장하고 공유하는 중앙 저장소입니다. 다른 사람들이 공유한 이미지를 내부(local) 도커 호스트에 pull 할 수 있습니다. 이렇게 가져온 이미지를 run하면 컨테이너가 됩니다.
5. public 저장소: Docker Hub, QUAY
6. private 저장소: AWS ECR 혹은 Docker Registry를 직접 띄워서 비공개로 사용하는 방법 등이 존재합니다.

- Docker Iamge & Container
  - 도커의 기본 단위중 하나이며, 도커 엔진의 핵심이다. 도커 이미지와 컨테이너는 1:N 관계이다.
  - Docker File => Docker Image => Docker Container 순서로 생성된다.
  - docker build 명령어로 도커 이미지를 만들며, Docker File를 기반으로 만든다.
  - Docker Image
    도커 이미지는 컨테이너를 어떻게 만들어야 하는지에 대한 지침으로 애플리케이션 코드와 실행에 필요한 모든 구성 요소를 포함합니다.
    가상 머신을 생성할 때 사용하는 iso 파일과 비슷한 개념입니다.
  - Docker Container
    도커 컨테이너는 도커 이미지로 생성할 수 있습니다.
    애플리케이션과 그것을 실행하는 데 필요한 모든 것(라이브러리, 설정 파일 등)을 포함하는 독립된 실행환경입니다.
    컨테이너를 생성하면 해당 이미지의 목적에 맞는 파일이 들어 있는 호스트와 다른 컨테이너로부터 격리된 시스템 자원 및 네트워크를 사용할 수 있는 독립된 공간(프로세스)이 생성됩니다.

```jsx
  Docker Compose?
  - 단일 서버에서 여러개의 컨테이너를 하나의 서비스로 정의해 묶음으로 관리하는 관리도구이다.
  - kafka 같은 MSA 환경 구성을 매우 편하게 세팅 할 수 있게 해준다.
```

## 2. 도커를 설치해보자

나는 윈도우 환경과 우분투 환경에서 동시에 도커 설치를 해야했다.
도커는 가급적 리눅스로 설치할 것을 권장한다. 만약 윈도우나 맥에서는 가상머신을 설치한 후에 Docker를 설치하기 때문에 성능에서 손해를 본다.
그래도 docker desktop 이라는 프로그램을 지원해서 편하게 설치는 가능하다.
docker desktop은 '소규모 기업' 기준을 초과하는 경우이다. 
구체적으로는 직원 250명 미만이고 연간 매출 1,000만 달러 미만인 기업이 아니면 docker Desktop 사용을 위해 유료 구독이 필요하지만,
지금 단계에서는 고려할 사항이 아니였다.
일단은 편하게 설치해보도록 하자

- 윈도우 환경에서 도커 설치

  딱히 어려울만한 일이 없다.

  - 1. Docker Desktop 설치 URL: [https://docs.docker.com/get-started/get-docker/](https://docs.docker.com/get-started/get-docker/)
  - 2. 설치 후 cmd를 열어 docker을 입력하여 정상설치 여부 확인

- 우분투 환경에서 도커 설치

  버전관리에 신경을 써야한다.
  만약 기존에 도커를 설치한 이력이 있다면, 제거를 해줘야한다.
  순서대로 진행해주자.
  ```jsx
    삭제 명령어 : sudo apt-get remove docker docker-engine docker.io containerd runc
  ```
  - 패키지 업데이트
    ```jsx
      sudo apt-get update
    ```
  - 필수 패키지 설치
    ```jsx
      sudo apt-get install apt-transport-https ca-certificates curl software-properties-common
    ```
  - Docker 공식 GPG 키 추가
    GPG 키는 공식 저장소에서 제공하는 무결성 검증 디지털 서명 키다. 등록하지 않으면 Docker을 사용할 수 없다.
    ```jsx
      curl -fsSL [https://download.docker.com/linux/ubuntu/gpg](https://download.docker.com/linux/ubuntu/gpg) | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
    ```
  - Docker 저장소 추가
    ```jsx
      echo "deb \[arch=amd64 signed-by=/usr/share/keyrings/docker-archive-keyring.gpg\] [https://download.docker.com/linux/ubuntu](https://download.docker.com/linux/ubuntu) $(lsb\_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
    ```
  - 패키지 업데이트를 다시 진행  

  - Docker 관련 패키지 설치
    ```jsx
      sudo apt-get install docker-ce docker-ce-cli containerd.io
    ```
  - 설치 확인
    ```jsx
      sudo docker --version
    ```
- 번외) 도커 컴포즈 설치

  - 도커 컴포즈 설치
    다른 버전을 쓰고싶다면 가운데 v2.5.0을 변경하면 된다.
    ```jsx
      sudo curl -L "https://github.com/docker/compose/releases/download/v2.5.0/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
    ```
  - 도커 컴포즈 권한 부여
    ```jsx
      sudo chmod +x /usr/local/bin/docker-compose
    ```
  - 심볼릭 링크 연결
    ```jsx
      sudo ln -s /usr/local/bin/docker-compose /usr/bin/docker-compose
    ```
  - 설치 확인
    ```jsx
      docker-compose --version (v1버전)
      docker compose --version (v2버전)
    ```

## 3. Dockerfile 및 Docer-Compose File 생성

Dockerfile이란 docker 이미지를 생성하기 위한 **스크립트 파일**이다.
Dockerfile의 특징으로는
  - 재사용성 : 어느 환경에서도 동일한 환경을 재생성할 수 있다. <--- 내가 docker를 사용해 보고 싶었던 가장 큰 이유였다.
  - 자동화 : 명령어를 미리 입력해, 이미지를 자동으로 생성 할 수 있다.
  - 버전 관리 : 빌드 프로세스를 추적 할 수 있다.

Dockerfile 생성을 위해서는, 프로젝트 폴더에 dockerfile이 존재해야한다.  
현재 내가 프로젝트에서 사용하고 있는 dockerfile을 예시로 가져와보겠다.

```jsx
# Stage 1: 애플리케이션 빌드 스테이지
# Maven과 JDK 17을 사용하여 소스 코드를 빌드하고 JAR 파일을 생성합니다.
FROM maven:3.8-openjdk-17 AS build
WORKDIR /app
COPY . .

# Maven 빌드 시 메모리 부족(OOM) 방지를 위한 최대 힙 메모리 설정
ENV MAVEN_OPTS="-Xmx1024m"

# 테스트 코드를 제외하고 패키징 수행하여 빌드 속도 최적화
RUN mvn clean package -Dmaven.test.skip=true

# Stage 2: 실행 전용 스테이지
# 경량화된 JRE 환경(Alpine Linux)에서 빌드된 결과물만 실행합니다.
FROM eclipse-temurin:17-jdk-alpine
WORKDIR /app

# 빌드 스테이지에서 생성된 JAR 파일만 복사하여 이미지 크기를 최소화
COPY --from=build /app/target/*.jar app.jar

# 애플리케이션 실행 명령
ENTRYPOINT ["java", "-jar", "app.jar"]
```
Stage 1 에서는 소스를 컴파일하고, 메모리 부족을 예방하기 위해 최대메모리 제한을 걸어놧다.  
그리고 Stage 2 에서는 빌드 결과물만 가지고 와 경량화한 환경을 구축했다.  
이런 방식을 멀티 스테이지 빌드라고 한다.
이런식으로 미리 빌드를 하는 과정을 적어놓아 이미지를 생성하는 방법을 설명한 파일이 dockerfile이다.  

이제 Dockerfile의 명령어를 소개한다.  

|FROM|이미지 지정|
|---|---|
|RUN|명령어 실행|
|COPY/ADD|파일을 이미지에 복사|
|CMD|컨테이너 시작시 실행할 명령어(1개만)|
|ENTRYPOINT|컨테이너 시작시 실행할 명령어(CMD와 함께 사용)|
|WORKDIR|작업 디렉토리 지정|
|ENV|환경 변수 세팅|
|EXPOSE|컨테이너에서 노출할 포트|

dockerfile은 여기까지 알아보고, 이젠 docker compose에 대해서 알아보자.  

- Docker Compose란?  
  ```jsx
    - 여러 컨테이너를 한번에 실행할 수 있는 선언적 구성 도구이다.
    - 서비스 간의 의존성, 웹 서버와 db, 캐시등을 통합 관리할 수 있다.
  ```
처음 나의 사이드 프로젝트를 구상할때, 막연히 도커를 사용해봐야겠다! 라고 생각하고 맨땅에 헤딩하듯이 공부했다.
dockerfile까지 알아봣을때, 그러면 이미지를 일일히 구현해서 서버에 올려야 하는건가? 라는 생각을했다.
스크립트를 활용해서 따로 선언문을 작성하는게 손이 덜 가겠다라고 생각을 하던 와중 docker compose 라는 개념을 공부했다.
역시 똑똑한 사람들은 이미 내가 했던 생각을 다 먼저 생각했던 것이 틀림이 없다.
밑의 예시는 나의 docker compose 중 일부분을 발췌해서 가지고 왔다.  

  ```jsx
    services:
      mysql:
        image: mysql:8.0
        container_name: mysql-db
        environment:
          MYSQL_ROOT_PASSWORD: mysql 비밀번호
          MYSQL_DATABASE: mysql 데이터베이스
          MYSQL_USER: mysql 유저
          MYSQL_PASSWORD: mysql 비밀번호
          MYSQL_ROOT_HOST: '%' # 모든 IP에서 접속 허용 (로컬 개발 시 중요)

        ports:
          - "3306:3306" # 로컬 개발용 외부 접속 허용
        volumes: # docker로 db를 구성시, 이미지를 내렸을 때 데이터가 사라지는 것을 막기위해 밑의 경로에 db 데이터를 남겨놓는 방법
          - ./mysql-data:/var/lib/mysql
        networks:
          - fit-network
        command: --character-set-server=utf8mb4 --collation-server=utf8mb4_unicode_ci
        healthcheck:
          test: ["CMD", "mysqladmin", "ping", "-h", "localhost"]
          interval: 10s
          timeout: 5s
          retries: 5

      redis:
        image: redis:alpine
        container_name: redis
        ports:
          - "6379:6379" # 로컬 개발용 외부 접속 허용
        networks:
          - fit-network
  ```

현재 나의 프로젝트는 개발중이고, 편의를 위해서 모든 ip에서 접속이 가능하게 만들어놧지만, 추후에는 내가 지정하는 ip 내부망으로만 통신 될 수 있게 수정할거다.
이런식으로 핵심 섹션 (services) 안에 여러 컨테이너 정보를 담을 수 있다.

- Docker compose 명령어

```jsx
  1. docker compose build : 도커 이미지를 빌드한다. 
    * docker compose build --no-cache  
      기존 남아있는 캐시를 지우고 완전 처음부터 설치 시작한다.  
      만약 빌드중 이전 버전의 정보가 남아있어 충돌이 나는 경우, 캐시를 지우고 실행해보자.  
  2. docker compose up -d : 서비스를 시작한다. 뒤의 -d는 해당 콘솔이 종료되도 백그라운드에서 실행할 수 있게 해주는 명령어다.
  3. docker compose down : 컨테이너를 모두 중지하고 삭제한다.
    * docker compose down -v  
      down만 사용하면 볼륨은 남기고 삭제하므로, 볼륨까지 삭제하고 싶다면 위 명령어를 사용한다.
  4. docker compose ps : 컨테이너의 현제 상태를 표시해준다
  5. docker compose logs -f : 실시간 로그를 모니터링 해준다.
```
이외에도 많은 명령어들이 있으나, 아직 사용해본 명령어만 정리하도록 하겠다.

## 4. 마치며
처음 공식문서를 뒤지고, 여러 매체를 통해 공부를 해도 무슨 개념인지 파악하기 힘들었는데  
역시 실습을 하면서 공부를 하니 이해가 잘 되었다. 
추후에는 docker 서버를 스케일링 하고, 최적화 하는 방법을 더 공부 해 봐야겠다.