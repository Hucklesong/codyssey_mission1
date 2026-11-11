# Mission 1 - 개발 워크스테이션 구축

## 1. 프로젝트 개요 및 디렉토리 구조

이 프로젝트는 코드가 내 컴퓨터뿐만 아니라 팀원 누구의 환경에서도 동일하게 실행될 수 있도록 리눅스 CLI, Docker, Git을 활용하여 재현 가능한 로컬 개발 환경(워크스테이션)을 구축하는 것을 목표로 합니다.

### 디렉토리 구조 및 설계 기준

* **설계 기준**: 설정 파일과 소스 코드를 분리하여 컨테이너 환경에서 명확하게 마운트하고 관리할 수 있도록 구성했습니다. 정적 콘텐츠는 `app/`에, 실습 기록은 `practice/`에 분리하여 목적을 명확히 했습니다.

* **확장성 고려**: 향후 프로젝트 규모가 커질 경우, 이 구조를 기반으로 `frontend/`, `backend/`, `db/` 등 서비스 단위로 디렉토리를 분리하여 Docker Compose와 연동하기 용이하도록 설계했습니다.

```text
codyssey_mission1/
├── app/          # 웹 서버 소스코드 및 정적 콘텐츠 (HTML/CSS 등)가 위치하는 폴더
│   └── Dockerfile # 커스텀 웹 서버 구성을 위한 도커 빌드 설정 파일
├── practice/     # 기타 터미널 및 권한 실습을 위한 테스트 폴더
└── README.md     # 프로젝트 및 환경 구축 기술 문서
```

### 절대경로 vs 상대경로 사용 기준

경로(Path)란 파일 시스템에서 특정 파일이나 디렉토리의 위치를 나타내는 주소입니다. 경로를 표기하는 방식은 두 가지입니다.

* **절대경로 (`/home/user/...`, `$(pwd)` 등)**: 파일 시스템의 최상단인 루트 디렉토리(`/`)에서 시작하여 목적지까지의 전체 경로를 명시합니다. 현재 내가 어느 디렉토리에 있는지와 관계없이 항상 동일한 대상을 가리킵니다. **권장 사례**: 보안상 중요한 파일(SSH 키, DB 인증서 등)을 컨테이너에 마운트할 때는 경로 탈취(Path Traversal) 공격을 방지하기 위해 반드시 명확한 절대경로를 사용하는 것이 좋습니다.

* **상대경로 (`./app`, `../`)**: 현재 작업 중인 디렉토리(`pwd`)를 기준으로 경로를 명시합니다. 프로젝트 내에서 소스코드 간의 참조나 로컬 디렉토리 간 이동 시 간결하게 표현하기 위해 주로 사용합니다.

**예시로 비교하기**

아래와 같은 디렉토리 구조가 있다고 가정합니다.

```text
/home/hucklesong/           ← 루트(/) 아래 home, 그 아래 hucklesong
└── codyssey_mission1/
    ├── app/
    │   └── index.html      ← 이 파일에 접근하려는 상황
    └── practice/
```

현재 위치가 `/home/hucklesong/codyssey_mission1/practice/` 일 때:

| 경로 종류 | 표기 예시 | 설명 |
| ----- | ----- | ----- |
| **절대경로** | `/home/hucklesong/codyssey_mission1/app/index.html` | 루트(`/`)에서 시작, 위치 불문하고 항상 유효 |
| **상대경로** | `../app/index.html` | 현재 위치(`practice/`)에서 한 단계 위(`..`)로 올라가 `app/`으로 진입 |

> `..`은 부모 디렉토리를, `.`는 현재 디렉토리를 의미합니다. 현재 위치가 바뀌면 상대경로도 의미가 달라지므로, 스크립트나 자동화 도구에서는 절대경로를 선호합니다.

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

리눅스는 모든 파일과 디렉토리에 대해 **누가, 무엇을 할 수 있는지**를 세 가지 대상과 세 가지 행위로 구분하여 관리합니다.

**권한의 세 가지 대상**

| 대상 | 기호 | 설명 |
| ----- | ----- | ----- |
| 소유자 (Owner) | `u` | 파일을 만든 사용자 |
| 그룹 (Group) | `g` | 소유자가 속한 그룹의 다른 사용자들 |
| 기타 (Others) | `o` | 소유자도 그룹도 아닌 모든 사용자 |

**권한의 세 가지 종류와 숫자 값**

| 권한 | 기호 | 숫자 값 | 의미 |
| ----- | ----- | ----- | ----- |
| 읽기 | `r` | **4** | 파일 내용 읽기, 디렉토리 목록 조회 |
| 쓰기 | `w` | **2** | 파일 내용 수정, 디렉토리 내 파일 생성·삭제 |
| 실행 | `x` | **1** | 파일 실행(스크립트·바이너리), 디렉토리 진입(`cd`) |

**숫자 권한 해석 방법**

`ls -l` 출력에서 권한은 `rwxr-xr-x`처럼 9글자로 표시되며, 앞 3자리는 소유자, 중간 3자리는 그룹, 마지막 3자리는 기타 사용자의 권한입니다. 각 자리의 숫자는 해당 대상이 가진 권한 값의 합입니다.

```text
  755  →  7   5   5
         ↓   ↓   ↓
        rwx r-x r-x
        소유자 그룹 기타

  7 = r(4) + w(2) + x(1)  →  읽기 + 쓰기 + 실행
  5 = r(4) + w(0) + x(1)  →  읽기 + 실행만 (쓰기 없음)
```

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

### 5-2. Dockerfile 작성 및 빌드

**Dockerfile이란?**

Dockerfile은 커스텀 이미지를 만들기 위한 설계도(레시피)입니다. `docker build` 명령을 실행하면 Docker는 Dockerfile의 명령어를 위에서 아래로 순서대로 실행하며, 각 명령어마다 새로운 레이어(Layer)를 쌓아 최종 이미지를 완성합니다. 레이어는 캐싱되므로, 변경된 명령어 이후만 다시 빌드되어 반복 빌드 시 속도가 빨라집니다.

**주요 Dockerfile 명령어**

| 명령어 | 설명 | 예시 |
| ----- | ----- | ----- |
| `FROM` | 기반이 될 베이스 이미지를 지정. 모든 Dockerfile의 첫 번째 명령어 | `FROM ubuntu:22.04` |
| `RUN` | 이미지 빌드 중 셸 명령을 실행. 패키지 설치 등에 사용 | `RUN apt-get install -y curl` |
| `COPY` | 호스트의 파일/디렉토리를 이미지 안으로 복사 | `COPY ./app /usr/share/nginx/html` |
| `EXPOSE` | 컨테이너가 사용할 포트를 문서화 (실제 개방은 `-p` 옵션으로) | `EXPOSE 80` |
| `CMD` | 컨테이너가 시작될 때 기본으로 실행될 명령어 | `CMD ["nginx", "-g", "daemon off;"]` |

**커스텀 이미지를 만드는 흐름**

```text
Dockerfile 작성  →  docker build  →  커스텀 이미지 생성  →  docker run  →  컨테이너 실행
  (설계도)            (공장)              (완성품 틀)                          (실제 인스턴스)
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

### 6-1. 포트 매핑이 필요한 이유

컨테이너는 호스트 OS와 완전히 격리된 자체 네트워크를 가집니다. 즉, 컨테이너 내부의 Nginx가 80번 포트에서 대기하고 있어도, 외부(브라우저, curl 등)에서는 그 포트에 직접 접근할 수 없습니다.

**비유로 이해하기**: 아파트 건물(호스트)에 여러 세대(컨테이너)가 있다고 상상해봅시다. 각 세대는 독립된 내부 공간이므로, 외부 방문객이 특정 세대로 들어가려면 반드시 건물 안내 데스크(호스트의 포트)를 통해 연결해줘야 합니다. 이 연결 작업이 바로 **포트 매핑**입니다.

```text
브라우저 요청
  → 호스트의 8080번 포트
    → (포트 매핑: -p 8080:80)
      → 컨테이너 내부 80번 포트 (Nginx)
```

포트 매핑 없이 `docker run nginx`만 실행하면, Nginx는 컨테이너 내부에서 혼자 동작할 뿐 외부에서 접근할 방법이 없습니다.

**네트워크 네임스페이스와 모드 차이**

* **Bridge 모드 (기본)**: 컨테이너가 호스트와 완전히 격리된 내부 IP를 가지며, 명시적인 포트 매핑(`-p 호스트포트:컨테이너포트`)이 필수입니다.

* **Host 모드**: 컨테이너가 호스트의 네트워크 인터페이스를 그대로 공유하므로 포트 매핑 없이 호스트의 포트를 직접 사용합니다. 격리성이 낮아 보안에 더 주의가 필요합니다.

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

## 7. 데이터 영속성: Bind Mount vs Docker Volume

**왜 데이터 영속성이 필요한가?**

컨테이너는 기본적으로 **일회성**입니다. 컨테이너를 삭제(`docker rm`)하면 그 안에서 생성된 모든 데이터도 함께 사라집니다. 예를 들어, DB 컨테이너를 삭제하면 저장된 데이터가 모두 소멸됩니다. 이를 방지하려면 컨테이너의 생명주기와 무관하게 데이터를 보존하는 방법이 필요하며, Docker는 두 가지 방식을 제공합니다.

| 비교 항목 | Bind Mount | Docker Volume | 
| ----- | ----- | ----- | 
| **개념** | 호스트의 특정 디렉토리를 컨테이너 경로에 직접 연결 | Docker 엔진이 직접 관리하는 별도의 스토리지 영역을 마운트 | 
| **데이터 위치** | 개발자가 지정한 호스트 경로 (예: `$(pwd)/app`) | Docker가 내부적으로 관리 (예: `/var/lib/docker/volumes/`) | 
| **주요 용도** | 개발 중 소스코드를 실시간으로 컨테이너에 반영할 때 | DB, 로그 등 운영 데이터의 영속적 보관 | 
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

### 8-1. Docker 빌드 실패 (Dockerfile 오타)

* **문제 현상**: `docker build` 실행 시 `E: Invalid operation insall` 에러 발생 후 빌드 중단.

* **원인**: Dockerfile의 `RUN` 명령어에 오타(`insall`)가 포함되어 셸 명령이 실패.

* **진단**:

  ```bash
   => ERROR [2/2] RUN apt-get update && apt-get insall -y curl
  ------
   > [2/2] RUN apt-get update && apt-get insall -y curl:
  #5 0.512 E: Invalid operation insall
  ------
  executor failed running [/bin/sh -c apt-get update && apt-get insall -y curl]
  ```

* **해결**: Dockerfile에서 오타를 수정(`insall` → `install`)한 뒤 다시 빌드. Docker는 레이어를 캐싱하므로 수정된 줄 이전까지는 재실행 없이 빠르게 진행됩니다.

---

### 8-2. 포트 충돌 (port is already allocated)

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

---

### 8-3. Git Push 실패 (인증 에러)

* **문제 현상**: `git push origin main` 실행 시 `Support for password authentication was removed.` 에러 발생. (또는 403 Forbidden 에러)

* **원인**: GitHub 원격 저장소 푸시 시 기존 비밀번호 대신 Personal Access Token(PAT) 또는 SSH Key 인증이 필요함.

* **해결**: GitHub Developer Settings에서 Classic Token을 발급받은 후, 자격 증명 팝업 또는 터미널 프롬프트 비밀번호 란에 발급받은 토큰을 입력하여 인증 성공.

  ```bash
  $ git push origin main
  Enumerating objects: 5, done.
  To https://github.com/Hucklesong/codyssey_mission1.git
     a1b2c3d..7f8b9c0  main -> main
  ```

## 9. Git & GitHub 연동

### 9-1. Git과 GitHub의 역할 차이

이 둘은 이름이 비슷하지만 완전히 다른 도구입니다.

| 구분 | Git | GitHub |
| ----- | ----- | ----- |
| **종류** | 소프트웨어 (버전 관리 도구) | 서비스 (웹 플랫폼) |
| **동작 위치** | 내 컴퓨터 (로컬) | 인터넷 서버 (원격) |
| **핵심 역할** | 변경 이력을 추적하고 버전을 관리 | 원격 저장소를 호스팅하고 협업 기능 제공 |
| **인터넷 불필요** | 오프라인에서도 `commit`, `log`, `diff` 가능 | 필수 (push, pull, PR 모두 온라인) |

**비유로 이해하기**: Git은 내가 쓰는 개인 작업 일지(로컬 버전 관리)이고, GitHub는 그 일지를 팀원과 함께 볼 수 있도록 공유 게시판(원격 협업 플랫폼)에 올리는 행위입니다. GitHub가 없어도 Git은 혼자 완전하게 동작하며, GitHub 외에도 GitLab, Bitbucket 등 다른 원격 플랫폼을 사용할 수 있습니다.

**주요 작업 흐름**

```text
[로컬 - Git]                        [원격 - GitHub]
  작업 → git add → git commit   →(git push)→   원격 저장소
                                 ←(git pull)←   (팀원과 공유)
```

```bash
$ git config --global user.name "username"
$ git remote -v
origin  https://github.com/Hucklesong/codyssey_mission1.git (fetch)
origin  https://github.com/Hucklesong/codyssey_mission1.git (push)
```