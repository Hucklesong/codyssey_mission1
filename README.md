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