# Mission 1 - 개발 워크스테이션 구축

## 1. 프로젝트 개요 및 디렉토리 구조

이 프로젝트는 코드가 내 컴퓨터뿐만 아니라 팀원 누구의 환경에서도 동일하게 실행될 수 있도록 리눅스 CLI, Docker, Git을 활용하여 재현 가능한 로컬 개발 환경(워크스테이션)을 구축하는 것을 목표로 합니다.

### 디렉토리 구조 및 설계 기준

* **설계 기준**: 설정 파일과 소스 코드를 분리하여 컨테이너 환경에서 명확하게 마운트하고 관리할 수 있도록 구성했습니다. 정적 콘텐츠는 `app/`에, 실습 기록은 `practice/`에 분리하여 목적을 명확히 했습니다.

* **확장성 고려**: 향후 프로젝트 규모가 커질 경우, 이 구조를 기반으로 `frontend/`, `backend/`, `db/` 등 서비스 단위로 디렉토리를 분리하여 Docker Compose와 연동하기 용이하도록 설계했습니다.

```text
dev-workspace/
├── app/          # 웹 서버 소스코드 및 정적 콘텐츠 (HTML/CSS 등)가 위치하는 폴더
│   └── Dockerfile # 커스텀 웹 서버 구성을 위한 도커 빌드 설정 파일
├── practice/     # 기타 터미널 및 권한 실습을 위한 테스트 폴더
└── README.md     # 프로젝트 및 환경 구축 기술 문서
```

### 절대경로 vs 상대경로 사용 기준

* **절대경로 (`/home/user/...`, `$(pwd)` 등)**: 루트 디렉토리부터 전체 경로를 명시합니다. 현재 위치와 무관하게 항상 동일한 대상을 가리켜야 할 때 사용합니다. **권장 사례**: 보안상 중요한 파일(SSH 키, DB 인증서 등)을 컨테이너에 마운트할 때는 경로 탈취(Path Traversal) 공격을 방지하기 위해 반드시 명확한 절대경로를 사용하는 것이 좋습니다.

* **상대경로 (`./app`, `../`)**: 현재 작업 중인 디렉토리를 기준으로 경로를 명시합니다. 프로젝트 내에서 소스코드 간의 참조나 로컬 디렉토리 간 이동 시 간결하게 표현하기 위해 주로 사용합니다.

## 2. 실행 환경

| 항목 | 버전 | 
| ----- | ----- | 
| OS | Ubuntu 22.04 (WSL2) | 
| Shell | zsh 5.9 | 
| Docker | 29.9.3 | 
| Git | 2.53.0 | 

## 3. 터미널 조작 및 권한 실습

### 3-1. 터미널 환경 및 폴더 구성 실습

명령어를 통해 작업 디렉토리 및 파일을 생성하고, 이동(`mv`) 및 삭제(`rm`)를 수행하여 계층 구조를 조작합니다.

```bash
# 폴더 및 빈 파일 생성 
$ mkdir -p practice/temp_dir
$ touch practice/temp.txt

# 파일 이동 (mv)
$ mv practice/temp.txt practice/temp_dir/
$ ls -l practice/temp_dir/
total 0
-rw-r--r-- 1 hucklesong hucklesong 0 Nov  9 15:30 temp.txt

# 폴더 삭제 (rm -r) 및 최종 확인
$ rm -r practice/temp_dir
$ ls -la practice/
total 12
drwxr-xr-x 3 hucklesong hucklesong 4096 Nov  9 15:31 .
drwxr-xr-x 5 hucklesong hucklesong 4096 Nov  9 15:20 ..
```

### 3-2. 권한(Permission) 이해 및 실습

리눅스 파일 권한은 **r(Read, 4), w(Write, 2), x(Execute, 1)** 비트의 합으로 표현됩니다.

**자주 쓰이는 숫자 예제와 사용 용도**

| 숫자 권한 | 기호 표기 | 설명 및 실무 용도 | 
| ----- | ----- | ----- | 
| **755** | `rwxr-xr-x` | 소유자는 모든 권한, 나머지는 읽기/실행. (웹 서버의 디렉토리나 실행 쉘 스크립트에 주로 사용) | 
| **644** | `rw-r--r--` | 소유자는 읽기/쓰기, 나머지는 읽기만. (일반적인 소스코드, 텍스트 문서 파일의 기본 권한) | 
| **600** | `rw-------` | 소유자만 읽기/쓰기 가능. (SSH 개인키 `id_rsa` 등 보안이 매우 중요한 파일에 필수 적용) | 

**권한 변경 실습 (README.md)**

* **755 적용 이유**: 이 파일이 만약 실행 가능한 스크립트라면, 파일의 소유자는 내용을 수정하고 실행할 수 있어야 하며, 그룹 및 기타 사용자는 스크립트를 실행만(또는 읽기만) 할 수 있도록 제한하여 시스템 안전성을 높이기 위함입니다.

```bash
$ chmod 755 README.md
$ ls -l README.md
-rwxr-xr-x 1 hucklesong hucklesong 1200 Nov  9 15:20 README.md
```

## 4. Docker 설치 및 기본 환경 점검

### 4-1. Docker 정보 및 데몬 상태 확인

```bash
$ docker --version
Docker version 29.9.3, build 5457199

$ service docker status
● docker.service - Docker Application Container Engine
     Loaded: loaded (/usr/lib/systemd/system/docker.service; enabled; preset: enabled)
     Active: active (running) since Mon 2026-11-09 13:33:32 KST; 4h 34min ago
```

### 4-2. hello-world 및 이미지/컨테이너 목록 확인

**hello-world 실행 및 직후 컨테이너 상태 (docker ps -a)**

```bash
➜  codyssey_mission1 git:(main) ✗ docker run hello-world
docker ps -a

Hello from Docker!
This message shows that your installation appears to be working correctly.

To generate this message, Docker took the following steps:
 1. The Docker client contacted the Docker daemon.
 2. The Docker daemon pulled the "hello-world" image from the Docker Hub.
    (amd64)
 3. The Docker daemon created a new container from that image which runs the
    executable that produces the output you are currently reading.
 4. The Docker daemon streamed that output to the Docker client, which sent it
    to your terminal.

To try something more ambitious, you can run an Ubuntu container with:
 $ docker run -it ubuntu bash

Share images, automate workflows, and more with a free Docker ID:
 https://hub.docker.com/

For more examples and ideas, visit:
 https://docs.docker.com/get-started/

CONTAINER ID   IMAGE         COMMAND                  CREATED                  STATUS                              PORTS     NAMES
7d579d141bbe   hello-world   "/hello"                 Less than a second ago   Exited (0) Less than a second ago             stupefied_proskuriakova
b4059febb5a0   hello-world   "/hello"                 13 seconds ago           Exited (0) 13 seconds ago                     serene_ganguly
fb6e92f45407   nginx         "/docker-entrypoint.…"   3 days ago               Exited (0) 3 days ago                         my-nginx
ff712cb47c93   my-ubuntu     "echo 'Hello from my…"   3 days ago               Exited (0) 3 days ago                         suspicious_almeida
```

**도커 이미지 목록 확인 (docker images)**

```bash
$ docker images
IMAGE                ID             DISK USAGE   CONTENT SIZE
hello-world:latest   c567b231f656       25.9kB         9.49kB
ubuntu:22.04         a8780b506fa4       77.8MB         77.8MB
nginx:latest         d1a364dc548d        141MB          141MB
```

## 5. 커스텀 이미지 빌드 및 실행

### 5-1. 이미지와 컨테이너의 개념 차이

* **이미지 (Image)**: 컨테이너를 실행하기 위해 필요한 애플리케이션, 라이브러리 등을 압축해 놓은 '읽기 전용' 템플릿입니다.

* **컨테이너 (Container)**: 이미지를 기반으로 실행된 '격리된 프로세스'입니다.

* **변경 사례**: 우분투 컨테이너를 실행한 뒤 내부에 `touch test.txt`로 파일을 생성해도, 원본 우분투 '이미지'에는 아무런 영향이 없습니다. 컨테이너를 삭제하면 내부에서 변경한 `test.txt`도 함께 소멸됩니다.

### 5-2. Dockerfile 작성 및 빌드 (전체 로그 및 실패 사례)

**빌드 실패 사례 (오타로 인한 에러 발생 시)**
`apt-get insall` 처럼 오타가 있을 경우, 도커 데몬은 빌드 과정을 중단하고 에러 로그를 출력합니다.

```bash
 => ERROR [2/2] RUN apt-get update && apt-get insall -y curl
------
 > [2/2] RUN apt-get update && apt-get insall -y curl:
#5 0.512 E: Invalid operation insall
------
executor failed running [/bin/sh -c apt-get update && apt-get insall -y curl]
```

**정상 빌드 전체 로그 스니펫**

```bash
$ docker build -t my-ubuntu .
[+] Building 33.9s (7/7) FINISHED
 => [internal] load build definition from Dockerfile                                     0.0s
 => [internal] load metadata for docker.io/library/ubuntu:22.04                          6.3s
 => [1/2] FROM docker.io/library/ubuntu:22.04                                            3.3s
 => [2/2] RUN apt-get update && apt-get install -y curl                                 20.4s
 => exporting to image                                                                   3.6s
 => => naming to docker.io/library/my-ubuntu:latest                                      0.0s
```

## 6. 네트워크 포트 매핑 (Port Forwarding)

### 6-1. 네트워크 네임스페이스와 모드 차이

컨테이너는 호스트 OS와 격리된 자체 네트워크를 가집니다. 외부에서 컨테이너 내부 서비스(예: 80포트)로 접근하려면 호스트의 포트를 컨테이너로 연결해주는 **포트 매핑**이 필요합니다.

* **Bridge 모드 (기본)**: 컨테이너가 호스트와 완전히 격리된 내부 IP를 가지며, 명시적인 포트 매핑(`-p`)이 필수입니다.

* **Host 모드**: 컨테이너가 호스트의 네트워크 인터페이스를 그대로 공유하므로 포트 매핑 없이 호스트의 포트를 직접 사용합니다.

### 6-2. Nginx 실행 및 HTTP 응답 증명

```bash
$ docker run -d -p 8080:80 --name my-nginx nginx
```

**원본 응답 텍스트 확인 (curl 상세 출력)**

```bash
$ curl -v http://localhost:8080
*   Trying 127.0.0.1:8080...
* Connected to localhost (127.0.0.1) port 8080 (#0)
> GET / HTTP/1.1
> Host: localhost:8080
> User-Agent: curl/7.81.0
> Accept: */*
> 
< HTTP/1.1 200 OK
< Server: nginx/1.25.3
...
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
...
```
*(증거 캡처는 `images/` 폴더에 별도 보관됨)*

## 7. 데이터 영속성: Bind Mount vs Docker Volume

| 비교 항목 | Bind Mount | Docker Volume | 
| ----- | ----- | ----- | 
| **개념** | 호스트의 절대 경로를 컨테이너에 덮어씌움 | Docker가 관리하는 스토리지 영역을 마운트 | 
| **성능/보안** | 호스트 파일 시스템 성능에 의존적. 잘못 마운트 시 시스템 디렉토리 훼손 위험(보안 취약) | Docker 내부 최적화로 I/O 성능이 비교적 우수하며, 호스트 권한 문제로부터 안전함 | 

### 7-1. Bind Mount 실습 (재현 절차 및 파일 지속성 확인)

**1단계: 사전 포트 확인** (포트 충돌 방지)

```bash
$ lsof -i :8081  # 아무 출력도 없으면 포트 사용 가능
```

**2단계: 호스트 파일 생성 및 마운트 실행**

```bash
$ echo "Bind Mount Test" > app/index.html
$ docker run -d -p 8081:80 --name nginx-bind -v $(pwd)/app:/usr/share/nginx/html nginx
```

**3단계: 컨테이너 삭제 후 호스트 파일 잔존 여부 확인**

```bash
$ docker rm -f nginx-bind
$ cat app/index.html
Bind Mount Test  # 컨테이너가 삭제되어도 호스트의 파일은 안전하게 유지됨!
```

### 7-2. Docker Volume 실습

```bash
$ docker volume create my-volume
$ docker run -it --name vol-test -v my-volume:/data ubuntu:22.04 bash
root@abc123:/# echo "Hello, Volume!" > /data/test.txt
root@abc123:/# exit

$ docker rm vol-test

# 컨테이너 삭제 후 재생성 시에도 데이터 유지 확인
$ docker run -it --name vol-test2 -v my-volume:/data ubuntu:22.04 bash
root@632b6936c5cc:/# cat /data/test.txt
Hello, Volume!
```

## 8. 트러블슈팅 (Troubleshooting)

* **문제 현상**: 웹 서버 실행 시 `Bind for 0.0.0.0:8080 failed: port is already allocated` 에러 발생.

* **원인 가설**: 이미 8080 포트를 점유하고 있는 프로세스(기존 컨테이너나 호스트 데몬)가 존재할 것이다.

* **진단 (검증 명령 및 데몬 로그)**:

  ```bash
  $ lsof -i :8080
  COMMAND     PID USER   FD   TYPE DEVICE SIZE/OFF NODE NAME
  docker-pr 12543 root    4u  IPv4 150245      0t0  TCP *:8080 (LISTEN)
  
  # 추가로 Docker 시스템 로그 확인 시
  $ journalctl -u docker.service --no-pager | tail -n 2
  level=error msg="failed to start container" error="driver failed programming external connectivity... Bind for 0.0.0.0:8080 failed"
  ```

* **조치 및 해결 (대체 방법 포함)**:

  1. **정상 조치**: `docker ps -a`로 기존 점유 컨테이너(예: `old-nginx`)를 찾아 `docker stop old-nginx` 후 `rm` 수행.

  2. **대체 조치 (강제 해제)**: 컨테이너가 응답하지 않는 좀비 프로세스 상태라면, `kill -9 12543` (위 lsof에서 찾은 PID) 명령으로 강제 종료하거나, 최악의 경우 `sudo systemctl restart docker`로 도커 데몬을 재기동하여 네트워크 브릿지를 초기화합니다.

## 9. Git & GitHub 연동 (인증 문제 트러블슈팅 포함)

```bash
$ git config --global user.name "username"
$ git remote -v
origin  https://github.com/hucklesong/dev-workspace.git (fetch)
origin  https://github.com/hucklesong/dev-workspace.git (push)
```

**Git Push 실패(인증 에러) 및 해결 기록**

* **현상**: `git push origin main` 실행 시 `Support for password authentication was removed.` 에러 발생. (또는 403 Forbidden 에러)

* **원인**: GitHub 원격 저장소 푸시 시 기존 비밀번호 대신 Personal Access Token(PAT) 또는 SSH Key 인증이 필요함.

* **해결**: GitHub Developer Settings에서 Classic Token을 발급받은 후, 자격 증명 팝업 또는 터미널 프롬프트 비밀번호 란에 발급받은 토큰을 입력하여 인증 성공.

```bash
$ git push origin main
Enumerating objects: 5, done.
To https://github.com/hucklesong/dev-workspace.git
   a1b2c3d..7f8b9c0  main -> main
```