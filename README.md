# Mission 1 - 개발 워크스테이션 구축

## 1) 실행 환경

| 항목 | 버전 |
|------|------|
| OS | Ubuntu 22.04 (WSL2) |
| Shell | zsh |
| Docker | 29.9.3 |
| Git | 2.53.0 |


## 2) 수행 체크리스트

- [x] 터미널 환경 구성 (zsh)
- [x] 파일/디렉토리 권한 확인
- [x] Docker 설치 및 실행 확인
- [x] hello-world 컨테이너 실행
- [x] Dockerfile 작성 및 이미지 빌드
- [x] 포트 매핑 실습
- [x] 바인드 마운트 실습
- [x] 볼륨 실습
- [x] Git & GitHub 연동


## 3) 수행 로그

### 터미널 환경 확인
```bash
echo $SHELL
zsh --version

#출력 결과
/usr/bin/zsh
zsh 5.9 (x86_64-ubuntu-linux-gnu)
```

### Docker 확인

```bash
docker --version
docker info
```
```bash
#출력 결과 :
docker version 29.9.3, build 5457199

Client: Docker Engine - Community
 Version:    29.9.3
 Context:    default
 Debug Mode: false
 Plugins:
  buildx: Docker Buildx (Docker Inc.)
    Version:  v0.38.1
  compose: Docker Compose (Docker Inc.)
    Version:  v5.7.0

Server:
 Containers: 1
  Running: 0
  Stopped: 1
 Images: 1
 Server Version: 29.9.3
 OS: Ubuntu 26.04.1 LTS
 Kernel: 6.18.54.1-microsoft-standard-WSL2
 Architecture: x86_64
 CPUs: 20
 Total Memory: 15.47GiB
```
### 권한 변경 실습
**파일 권한 변경 (README.md)**

**변경 전**

```bash
ls -l README.md
.rw-r--r-- 1.2k hucklesong README.md
```
**변경 후**

```bash
chmod 755 README.md
ls -l README.md
.rwxr-xr-x 1.2k hucklesong README.md
```
**디렉토리 권한 변경 (app/)**

** 변경 전**
```bash
ls -l
drwxr-xr-x - hucklesong app
drwxr-xr-x - hucklesong practice
```
** 변경 후**
```bash
chmod 700 app/
ls -l
drwx------ - hucklesong app
drwxr-xr-x - hucklesong practice
```

---

### hello-world 컨테이너 실행

```bash
docker run hello-world
```

```bash
# 출력 결과 :
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
```

## 4. Docker 커스텀 이미지 실습

### 4-1. Dockerfile 작성

```dockerfile
# app/Dockerfile
FROM ubuntu:22.04
RUN apt-get update && apt-get install -y curl
CMD ["echo", "Hello from my container!"]
```

### 4-2. 이미지 빌드

```bash
➜  app git:(main) ✗ docker build -t my-ubuntu .
[+] Building 33.9s (7/7) FINISHED                                              docker:default
 => [internal] load build definition from Dockerfile                                     0.0s
 => [internal] load metadata for docker.io/library/ubuntu:22.04                          6.3s
 => [1/2] FROM docker.io/library/ubuntu:22.04                                            3.3s
 => [2/2] RUN apt-get update && apt-get install -y curl                                 20.4s
 => exporting to image                                                                   3.6s
 => => naming to docker.io/library/my-ubuntu:latest                                      0.0s
 => => unpacking to docker.io/library/my-ubuntu:latest                                   0.0s
```

### 4-3. 이미지 확인

```bash
➜  app git:(main) ✗ docker images
IMAGE                ID             DISK USAGE   CONTENT SIZE
hello-world:latest   c567b231f656       25.9kB         9.49kB
my-ubuntu:latest     150ad5d13caf        265MB         87.6MB
```

### 4-4. 컨테이너 실행

```bash
➜  app git:(main) ✗ docker run my-ubuntu
Hello from my container!
```

## 포트 매핑 실습

### nginx 컨테이너 실행 (포트 매핑)
```bash
docker run -d -p 8080:80 --name my-nginx nginx
```

**실행확인**
```bash
docker ps
```

```bash
CONTAINER ID   IMAGE   COMMAND                  CREATED        STATUS        PORTS                               NAMES
db1ebfa5a6c4   nginx   "/docker-entrypoint.…"   4 seconds ago  Up 4 seconds  0.0.0.0:8080->80/tcp, [::]:8080->80/tcp   my-nginx
```

**curl 접속 테스트**
```bash
curl http://localhost:8080
```
Welcome to nginx! HTML 응답 확인 ✅

## Bind Mount 실습

- 로컬 app/ 폴더를 nginx 컨테이너에 마운트
- 명령어:
```bash
  docker run -d -p 8080:80 --name my-nginx \
    -v $(pwd)/app:/usr/share/nginx/html nginx
```
- 결과: 로컬 파일 수정 시 컨테이너 재시작 없이 즉시 반영

## 5. Docker Volume 실습

### 개념
Volume은 컨테이너가 삭제되어도 데이터가 유지되는 Docker 저장소입니다.
DB, 로그, 설정파일 등 영구 보존이 필요한 데이터에 사용합니다.

### 실습 순서
1. Volume 생성 → 2. 컨테이너에 마운트 후 데이터 저장 → 3. 컨테이너 삭제 → 4. 새 컨테이너에서 데이터 유지 확인

### 주요 명령어
| 명령어 | 설명 |
|---|---|
| `docker volume create <name>` | Volume 생성 |
| `docker volume ls` | Volume 목록 확인 |
| `docker volume rm <name>` | Volume 삭제 |
| `-v <volume>:<경로>` | 컨테이너에 Volume 마운트 |

### 수행 로그

#### Volume 생성
```bash
docker volume create my-volume
my-volume

docker volume ls
DRIVER    VOLUME NAME
local     my-volume
```

**컨테이너 마운트 후 데이터 저장**
```bash
docker run -it \
  --name my-volume-test \
  -v my-volume:/data \
  ubuntu:22.04 \
  bash

root@abc123:/# echo "Hello, Volume!" > /data/test.txt
root@abc123:/# cat /data/test.txt
Hello, Volume!
root@abc123:/# exit
```

***컨테이너 삭제 후 새 컨테이너에서 데이터 유지 확인***
```bash
docker rm my-volume-test
my-volume-test

docker run -it \
  --name my-volume-test2 \
  -v my-volume:/data \
  ubuntu:22.04 \
  bash

root@632b6936c5cc:/# cat /data/test.txt
Hello, Volume!
root@632b6936c5cc:/#
```

결과
컨테이너(my-volume-test) 삭제 후 새 컨테이너(my-volume-test2)에서
cat /data/test.txt 실행 시 Hello, Volume! 출력 확인 ✅
→ 컨테이너 생명주기와 무관하게 데이터가 Volume에 영구 보존됨결과
컨테이너(my-volume-test) 삭제 후 새 컨테이너(my-volume-test2)에서
cat /data/test.txt 실행 시 Hello, Volume! 출력 확인 ✅
→ 컨테이너 생명주기와 무관하게 데이터가 Volume에 영구 보존됨