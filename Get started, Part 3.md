## Docker 시작하기
> 도커 공식 문서의 Get started의 Part 1를 번역한 글입니다.

# Part3 : *서비스*

#### part3 시작에 앞서
Part 3 에서는 어플리케이션을 확장하고 부하분산을 가능케 할 것입니다. 이를 위해 우리는 배포된 어플리케이션의 계층구조에서 한 단계 위로 가야합니다. 이를 *서비스* 라고 합니다.


## 서비스란?

배포된 응용프로그램에서 응용프로그램의 다른 부분을 우리는 "서비스"라고 부릅니다. 예를 들어 비디오 공유 사이트를 생각해 보면 어플리케이션의 데이터를 데이터베이스에 저장하는 서비스, 사용자가 비디오를 업로드 할 경우 그것을 사이트에 맞게 백그라운드에서 변환해주는 서비스, 또는 화면에서의 동작을 위한 서비스.. 등등을 포함합니다.

서비스는 실제로 프로덕션의 컨테이너 입니다. 서비스는 오직 하나의 이미지를 실행하지만 이미지가 실행되는 방식, 실행되기 위해 필요한 포트, 실행해야 하는 컨테이너 수, 필요한 서비스 용량 등을 체계화합니다.

서비스를 확장하면 컨테이너 인스턴스의 갯수가 변경되어 서비스에 더 많은 컴퓨팅 리소스가 할당됩니다.

다행히, 도커플랫폼을 이용하면 서비스를 확장하고 정의하는 것은 매우 쉽습니다.
```docker-compose.yml```파일만 작성하면요!

## 첫 번째 ```docker-compose.yml``` 파일

```docker-compose.yml```파일은 도커컨테이너가 어떻게 행동할지 정의하는 YAML파일입니다.

원하는 곳에 다음 내용의 파일을 ```docker-compose.yml```이라는 이름으로 저장합니다. 저는 Test폴더에 저장했습니다.
여기서 주의할점은 image: 뒤에 본인이 part2에서 작성한대로! 이름, 저장소,태그를 써주서야 합니다. 아래파일을 복사한 후 이름부분을 image: 뒷부분을 수정 해주셔야 합니다.

```
version: "3"
services:
  web:
    # replace username/repo:tag with your name and image details
    image: hansol/get-started:part2
    deploy:
      replicas: 5
      resources:
        limits:
          cpus: "0.1"
          memory: 50M
      restart_policy:
        condition: on-failure
    ports:
      - "4000:80"
    networks:
      - webnet
networks:
  webnet:
```
이 파일은 도커에게 아래 사항들을 수행하도록 합니다.
```
- part2에서 업로드한 이미지를 원격저장소로 부터 받아옵니다.
- 웹서비스로 이미지의 5개 인스턴스를 실행하고 각 이미지 최대 CPU의 10%, RAM의 50MB를 사용하도록 제한합니다.
- 실패시 즉시 재실행합니다.
- 호스트의 4000포트를 웹의 80포트로 매핑합니다.
- webnet이라는 부하분산 네트워크를 통해 웹의 컨테이너가 80포트를 공유하도록 합니다.(내부적으로 컨테이너 자체는 임시프트에서 웹의 80포트로 배포됩니다.)
- webnet을 기본설정으로 세팅하도록 합니다.
```

## 새롭게 부하분산된 앱을 실행해보자

우선, 다음 명령을 실행해봅시다.
```
docker swarm init
```

이제 실행 해 봅시다. 먼저 앱의 이름을 지정해줍니다. 저는 ```getstartedlab```이라고 지정해주었습니다.
```
docker stack deploy -c docker-compose.yml getstartlab
```
이 서비스는 배포된 이미지의 5개 컨테이너 인스턴스를 한 호스트에서 실행합니다. 이를 알아보기위해 우리의 앱에서 하나의 서비스에 대한 서비스ID를 얻어봅시다.
```
docker servive ls
```
```
ID                  NAME                MODE                REPLICAS            IMAGE                      PORTS
l2h7262gy1ag        getstartlab_web     replicated          0/5                 hansol/get-started:part2   *:4000->80/tcp
```
여기서 제가 얻은 서비스ID값을 찾아봅시다. 이름에는 지정해준 이름이 표시될 것이고 리스트에는 복제본 수, 이미지 이름, 매핑포트도 표시됩니다.

서비스에서 실행되는 단일 컨테이너를 *task*(태스크)라고 부릅니다. 태스크들 에는 앞서 ```.yml```에서 정의해준 replicas의 수만큼 숫자가 증가하는 고유한ID를 부여됩니다.

태스크들의 리스트를 확인해봅시다.
```
docker service ps getstartedlab_web
```

## 앱 확장

이제, ```docker-compose.yml```파일 내의 ```replicas```값을 변경후 저장하고 다시 ```docker stack deploy```명령어를 이용하면 앱의 크기를 조절할 수 있습니다.
```
docker stack deploy -c docker-compose.yml getstartedlab
```
도커는 업데이트를 먼저 수행하기 때문에 스택을 다운시키거나 컨테이너를 제거할 필요가 없습니다.

이제 다시 ```docker container ls -q```명렁어를 실행해봅시다. 배포된 인스턴스가 재구성된것을 확인할 수 있습니다. 앱을 확장했다면 더 많은 태스크와 컨테이너가 실행될 것입니다.

## 앱 종료

앱을 다운시켜 봅시다.
```
docker stack rm getstartedlab
docker swarm leave --force
```

도커를 사용하여 앱을 확장하는 것을 매우 쉽습니다.

## 요약
```
docker stack ls                                            # List stacks or apps
docker stack deploy -c <composefile> <appname>  # Run the specified Compose file
docker service ls                 # List running services associated with an app
docker service ps <service>                  # List tasks associated with an app
docker inspect <task or container>                   # Inspect task or container
docker container ls -q                                      # List container IDs
docker stack rm <appname>                             # Tear down an application
docker swarm leave --force      # Take down a single node swarm from the manager
```