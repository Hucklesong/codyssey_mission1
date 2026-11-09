# Mission 1 - 개발 워크스테이션 구축

## 1. 프로젝트 개요 및 디렉토리 구조

이 프로젝트는 코드가 내 컴퓨터뿐만 아니라 팀원 누구의 환경에서도 동일하게 실행될 수 있도록 리눅스 CLI, Docker, Git을 활용하여 재현 가능한 로컬 개발 환경(워크스테이션)을 구축하는 것을 목표로 합니다.

### 디렉토리 구조 및 설계 기준

* **설계 기준**: 설정 파일과 소스 코드를 분리하여 컨테이너 환경에서 명확하게 마운트하고 관리할 수 있도록 구성했습니다. 정적 콘텐츠는 `app/`에, 실습 기록은 `practice/`에 분리하여 목적을 명확히 했습니다.

```text
dev-workspace/
├── app/          # 웹 서버 소스코드 및 정적 콘텐츠 (HTML/CSS 등)가 위치하는 폴더
│   └── Dockerfile # 커스텀 웹 서버 구성을 위한 도커 빌드 설정 파일
├── practice/     # 기타 터미널 및 권한 실습을 위한 테스트 폴더
└── README.md     # 프로젝트 및 환경 구축 기술 문서
```

### 절대경로 vs 상대경로 사용 기준

* **절대경로 (`/home/user/...`, `$(pwd)` 등)**: 루트 디렉토리부터 전체 경로를 명시합니다. 현재 위치와 무관하게 항상 동일한 대상을 가리켜야 할 때(예: Docker 볼륨/바인드 마운트 설정 시) 사용합니다.
* **상대경로 (`./app`, `../`)**: 현재 작업 중인 디렉토리를 기준으로 경로를 명시합니다. 프로젝트 내에서 소스코드 간의 참조나 로컬 디렉토리 간 이동 시 간결하게 표현하기 위해 사용합니다.

---

## 2. 실행 환경

| 항목 | 버전 | 
| ----- | ----- | 
| OS | Ubuntu 22.04 (WSL2) | 
| Shell | zsh 5.9 | 
| Docker | 29.9.3 | 
| Git | 2.53.0 | 

---

## 3. 터미널 조작 및 권한 실습

### 3-1. 터미널 환경 및 폴더 구성 실습

명령어를 통해 작업 디렉토리 및 파일을 생성하고 계층 구조를 확인합니다.

```bash
# 폴더 및 빈 파일 생성 실습
$ mkdir -p practice/test_dir
$ touch practice/test.txt

# 목록 확인 (숨김 파일 포함)
$ ls -la practice/
total 12
drwxr-xr-x 3 hucklesong hucklesong 4096 Nov  9 15:20 .
drwxr-xr-x 5 hucklesong hucklesong 4096 Nov  9 15:20 ..
-rw-r--r-- 1 hucklesong hucklesong    0 Nov  9 15:20 test.txt
drwxr-xr-x 2 hucklesong hucklesong 4096 Nov  9 15:20 test_dir
```

### 3-2. 권한(Permission) 이해 및 실습

리눅스 파일 권한은 **r(Read, 4), w(Write, 2), x(Execute, 1)** 비트의 합으로 표현됩니다.

* **755의 의미**: 소유자(Owner)는 7(4+2+1, 읽기/쓰기/실행), 그룹(Group) 및 기타(Others)는 5(4+1, 읽기/실행) 권한을 가짐을 의미합니다. 스크립트 실행이나 디렉토리 접근이 필요한 경우 주로 부여합니다.

**파일 권한 변경 (README.md)**

```bash
# 변경 전
$ ls -l README.md
-rw-r--r-- 1 hucklesong hucklesong 1200 Nov  9 15:20 README.md

# 변경 후 (소유자에게 실행 권한 추가, 다른 사용자는 읽기/실행만 가능하도록 755 부여)
$ chmod 755 README.md
$ ls -l README.md
-rwxr-xr-x 1 hucklesong hucklesong 1200 Nov  9 15:20 README.md
```

**디렉토리 권한 변경 (app/)**

```bash
# 변경 전
$ ls -l
drwxr-xr-x 2 hucklesong hucklesong 4096 Nov  9 15:20 app
drwxr-xr-x 3 hucklesong hucklesong 4096 Nov  9 15:20 practice

# 변경 후 (소유자만 접근할 수 있도록 700 부여하여 보안 강화)
$ chmod 700 app/
$ ls -l
drwx------ 2 hucklesong hucklesong 4096 Nov  9 15:20 app
drwxr-xr-x 3 hucklesong hucklesong 4096 Nov  9 15:20 practice
```

---

## 4. Docker 설치 및 기본 환경 점검

### 4-1. Docker 정보 및 데몬 상태 확인

```bash
$ docker --version
Docker version 29.9.3, build 5457199

# Docker 데몬 동작 상태 확인 (서비스 단위)
$ service docker status
 * Docker is running
```

### 4-2. hello-world 및 이미지/컨테이너 목록 확인

**hello-world 실행**

```bash
$ docker run hello-world
Hello from Docker!
This message shows that your installation appears to be working correctly.
...
```

**컨테이너 전체 목록 (정지된 컨테이너 포함)**

```bash
$ docker ps -a
CONTAINER ID   IMAGE         COMMAND    CREATED         STATUS                     PORTS     NAMES
a1b2c3d4e5f6   hello-world   "/hello"   2 minutes ago   Exited (0) 2 minutes ago             magical_turing
```

---

## 5. 커스텀 이미지 빌드 및 실행

### 5-1. 이미지와 컨테이너의 개념 차이

* **이미지 (Image)**: 컨테이너를 실행하기 위해 필요한 애플리케이션, 라이브러리, 환경설정 등을 압축해 놓은 '읽기 전용' 템플릿입니다. 실행 환경의 설계도 역할을 하며 변경이 불가합니다.
* **컨테이너 (Container)**: 이미지를 기반으로 메모리에 적재되어 실행된 '격리된 프로세스'입니다. 컨테이너 내부에서 파일을 변경(쓰기)하더라도 원본 이미지는 변하지 않으며, 컨테이너가 삭제되면 내부에서 변경된 데이터도 함께 사라집니다(영속성 없음).

### 5-2. Dockerfile 작성 및 빌드

```dockerfile
# app/Dockerfile
FROM ubuntu:22.04
RUN apt-get update && apt-get install -y curl
CMD ["echo", "Hello from my container!"]
```

**빌드 명령어 설명**
* `-t my-ubuntu`: 빌드된 이미지에 `my-ubuntu:latest`라는 태그(이름)를 지정합니다.
* `.` : 빌드 컨텍스트 경로. 현재 디렉토리의 파일들을 Docker 데몬으로 전달하여 빌드를 수행합니다.

```bash
$ cd app
$ docker build -t my-ubuntu .
[+] Building 33.9s (7/7) FINISHED
 => [internal] load build definition from Dockerfile
...
 => => naming to docker.io/library/my-ubuntu:latest

$ docker run my-ubuntu
Hello from my container!
```

---

## 6. 네트워크 포트 매핑 (Port Forwarding)

### 6-1. 네트워크 경계와 포트 매핑의 필요성

컨테이너는 호스트 OS와 격리된 자체적인 네트워크 네임스페이스와 IP를 할당받습니다. 따라서 호스트 외부에서 컨테이너 내부의 서비스(예: 80포트의 웹서버)로 직접 통신할 수 없습니다. 이를 해결하기 위해 호스트의 특정 포트로 들어온 요청을 컨테이너의 내부 포트로 전달해주는 **포트 매핑(Port Forwarding)** 과정이 필요합니다.

### 6-2. Nginx 실행 및 HTTP 응답 증명

```bash
$ docker run -d -p 8080:80 --name my-nginx nginx
d1b2c3a4e5f6...
```

**원본 응답 텍스트 확인 (curl)**

```bash
$ curl http://localhost:8080
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
html { color-scheme: light dark; }
body { width: 35em; margin: 0 auto;
font-family: Tahoma, Verdana, Arial, sans-serif; }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>
...
</body>
</html>
```
응답 HTML 문서를 통해 포트 매핑 및 서비스 정상 동작 검증 완료 ✅

---

## 7. 데이터 영속성: Bind Mount vs Docker Volume

컨테이너가 삭제되어도 데이터를 안전하게 유지하기 위한 방법입니다.

| 비교 항목 | Bind Mount | Docker Volume | 
| ----- | ----- | ----- | 
| **개념** | 호스트의 특정 '절대 경로'를 컨테이너 경로에 직접 마운트 | Docker 데몬이 관리하는 별도의 스토리지 영역을 생성하여 마운트 | 
| **장점** | 호스트에서 파일 수정 시 즉각 컨테이너에 반영됨 (개발 환경에 적합) | OS에 종속적이지 않아 이식성이 높고 백업/복원 등 관리가 용이함 | 
| **단점** | 호스트 OS의 디렉토리 구조 및 권한에 강하게 의존함 | 호스트에서 직접 해당 파일을 찾아 수정하기 까다로움 | 

### 7-1. Bind Mount 실습

매핑을 위해 먼저 현재 경로(`pwd`)를 정확히 확인한 후 마운트합니다.

```bash
# 1) 현재 폴더 위치 확인
$ pwd
/home/hucklesong/dev-workspace

# 2) 호스트의 app 폴더를 컨테이너 nginx 정적 경로에 마운트
$ docker run -d -p 8081:80 --name nginx-bind \
    -v $(pwd)/app:/usr/share/nginx/html nginx
```

### 7-2. Docker Volume 실습

* **사용 볼륨 이름**: `my-volume`
* **마운트 경로**: `/data`
* **드라이버**: `local` (도커 기본 로컬 스토리지 드라이버)

```bash
$ docker volume create my-volume
my-volume

$ docker run -it --name my-volume-test -v my-volume:/data ubuntu:22.04 bash
root@abc123:/# echo "Hello, Volume!" > /data/test.txt
root@abc123:/# exit

$ docker rm my-volume-test
my-volume-test

# 새로운 컨테이너에 동일한 볼륨 연결하여 데이터 확인
$ docker run -it --name my-volume-test2 -v my-volume:/data ubuntu:22.04 bash
root@632b6936c5cc:/# cat /data/test.txt
Hello, Volume!
```
결과: 컨테이너 삭제 후 재생성 시에도 볼륨에 저장된 데이터는 완벽히 유지됨 검증 ✅

---

## 8. 트러블슈팅 (Troubleshooting)

실습 중 발생한 포트 충돌 문제와 해결 과정입니다.

* **문제 현상**: `docker run -p 8080:80` 명령어로 웹 서버 실행 시 `Bind for 0.0.0.0:8080 failed: port is already allocated` 에러 발생.
* **원인 가설**: 이미 8080 포트를 점유하고 있는 다른 프로세스나 이전 실습에서 중지되지 않은 컨테이너가 존재할 것이다.
* **진단 (검증 명령)**: `lsof` 커맨드로 8080 포트 사용 프로세스 확인
```bash
$ lsof -i :8080
COMMAND     PID       USER   FD   TYPE DEVICE SIZE/OFF NODE NAME
docker-pr 12543 hucklesong    4u  IPv4 150245      0t0  TCP *:8080 (LISTEN)
```
* **조치 및 해결**: 점유 중인 기존 컨테이너를 찾아 중지 및 삭제 후 재시도
```bash
$ docker ps -a | grep 8080
a2b3c4d5e6f7   nginx     "/docker-entrypoint.…"   2 hours ago   Up 2 hours    0.0.0.0:8080->80/tcp   old-nginx

$ docker stop old-nginx
old-nginx
$ docker rm old-nginx
old-nginx

# 이후 다시 실행 성공
$ docker run -d -p 8080:80 --name my-nginx nginx
```

---

## 9. Git & GitHub 연동

```bash
# Git 초기 설정
$ git config --global user.name "username"
$ git config --global user.email "user@email.com"

# 리모트 설정 확인
$ git remote -v
origin  https://github.com/hucklesong/dev-workspace.git (fetch)
origin  https://github.com/hucklesong/dev-workspace.git (push)

# 커밋 및 푸시 연동 증거
$ git add .
$ git commit -m "docs: 개발 워크스테이션 환경 구축 미션 완료"
[main 7f8b9c0] docs: 개발 워크스테이션 환경 구축 미션 완료
 1 file changed, 45 insertions(+), 12 deletions(-)

$ git push origin main
Enumerating objects: 5, done.
Counting objects: 100% (5/5), done.
Delta compression using up to 20 threads
Compressing objects: 100% (3/3), done.
Writing objects: 100% (3/3), 2.50 KiB | 2.50 MiB/s, done.
Total 3 (delta 1), reused 0 (delta 0)
To https://github.com/hucklesong/dev-workspace.git
   a1b2c3d..7f8b9c0  main -> main
```