## Docker 시작하기
> 도커 공식 문서의 Get started의 Part 1를 번역한 글입니다. https://docs.docker.com/get-started/
# Part1 : *오리엔테이션, 설정*
## **도커**의 개념
도커는 개발자와 시스템 관리자가 어플리케이션을 컨테이너를 이용하여 개발,배포 및 실행하기 위한 플랫폼입니다.

*containerization* (컨테이너화) 는 리눅스 컨테이너를 이용하여 어플리케이션을 배포하는것을 말합니다.

컨테이너라는 개념은 새로운 것이 아닙니다. 하지만 어플리케이션을 쉽게 배포하는데 사용 될 수 있습니다.

*containerization*는 다음과 같은 이유로 유용합니다.
```
- 유연성 : 아무리 복잡한 어플리케이션이라도 컨테이너화가 가능합니다.
- 가벼움 : 컨테이너는 호스트 커널을 활용,공유합니다.
- 상호교환적 : 어플리케이션을 즉시 배포, 업그레이드 할 수 있습니다.
- 휴대성 : 어디에서나 로컬 환경을 만들고, 클라우드에 배포하고 실행할 수 있습니다.
- 확장성 : 컨테이너를 확장하고 자동으로 복제본을 배포할 수 있습니다.
- 구축성 : 언제든 서비스를 병렬적으로 저장할 수 있습니다.
```

## **이미지**와 **컨테이너**
컨테이너는 이미지를 구동함으로써 실행됩니다.

*이미지* 는 어플리케이션을 구동하기 위한 모든 정보(코드, 라이브러리, 실행환경, 설치파일 등)를 포함하는 패키지 입니다.

*컨테이너*는 이미지를 실행될 때 메모리에 저장되는 이미지의 런타임 인스턴스 입니다. 리눅스에서는 ```docker ps``` 명령어를  통해 실행중인 컨테이너 목록을 확인 할 수 있습니다.

## **컨테이너**와 **가상머신**
*컨테이너*는 기본적으로 리눅스에서 실행되고 다른 컨테이너와 호스트 커널로 공유됩니다. 이 방식은 개별 프로세를 통해 실행되어 메모리 사용을 줄이고 가볍게 만듭니다.

반대로 *Virtual Machine*(가상머신)은 하이퍼바이저를 통해 호스트 리소스에 대한 가상 액세스 권한을 가진 완전한 "게스트"운영체제를 실행합니다. 일반적으로 가상머신은 어플리케이션을 실행하기 위해 필요한 것보다 더 큰 환경을 제공합니다.

## **도커 실행** 준비
도커 공식홈페이지에서 도커 다운받으시면 됩니다.

## **도커 버전** 확인
도커를 다운받으셨으면 제대로 설치 되었는지 다음과 같이 확인 해 볼 수 있습니다.

1. ```docker --version``` 명령어를 이용하여 도커 버전을 확인하세요:

```
docker --version
```
```
Docker version 18.09.0, build 4d60db4
```


2. ```docker info``` 명령어를 통해 더 자세한 정보를 얻을 수 있습니다.

```
docker info
```
```
Containers: 0
 Running: 0
 Paused: 0
 Stopped: 0
Images: 0
Server Version: 18.09.0
Storage Driver: overlay2
...
```

>```sudo```명령어를 입력하지 않을 경우 권한 오류가 생깁니다. 사용자를 ```docker```그룹에 추가하여 권한을 주시면 됩니다.
>```
>$ sudo groupadd docker #도커그룹생성
>$ sudo usermod -aG docker $USER
>```


## **도커 설치** 확인
1. 간단한 도커 이미지인 ```hello-world```를 실행하여 설치가 제대로 되었는지 확인해봅시다.
```
docker run hello-world

Unable to find image 'hello-world:latest' locally
latest: Pulling from library/hello-world
ca4f61b1923c: Pull complete
Digest: sha256:ca0eeb6fb05351dfc8759c20733c91def84cb8007aa89a5bf606bc8b315b9fc7
Status: Downloaded newer image for hello-world:latest

Hello from Docker!
This message shows that your installation appears to be working correctly.
...
```
2. ```hello-world``` 이미지가 제대로 다운되었는지 확인해봅시다. 다운받은 이미지를 다음과 같인 확인 할 수 있습니다.
```
docker image ls
```
```
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
hello-world         latest              4ab4c602aa5e        2 months ago        1.84kB
```
3. ```hello-world```이미지로 생성된 컨테이너 목록을 확인합니다. 만약 실행중이라면 ```--all```을 생략해도 됩니다.
```
docker container ls -all
```
```
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS                     PORTS               NAMES
da32d5db2a26        hello-world         "/hello"            4 minutes ago       Exited (0) 4 minutes ago                       reverent_visvesvaraya
```
## 요약
```
## List Docker CLI commands
docker
docker container --help

## Display Docker version and info
docker --version
docker version
docker info

## Execute Docker image
docker run hello-world

## List Docker images
docker image ls

## List Docker containers (running, all, all in quiet mode)
docker container ls
docker container ls --all
docker container ls -aq
```