## Docker 시작하기
> 도커 공식 문서의 Get started의 Part 2를 번역한 글입니다.

# Part2 : *컨테이너*

#### part2의 시작에 앞서
이제 도커를 이용하여 앱을 제작해봅시다. 이 페이지에서 다루는 앱, 컨테이너의 가장 아래계층에서 부터 시작합니다. 이 단계를 끝내면 Part3에서 컨테이너가 어떻게 생산에서 활용되는지 즉, 서비스에 대해서 다룰 것 입니다.

## 새로운 **개발환경**
당신이 파이썬을 이용하여 앱을 만들고 싶다면 예전에는 먼저 파이썬을 당신의 컴퓨터에 설치해야 했습니다. 그러기 위해서는 당신의 앱이 작동하기 기대하는 환경과 당신의 컴퓨터 환경을 똑같이 만들어야 했고, 사용자의 환경도 똑같아야 되는 문제가 있었습니다.

도커를 활용한다면, 당신은 파이썬을 설치할 필요 없이 그냥 파이썬 이미지를 가져오기만 하면 됩니다. 그 후 파이썬 이미지와 당신의 앱 코드를 포함해 앱을 만들 수 있고, 모든 것과 함께 배포할 수 있습니다.

이 놀라운 이미지는 ```Dockerfile```이라고 불립니다.

## ```Dockerfile```로 컨테이너를 만들기

도커파일(```Dockerfile```)은 컨테이너내의 환경을 정의합니다. 네트워크 인터페이스나 디스크 드라이브와 같은 리소스에 접근하는 것은 환경내에서 가상화 되며 나머지 시스템과 격리되어있습니다. 따라서 앱을 외부로 보내기 전에 포트를 매핑하여야 하고 무슨 파일을 환경에 추가할 것인지 정확히 지정해야 합니다.
이러한 절차를 거치고 나면 도커파일로 정의된 당신의 앱은 어디서든지 기대한 대로 작동될 것입니다.

## Dockerfile
우선 빈 디렉토리를 만들고 빈 디렉토리에 ```Dockerfile```를 생성합니다. 다음 내용과 똑같이 파일내용을 만듭니다. 간단히 주석을 작성해 줍니다.
```
# Use an official Python runtime as a parent image
FROM python:2.7-slim

# Set the working directory to /app
WORKDIR /app

# Copy the current directory contents into the container at /app
COPY . /app

# Install any needed packages specified in requirements.txt
RUN pip install --trusted-host pypi.python.org -r requirements.txt

# Make port 80 available to the world outside this container
EXPOSE 80

# Define environment variable
ENV NAME World

# Run app.py when the container launches
CMD ["python", "app.py"]
```
이 도커파일에는 아직 만들지 않은 파일인 ```app.py```와 ```requirements.txt```가 언급되어 있습니다. 그러면 만들어 봅시다!

## 어플리케이션
```requirements.txt``` 와 ```app.py```파일을 만들어 도커파일이 위차한 디렉토리에 위치 시킵니다.
두 파일은 보시는 바와 같이 매우 간단히 우리의 앱을 완성시킵니다. 도커파일이 이미지에 내장이 되면 ```app.py```와 ```requirments.txt```는 도커파일내의 ```COPY```명령에 의해 이미지에 존재하게 되고 ```EXPOSE```명령에 의해 HTTP를 통해 ```app.py```에 접근할 수 있습니다.
```
#requirments.txt

Flask
Redis
```
```
#app.py
from flask import Flask
from redis import Redis, RedisError
import os
import socket

# Connect to Redis
redis = Redis(host="redis", db=0, socket_connect_timeout=2, socket_timeout=2)

app = Flask(__name__)

@app.route("/")
def hello():
    try:
        visits = redis.incr("counter")
    except RedisError:
        visits = "<i>cannot connect to Redis, counter disabled</i>"

    html = "<h3>Hello {name}!</h3>" \
           "<b>Hostname:</b> {hostname}<br/>" \
           "<b>Visits:</b> {visits}"
    return html.format(name=os.getenv("NAME", "world"), hostname=socket.gethostname(), visits=visits)

if __name__ == "__main__":
    app.run(host='0.0.0.0', port=80)
```
이제 우리는 도커파일내의 ```pip install -r requirements.txt```명령에 의해 Flask와 Redis 라이브러리가 설치되고 앱이 ```socket.gethostname()```에 대한 호출 결과인  환경변수 ```NAME```을 출력하는 것을 볼 수 있습니다. 마지막으로 Redis가 실행되지고 있지 않기 때문에(우리는 오직 파이썬 라이브러리만 설치했지, Redis를 자체를 설치하지 않았다.)사용시도가 실패되고 오류메세지가 출력됩니다.

> 참고
> 컨테이너 내부에서 호스트 이름에 액세스할 경우 실행중인 실행 파일의 프로세스 ID와 같은 컨테이너 ID를 검색합니다.

자, 끝입니다! 우리의 시스템에는 파이썬이나 ```requirements.txt```같은 파일들이 필요하지 않습니다. 또한 이미지를 빌드하고 실행하더라도 시스템에 설치되지 않습니다. 파이썬이나 플라스크로 환경을 구성한것 처럼 보이진 않지만 그렇게 한 것입니다.

## 앱을 빌드하자
이제 앱을 빌드할 준비가 끝났습니다. 우선 디렉토리의 최상위 레벨에 있는지 확인해봅시다.
```
ls
```
```
app.py  Dockerfile  requirments.txt
```
다음과 같이 표시가 된다면 빌드 명령을 실행해 봅시다. 빌드 명령을 실행하면 도커이미지가 생성이됩니다. ```-t```명령어를 통해 자신에게 친숙한 이름으로 이미지를 저장할 수 있습니다.
```
docker build -t friendlyhello .
```
저는 friendlyhello. 라는 이름으로 저장했습니다. 저장된 이미지는 어디 있을까요? 컴퓨터의 로컬 도커 이미지 저장소에 저장이 됩니다.
```
docker image ls
```
```
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
friendlyhello       latest              07277397b5c6        6 seconds ago       131MB
```

## 앱을 실행하자
 ```-p```를 이용해 컨테이너에 제공된 80포트를 컴퓨터의 400포트로 매핑하여 앱을 실행하여 봅시다.
 ```
 docker run -p 4000:80 friendlyhello
 ```
 ```
 * Serving Flask app "app" (lazy loading)
 * Environment: production
   WARNING: Do not use the development server in a production environment.
   Use a production WSGI server instead.
 * Debug mode: off
 * Running on http://0.0.0.0:80/ (Press CTRL+C to quit)
 ```
 파이썬이 http://0.0.0.0:80 로 앱을 실행한다는 메세지를 확인할 수 있습니다. 하지만 이 메세지는 컨테이너 내부에서 오므로 컴퓨터의 4000포트와 제대로 매핑이 되었는지 알 수 없습니다. 
 
 그러면 제대로 되었는지 인터넷 브라우저 창에 http://localhost:4000 를 입력하여 확인해봅시다.
 
 ![](/picture/1.png)

 또는 ```curl```명령을 통해서도 확인이 가능합니다.
 ```
 curl http://localhost:4000
 ```
 ```
 <h3>Hello World!</h3><b>Hostname:</b> 0872d07512f8<br/><b>Visits:</b> <i>cannot connect to Redis, counter disabled</i>
```

4000:80에 의해 매핑된 포트는 도커파일내의 ```EXPOSE```와 도커를 실행시킬때 ```-p```로 설정해준 값과의 차이를 보여줍니다. 다음 단계에서는 호스트의 포트4000을 컨테이너의 포트80에 매핑하고 http://localhost를 사용합니다.

```CTRL+C```를 눌러 터미널을 종료합니다.

이제 분리모드로 백그라운드에서 앱을 실행시켜 봅시다.
```
docker run -d -p 4000:80 friendlyhello
```
```
d0cee9ba20625e63766905be92b14699ceb09f5b489a84b260a114a22aad6d4e
```
위 명령어를 실행하면 컨테이너ID를 얻을 수 있습니다. 이때 컨테이너는 백그라운드에서 실행중입니다.

```docker container ls```명령어를 통해 축약된 ID를 볼 수도 있습니다.
```
docker container ls
```
```
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS                  NAMES
d0cee9ba2062        friendlyhello       "python app.py"     2 minutes ago       Up 2 minutes        0.0.0.0:4000->80/tcp   admiring_edison
```
컨테이너 ID는 http://localhost:4000에 표시된것과 일치합니다.

이제 ```docker container stop```명령어와 컨테이너ID를 통해 실행을 중지해봅시다.

```
docker container stop d0cee9ba20625e63766905be92b14699ceb09f5b489a84b260a114a22aad6d4e
```
or
```
docker container stop d0cee9ba2062
```

컨테이너ID가 표시되며 중지됩니다.

## 이미지 공유

방금 만든 이미지를 어디에서나 사용할 수 있음을 보여주기 위해 이미지를 업로드하고 다른데서 실행해봅시다. 그러면 컨테이너를 사용자에게 배포하기 위해 어떻게 저장소에 업로드 하는지 알아봅시다.

*registry*(저장소)에는 많은 저장된 이미지들이 있으며 깃헙저장소와 비슷합니다. 도커는 기본적으로 오픈소스를 제공합니다.
>참고
>공용 레지스트리는 무료이지만, 유료로 개인 레지스트리를 사용할 수 도 있습니다.

## Docker ID 로 로그인

hub.docker.com 에 들어가시면 도커계정을 생성할 수 있습니다.

로컬시스템에서 도커공용레지스트리에 로그인 해봅시다.

```
docker login 
```
한번만 로그인 하면 다음부터는 자동으로 로그인이 됩니다.

## 이미지에 태그추가
로컬 이미지를 레지스트리의 저장소와 연동하기 위한 표기법은 다음과 같습니다.
```
username/repository:tag
```
태그는 선택사항이지만 도커 이미지에 버전을 표시하는 매커니즘 이기때문에 권장됩니다. 저장소와 태그에 의미있는 이름을 붙여봅시다. 만약 ```get-started:part2```으로 이름짓는다면 ```get-started```라는 저장소에 ```part2```라는 태그로 저장됩니다.

자, 이제 이미지에 태그를 추가해봅시다.

```
docker tag image username/repository:tag
```
저 같은 경우는 다음과 같이 작성했습니다.
```
docker tag friendlyhello hansol/get-started:part2
```
이제, 이미지 목록을 보면 새롭게 태그된 이미지를 확인 할 수 있습니다.
```
docker image ls
```
```
REPOSITORY           TAG                 IMAGE ID            CREATED             SIZE
hansol/get-started   part2               07277397b5c6        About an hour ago   131MB
friendlyhello        latest              07277397b5c6        About an hour ago   131MB
```

## 이미지 업로드
태그한 이미지를 저장소에 업로드 해봅시다.
```
docker push username/repository:tag
```
저 같은 경우는 
```
docker push hansol/get-started:part2
```
가 되겠네요

완료가 되면 도커허브(hub.docker.com)에 들어가서 업로드가 제대로 되었는지 확인 할 수 있습니다.

![](/picture/2.png)

## 이미지 다운로드 및 실행
이제 원격저장소에 업로드된 이미지를 누구나 다운받아 모든 컴퓨터에서 실행할 수 있습니다.
```
docker run -p 4000:80 username/repository:tag
```
방금 제가 업로드한 이미지를 당겨올려면 다음과 같이 쓸 수 있겠네요.
```
docker run -p 4000:80 hansol/get-started:part2
```
만약 이미지가 로컬저장소에 없는 경우 도커는 원격저장소에서 자동으로 이미지를 가져옵니다. 공개된 저장소의 이미지들은 위와 같은 방식으로 누구나 사용할 수 있습니다. 어느 컴퓨터에서 이미지가 실행이 되든 requirments.txt와 파이썬등 모든 필요한 환경이 이미지에 포함되어 있으므로 시스템에 아무것도 설치할 필요가 없습니다.

##요약
```
docker build -t friendlyhello .  # Create image using this directory's Dockerfile
docker run -p 4000:80 friendlyhello  # Run "friendlyname" mapping port 4000 to 80
docker run -d -p 4000:80 friendlyhello         # Same thing, but in detached mode
docker container ls                                # List all running containers
docker container ls -a             # List all containers, even those not running
docker container stop <hash>           # Gracefully stop the specified container
docker container kill <hash>         # Force shutdown of the specified container
docker container rm <hash>        # Remove specified container from this machine
docker container rm $(docker container ls -a -q)         # Remove all containers
docker image ls -a                             # List all images on this machine
docker image rm <image id>            # Remove specified image from this machine
docker image rm $(docker image ls -a -q)   # Remove all images from this machine
docker login             # Log in this CLI session using your Docker credentials
docker tag <image> username/repository:tag  # Tag <image> for upload to registry
docker push username/repository:tag            # Upload tagged image to registry
docker run username/repository:tag                   # Run image from a registry
```





