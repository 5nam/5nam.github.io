---
layout: single
title:  "[니어니어 프로젝트] 배포 자동화 및 무중단 배포"
published : false
categories: niornear_project
---

# 배포 전략

## 1. 롤링 배포

롤링 배포는 전체 시스템을 중단하지 않고 새로운 버전을 점진적으로 배포, 업데이트 하는 것

서비스의 가용성을 유지하면서도 새로운 기능이나 버그 수정분을 배포할 수가 있음

예) 4대의 서버로 구성된 시스템을 운영한다고 하면, 먼저 첫 번째 서버에 배포를 시작

그리고 새로운 버전의 소프트웨어가 서버에 배포가 되고 정상 작동되는지 확인

만약, 정상적으로 동작하지 않으면, 다음 서버로 배포를 계속 진행하지 않고 롤백 진행

롤백을 하기 위해서 애플리케이션이 정상적으로 동작하는지 테스트하는 과정이 CD

CD 테스트를 첫 번재 서버가 배포 후에 잘 통과했으면 두 번째 서버에도 배포

이렇게 점진적으로 모든 서버에 배포하는 것을 롤링 배포라고 함

**장점**

![image-20241014153757051]({{site.url}}/images/2024-10-14-niornear_project(30)/image-20241014153757051.png)

- 전체 시스템의 처리 능력을 대부분 유지하면서 배포할 수 있음
  - 사용자가 서비스를 이용하고 있는 도중에 배포가 진행되어도 사용자는 여전히 기존 서버에서 제공하고 있는 이 세 가지 기존 서버들로 안정적으로 제공하는 서비스를 이용할 수 있

![image-20241014153845288]({{site.url}}/images/2024-10-14-niornear_project(30)/image-20241014153845288.png)

- 점진적인 적용을 통해 리스크 최소화
  - 먼저 배포된 서버가 문제가 있는지 없는지 알 수 있음
  - CD 테스트에 걸리지는 않았지만 유저가 체감하는 문제 발생하는 등
  - 일부 유저에서만 문제가 발생을 하고, 그 문제가 발생한 걸 캐치해서 더 큰 장애로 번지는 것을 막을 수 있음

**단점**

![image-20241014154525822]({{site.url}}/images/2024-10-14-niornear_project(30)/image-20241014154525822.png)

- 배포 속도 느림
  - 서버를 하나씩 업데이트 하다 보니, 전체 시스템에 새로운 버전을 적용하는데 시간이 많이 소요
  - 특히, 대규모 시스템이라고 하면 모든 서버 업데이트에 상당한 시간 소요될 수 있음
  - 그래서 긴급하게 업데이트 해야 하는 상황이라면 문제될 수 있음
- 동시에 서로 다른 버전으로 서비스될 수 있음
  - 새로운 버전이 DB 스키마가 크게 달라졌다던지 아니면 API 간 호환이 안 되는 문제 등이 있으면 동시에 같이 배포되는 것이 좋음

## 2. 블루그린 배포

전체 시스템을 중단하지 않고, 새로운 버전을 별도 환경에 배포하고 즉시 전환

블루그린 배포에서의 두 환경을 각각 블루와 그린이라고 부름. 현재 서비스 중인 환경이 블루라면 새로운 버전은 그린 환경이 되는 것

블루와 그린은 그냥 순서를 번갈아 가면서 사용하는 것일 뿐 현재 버전을 무조건 블루라고 부르는 것은 아님

![image-20241014161700125]({{site.url}}/images/2024-10-14-niornear_project(30)/image-20241014161700125.png)

예) 현재 블루라는 환경에서 서비스를 운영하고 있다고 하면 이 상태에서 배포가 시작되면 현재 운영 중인 블루 환경과 완전히 동일한 그린 환경을 준비를 함. 이 상태에서 그린 환경에는 새로운 버전의 소프트웨어를 배포해가지고 실행을 시키는 것. 그 다음 모든 것이 정상적으로 동작하는지 CD 테스트를 해보기도 하고 사람이 직접 테스트를 해보기도 함

아직 사용자들에게는 그린 환경 쪽으로 트래픽을 포워딩하지 않은 상태. 원래 있던 블루 환경으로만 트래픽이 포워딩 되고 있는 상태

![image-20241014161858646]({{site.url}}/images/2024-10-14-niornear_project(30)/image-20241014161858646.png)

그린 환경에서 모든 것들이 동작한다고 확인이 됐다면 트래픽을 블루에서 그린으로 전환을 하는 것.

이 전환은 보통 로드밸런싱 설정을 변경하는 것으로 이루어지는데, 중요한 점은 이 전환이 거의 즉각적으로 이루어진다는 것. 그래서 사용자들은 내가 원래 블루 환경을 쓰다가 그린 환경을 쓰게 됐다는 변화를 거의 느끼지 못할 정도로 빠르게 변화가 됨.

그럼 블루 환경은 더 이상은 트래픽을 받지 않는 것. 그린 환경이 새로운 버전으로 트래픽을 받게 되더라도 블루 환경을 일정 기간 동안 유지를 함. 그린 환경의 배포가 잘 끝났다고 해도 예상치 못한 문제로 서비스를 운영하는 도중에 서비스 장애가 생길 수 있기 때문에 블루 그린에서는 롤백을 하기 위해서 그린 환경으로 포워딩 되던 트래픽을 다시 블루 환경으로 포워딩 하도록 바꿔주는 것이 롤백

여기서 그린 환경으로 바꾼 다음에 블루 환경을 없애버리면 롤백을 할 수가 없는 것. 그래서 그린 환경으로 새로 배포가 됐다고 하더라도 블루 환경은 남겨둬야 하는 것

**장점**

1. 즉각적인 전환과 롤백이 가능

   - 새로운 버전이 완전히 준비된 후에 트래픽을 전환하기 때문에 전체 시스템을 한 번에 새로운 버전으로 업데이트 할 수가 있음
   - 문제가 발생하면 그때 또 다시 즉시 블루로 바꿀 수도 있는 것
   - 이렇게 함으로써 서비스 중단 시간을 최소화 시킬 수 있음

   > 만약 롤링 배포였다면?
   >
   > 블루와 그린을 바로 왔다갔다 할 수 없었을 것. 한 대씩 서버가 배포되기도 하고 롤백도 한 대씩 차례로 이루어질 테니

2. 새 버전을 실제 환경과 동일한 조건에서 테스트

   ![image-20241014162550722]({{site.url}}/images/2024-10-14-niornear_project(30)/image-20241014162550722.png)

   - 새로운 버전을 실제 운영 환경과 동일한 환경에서 테스트를 해볼 수 있는 것
   - 그린 환경에서 모든 테스트랑 검증을 마친 후 트래픽을 전환하므로 예상치 못한 문제가 발생할 가능성을 크게 줄 일 수 있음
   - 특히 이 경우 성능 테스트 같은 것도 해볼 수 있음. 물론 이 서버들 말고 데이터베이스나 아니면 다른 외부에 있는 API 들에 트래픽이 많이 몰리게 되니까 그런 부분은 따로 처리를 해줘야 할 필요가 있지만, 이렇게 실제 서버로 테스트를 해볼 수 있다는 것이 중요한 장점

**단점**

1. 리소스 사용량이 많음

   ![image-20241014162909069]({{site.url}}/images/2024-10-14-niornear_project(30)/image-20241014162909069.png)

   - 서버가 두 배로 필요함
   - 두 개의 완전히 동일한 운영 서비스 환경을 유지를 해야 하므로 서버 비용이 거의 2배로 늘어날 수 있음
   - 이 경우 서버가 한 두 대가 아니라 대규모로 서버를 운영할 때는 당연히 상당한 부담이 될 수 있음

1. 33

## 최종 결정한 배포 전략 구조도

- ~~ 방식을 결정한 이유
  - 공익 목적으로 일주일동안의 운영을 목표로 하는 프로젝트여서 비용이 저렴해야 함
  - 또한, 서버를 두 대 돌릴 만큼의 규모가 아니기 때문에 서버 한 대에서 nginx 를 통해 무중단 배포를 하는 것이 가장 최적임
  - 최대한 EC2 서버의 부담을 줄이기 위해 EC2 서버 내에서 빌드를 진행하는 것이 아닌, github actions 서버 내부에서 진행하는 방식 선택

![image-20241014095335807]({{site.url}}/images/2024-10-14-niornear_project(30)/image-20241014095335807.png)

1. 개발 완료 후 main 브랜치에 push
2. github actions 는 push 를 인지하여, 빌드 후 빌드 파일 ec2 에 scp 로 전송
3. ec2 내부 스크립트를 실행하여 현재 구동 중이지 않은 포트 번호를 알아내고, 해당 포트 번호로 배포를 진행
4. 방금 배포한 포트번호가 안정적으로 잘 배포가 되었는지 health check 후, nginx reverse_proxy 의 주소 포트번호를 방금 배포한 포트 번호로 변경

# 배포

- 실제 배포를 위해 작성한 코드를 살펴보자.

## Github Actions

deploy.yml

```yaml
name: CI/CD 배포 자동화 # 워크플로우의 이름을 붙이는 것

on:
  push:
    branches:
      - master

jobs:
  Deploy:
    runs-on: ubuntu-latest

    steps:
      - name: 리포지토리 파일 불러오기
        uses: actions/checkout@v4

      - name: 빌드, 테스트를 위한 java17 설치
        uses: actions/setup-java@v4
        with:
          distribution: temurin
          java-version: 17

      - name: application.properties, application.properties, application.properties 생성
        env:
          APPLICATION_PROPERTIES: ${{ secrets.APPLICATION_PROPERTIES }}
        run:
          echo "$APPLICATION_PROPERTIES" > src/main/resources/application.properties

      - name: 빌드 및 테스트
        run: ./gradlew clean build

      - name: SCP 로 EC2 에 파일 전송
        uses: appleboy/scp-action@v0.1.7
        with:
          host: ${{ secrets.EC2_HOST }}
          username: ${{ secrets.EC2_USERNAME }}
          key: ${{ secrets.EC2_PASSWORD }}
          source: ./build/libs/*SNAPSHOT.jar
          target: ~/CI-CD-practice/tobe

      - name: 서버 배포
        uses: appleboy/ssh-action@v1.0.3 # 사용할 라이브러리 명시

        with:
          host: ${{ secrets.EC2_HOST }}
          username: ${{ secrets.EC2_USERNAME }}
          key: ${{ secrets.EC2_PASSWORD }}

          script: |
            cd ~/CI-CD-practice/scripts
            bash stop.sh
            bash start.sh
            bash health.sh
```

1. push 된 리포지토리 파일 불러오기
2. 빌드, 테스트를 위한 java 17 설치
3. application.properties 생성
4. 빌드 및 테스트
5. scp 로 ec2 에 빌드 완료 된 jar 파일 전송
6. 스크립트를 이용하여 서버에 배포

## EC2 JDK 설치

1. JDK 설치

   ```shell
   sudo apt update && /
   sudo apt install openjdk-17-jdk -y
   java -version
   ```

2. swap 생성

## nginx 설정

1. nginx 설치(linux 기준)

   ```shell
   sudo apt install nginx
   ```

2. nginx 기본 파일에 include 로 proxy 설정 파일 연동하기

   ```shell
   sudo vim /etc/nginx/nginx.conf
   ```

   ```shell
   http {
   	...
   	include /etc/nginx/conf.d/*.conf;
     include /etc/nginx/sites-enabled/*;
     include /etc/nginx/sites-available/mynginx.conf; # 이 코드 추가
   }
   ```

3. nginx 기본 파일에 연동할 proxy 설정 파일 생성

   ```shell
   sudo vim /etc/nginx/sites-available/mynginx.conf
   ```

   ```shell
   server {
           listen 80;
           listen [::]:80;
           // 외부에서 접근할 때는 localhost 로 되어 있으면 접근 안되므로 실제 EC2 서버 주소로 설정
           server_name {EC2 서버 퍼블릭 주소};
           root /usr/share/nginx/html;
   
           # Load configuration files for the default server block
           include /etc/nginx/default.d/*.conf;
           include /etc/nginx/conf.d/service-url.inc;
   
           location / {
                   proxy_pass $service_url;
                   proxy_set_header X-Real-IP $remote_addr;
                   proxy_set_header X-Fowarded-For $proxy_add_x_forwarded_for;
                   proxy_set_header Host $http_host;
           }
   }
   ```
   
   - `include /etc/nginx/conf.d/service-url.inc;` 로 proxy_pass 변수 설정
   
   ```shell
   sudo vim /etc/nginx/conf.d/service-url.inc
   ```
   
   ```shell
   set $service_url http://127.0.0.1:8082;
   ```
   
   - `switch.sh` 의 `echo "set \$service_url http://127.0.0.1:${IDLE_PORT};" | sudo tee /etc/nginx/conf.d/service-url.inc` 코드로 해당 파일의 내용 중 새로 배포한 포트 번호를 반영해서 변경해줌
   
   

## resources 관리

- EC2 내부에 properties 경로 만들기

```shell
cd 프로젝트명
mkdir resources
cd resources
sudo vim application-port1.properties
sudo vim application-port2.properties
```

## 무중단 배포 스크립트

> scripts 폴더에 관리

profile.sh

```sh
#!/usr/bin/env bash

# 쉬고 있는 profile 찾기 : real1 이 사용 중이면 real2 가 쉬고 있고, 반대면 real1 이 쉬고 있음

function find_idle_profile() {
  RESPONSE_CODE=$(curl -s -o /dev/null -w "%{http_code}" http://localhost/profile)

  if [ ${RESPONSE_CODE} -ge 400 ] # 400 보다 크면(즉, 40x/50x 에러 모두 포함)

  then
    CURRENT_PROFILE=real2
  else
    CURRENT_PROFILE=$(curl -s http://localhost/profile)
  fi

  if [ ${CURRENT_PROFILE} == real1 ]
  then
    IDLE_PROFILE=real2
  else
    IDLE_PROFILE=real1
  fi

  echo "${IDLE_PROFILE}"

}

# 쉬고 있는 profile 의 port 찾기
function find_idle_port() {

  IDLE_PROFILE=$(find_idle_profile)

  if [ ${IDLE_PROFILE} == real1 ]
  then
    echo "8081"
  else
    echo "8082"
  fi
}
```

- find_idle_profile()
  - 서버 80 포트로 `/profile` 요청을 보내서, 현재 실행 중인(nginx 가 가리키고 있는) 스프링부트 서버의 profile 이 무엇인지 알아냄
  - 그리고 현재 구동 중인 profile 이 아닌 그 반대 profile 이름 반환
- find_idle_prot()
  - 프로파일 찾은 것을 바탕으로 port 번호 찾기

stop.sh

```sh
#!/usr/bin/env bash

ABSPATH=$(readlink -f $0)
ABSDIR=$(dirname $ABSPATH) # 현재 stop.sh 가 속해있는 경로를 찾음. 하단의 코드와 같이 profile.sh 의 경로를 찾기 위해 사용됨
source ${ABSDIR}/profile.sh # 일종의 import 구문, 해당 코드로 인해 stop.sh 에서도 profile.sh 의 여러 function 을 사용할 수 있게 됨

IDLE_PORT=$(find_idle_port)

echo "> $IDLE_PORT 에서 구동 중인 애플리케이션 pid 확인"
IDLE_PID=$(lsof -ti tcp:${IDLE_PORT})

if [ -z ${IDLE_PID} ]
then
  echo "> 현재 구동 중인 애플리케이션이 없으므로 종료하지 않습니다."
else
  echo "> kill -15 $IDLE_PID"
  kill -15 ${IDLE_PID}
  sleep 5
fi
```

- profile.sh 에 있는 function 사용하여, 현재 구동 중이지 않은(nginx 가 가리키고 있지 않은) 서버 포트번호가 실행 중이라면 종료

start.sh

```sh
#!/usr/bin/env bash


ABSPATH=$(readlink -f $0)
ABSDIR=$(dirname $ABSPATH)
source ${ABSDIR}/profile.sh

REPOSITORY=~/
PROJECT_NAME=CI-CD-practice
IDLE_PROFILE=$(find_idle_profile)

echo "> 새로운 빌드 파일로 파일 교체"
rm -rf ~/CI-CD-practice/current
mkdir ~/CI-CD-practice/current
mv ~/CI-CD-practice/tobe/build/libs/*SNAPSHOT.jar ~/CI-CD-practice/current/project.jar
cd ~/CI-CD-practice/current
#sudo fuser -k -n tcp 8080 || true
#nohup java -jar project.jar > ./output.log 2>&1 &


echo "> 새 어플리케이션 배포"
nohup java -jar \
    -Dspring.config.location=/home/ubuntu/$PROJECT_NAME/resources/application-$IDLE_PROFILE.properties \
    -Dspring.profiles.active=$IDLE_PROFILE \
    project.jar > ../output.log 2>&1 &
rm -rf ~/CI-CD-practice/tobe

```

- 현재 구동 중이지 않은(nginx 가 가리키지 않은) profile 이름을 가져옴
- 그리고 GitHub actions 에서 받아온 tobe 디렉터리의 jar 파일을 current 폴더로 이동
- 그리고 current 경로로 이동한 뒤, 받아온 profile 이름으로 프로젝트 nohup 배포(active 설정으로 profile 설정해줌)
- 그 후 버전 관리를 위해 tobe 에 있는 jar 파일 삭제

health.sh

```sh
#!/usr/bin/env bash

ABSPATH=$(readlink -f $0)
ABSDIR=$(dirname $ABSPATH)
source ${ABSDIR}/profile.sh
source ${ABSDIR}/switch.sh

IDLE_PORT=$(find_idle_port)

echo "> Health Check Start!"
echo "> IDLE_PORT: $IDLE_PORT"
echo "> curl -s http://localhost:$IDLE_PORT/profile "
sleep 10

for RETRY_COUNT in {1..10}
do
  RESPONSE=$(curl -s http://localhost:${IDLE_PORT}/profile)
  UP_COUNT=$(echo ${RESPONSE} | grep 'real' | wc -l)

  if [ ${UP_COUNT} -ge 1 ]
  then # $up_count >= 1 ("real" 문자열이 있는지 검증)
      echo "> Health check 성공"
      switch_proxy
      break
  else
      echo "> Health check의 응답을 알 수 없거나 혹은 실행 상태가 아닙니다."
      echo "> Health check: ${RESPONSE}"
  fi

  if [ ${RETRY_COUNT} -eq 10 ]
  then
    echo "> Health check 실패. "
    echo "> 엔진엑스에 연결하지 않고 배포를 종료합니다."
    exit 1
  fi

  echo "> Health check 연결 실패. 재시도..."
  sleep 10
done
```

- **for 루프** 최대 10번까지 Health Check를 반복
- **curl -s http://localhost:${IDLE_PORT}/profile**  `IDLE_PORT`에서 `profile` 엔드포인트로 요청 보냄
- **grep 'real'**  응답에서 `real`이라는 문자열이 포함된 경우, 이는 애플리케이션이 정상적으로 실행되고 있다는 것을 의미
- **UP_COUNT**  `grep`의 결과로 `real`이 포함된 경우, `wc -l` 명령어를 사용해 그 줄의 개수를 셈. `real` 문자열이 있으면 `UP_COUNT`는 1 이상이 됨
  - `if [ ${UP_COUNT} -ge 1 ]` `UP_COUNT`가 1 이상이면 Health Check에 성공한 것으로 간주.
  - **switch_proxy**  `switch.sh`에서 불러온 함수로, Nginx가 새로운 애플리케이션으로 트래픽을 전환하도록 설정을 변경하는 역할. 전환이 성공하면 `break`로 루프를 빠져나옴

switch.sh

```sh
#!/usr/bin/env bash

ABSPATH=$(readlink -f $0)
ABSDIR=$(dirname $ABSPATH)
source ${ABSDIR}/profile.sh

function switch_proxy() {
    IDLE_PORT=$(find_idle_port)

    echo "> 전환할 Port: $IDLE_PORT"
    echo "> Port 전환"
    echo "set \$service_url http://127.0.0.1:${IDLE_PORT};" | sudo tee /etc/nginx/conf.d/service-url.inc

    echo "> 엔진엑스 Reload"
    sudo service nginx reload
}
```

- `/etc/nginx/conf.d/service-url.inc` 파일 내부에 있는 proxy_pass url 현재 최신 코드 반영해서 새로 배포한 서버로 다시 써줌
- 그 후, 엔진엑스 다시 로드