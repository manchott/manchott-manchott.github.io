---
layout: post
title: AWS 프리 티어 서버 배포
sitemap: true
categories:
  - project
  - Table Top Tracker
hide_last_modified: true
---
**AWS 진짜 시작이다**  
배포 환경설정만 3일 걸렸다.(중간에 인스턴스 날리고, IP 설정 제대로 안해서 시간 낭비를 많이 하긴 했다...) 이제부터 진짜 배포 시작이다.  
[스프링 부트와 aws로 혼자 구현하는 웹 서비스](https://m.yes24.com/Goods/Detail/83849117) 책의 서비스 배포 부분을 참고해서 진행했고, 책과 다르게 진행한 부분이 있다.

1. this ordered seed list will be replaced by the toc
{:toc}

# EC2에서 프로젝트 Build하기
## EC2에서 프로젝트 Clone 받고 빌드해보기
1. EC2에 git을 설치(책과 다름)  
`sudo dnf install git -y`
2. 프로젝트를 저장할 디렉토리 생성 후 이동, git clone
~~~shell
mkdir ~/app
cd ~/app
git clone [git repository 주소]
~~~
3. 프로젝트 파일로 들어가서 빌드  
`./gradlew build`  
책에서는 테스트를 돌렸는데 나는 테스트 코드가 없었어서 그냥 빌드를 진행했다. 이 때, 빌드가 도저히 끝나지 않는 문제를 마주했었는데 이는 [SWAP 메모리 지정](https://manchott.github.io/project/table%20top%20tracker/2023-12-30-AWS-errors/#프로젝트를-빌드할-때-시간이-너무-오래걸림)을 통해 해결했다.
4. 만약 `Permission denied` 메세지가 뜨면 `chmod +x ./gradlew`명령어로 실행 권한을 추가

## 배포 스크립트 작성
개발자가 작성한 코드를 서버에 반영하는 것을 **배포**라고 한다. 이는 여러 단계로 이루어져 있는데, 매번 이 단계를 거치는건 불편하니 배포 스크립트를 작성하고 실행하는 방법이 책에 나와있다.  
그러나 **코드를 많이 수정했다**. profile에 따른 yml 파일을 다르게 설정했기 때문에 서버 내에 환경변수로 설정해서 컨트롤해보려 했으나 도저히 되지 않아서 책에서 나와 있는 방법을 따르되, 약간을 변경했다.  
1. 외부 Security 파일 등록. app 디렉토리에 application.yml과 application-oauth.yml을 생성하고 기존 내용을 복붙했다.(`org.springframework.context.ApplicationContextException: Unable to start web server` 오류 생기면 여기 확인)
2. app 디렉토리에 deploy.sh 파일을 생성 후 아래 내용 작성  
~~~shell
#!/bin/bash
#
REPOSITORY=[프로젝트 디렉토리]
PROJECT_NAME=[프로젝트명]
#
cd $REPOSITORY/$PROJECT_NAME/
#
echo "> Git Pull"
#
git pull
#
echo "> 프로젝트 Build 시작"
#
./gradlew build
#
echo "> step1 디렉토리로 이동"
#
cd $REPOSITORY
#
echo "> Build 파일 복사"
#
cp $REPOSITORY/$PROJECT_NAME/build/libs/*.jar $REPOSITORY/
#
echo "> 현재 구동중인 애플리케이션 pid 확인"
#
CURRENT_PID=$(pgrep -f $PROJECT_NAME)
#
echo "현재 구동중인 애플리게이션 pid: $CURRENT_PID"
#
if [ -z $CURRENT_PID ]
then
        echo ">현재 구동 중인 애플리케이션이 없으므로 종료하지 않습니다."
else
        echo "> kill -15 $CURRENT_PID"
        kill -15 $CURRENT_PID
   sleep 5
fi
#
echo "> 새 애플리케이션 배포"
#
JAR_NAME=$(ls -tr $REPOSITORY/ | grep jar | tail -n 1)
#
echo "> JAR Name: $JAR_NAME"
echo $profile
nohup java -jar \
    -Dspring.config.location=optional:/application.yml,/home/ec2-user/app/application-prod.yml,/home/ec2-user/app/application-oauth.yml \
    -Dspring.profiles.active=prod \
    $REPOSITORY/$JAR_NAME 2>&1 &
~~~
3. 생성한 스크립트에 실행 권한 추가  
`chmod +x ./deploy.sh`
4. `./deploy.sh` 명령어로 스크립트 실행. 이 떄 `no main manifest attribute in` 에러가 뜬다면 이는 [jar 파일이 2개 생겨서](http://127.0.0.1:4000/project/table%20top%20tracker/2023-12-30-AWS-errors/#no-main-manifest-attribute-in-에러) 발생하는 에러이다.
5. 레포지토리 주소로 이동한 뒤 nohup.out 파일을 열어서 로그 확인  
`vim nohup.out`

## RDS에 테이블 생성
서버에서는 JPA의 설정을 `ddl-auto: none`으로 했다. 서버가 돌 때마다 데이터베이스를 create한다면 데이터베이스로의 의미가 없으니까. 이건 그냥 로컬에서 Spring 시작하면 로그로 나오는 JPA의 Create문을 실행해서 RDS에 테이블을 생성했다.

## 구글에 EC2 주소 등록
구글 로그인을 위해 EC2의 주소를 구글 클라우드에 등록해야한다.
이 부분은 책의 내용을 기반으로 설정을 했다. 책이 발간된지 5년이니 그새 구글 클라우드 콘솔이 바뀌어서 설정이 이게 맞나 헷갈렸는데, 성공적으로 소셜 로그인을 마쳤다. 그러니 잘 한것이겠지...?
1. 왼쪽 메뉴 - API 및 서비스 - OAuth 동의 화면  
![이미지](https://manchott.github.io/assets/img/AWS-deploy/gcc_1.png "OAuth 동의 화면"){: width="250" }
2. 서비스 이름 오른쪽의 `앱 수정` 클릭  
3. 쭉 내려서 승인된 도메인에 EC2 인스턴스의 `퍼블릭 IPv4 DNS` 입력

## Redis 설정
추가로 나는 Refresh Token과 Access Token을 Redis에서 관리하기 때문에 Redis까지 설정을 했다.  
여기서 정말 고생을 많이 했는데, AL2023에서는 Redis6을 지원한다. 이전 AMI에서는 Redis 사용이 더 까다로웠지만 Redis6은 개선되었다는 것을 모르고 삽질을 많이 했다... 레디스를 백그라운드에서 시작하게 하는 `daemonize` 설정을 Redis6에서는 할 필요 없었는데 계속 찾아보느라 고생했다.  
또한 [Redis의 백업 방법](https://inpa.tistory.com/entry/REDIS-%F0%9F%93%9A-%EB%8D%B0%EC%9D%B4%ED%84%B0-%EC%98%81%EA%B5%AC-%EC%A0%80%EC%9E%A5%ED%95%98%EB%8A%94-%EB%B0%A9%EB%B2%95-%EB%8D%B0%EC%9D%B4%ED%84%B0%EC%9D%98-%EC%98%81%EC%86%8D%EC%84%B1)으로 AOF를 선택했기 때문에 이것도 설정했다.  
1. Redis 설치
~~~shell
sudo dnf install -y redis6
sudo systemctl start redis6
sudo systemctl enable redis6
sudo systemctl is-enabled redis6
redis6-server --version
redis6-cli ping
~~~
2. Redis6 config 파일 변경
~~~shell
# config 파일 열기
sudo nano /etc/redis6/redis6.conf
# 수정할 부분
bind 0.0.0.0    # 모든 ip에서 오는 connection 허용
appendonly yes  # aof 파일 읽어오기
~~~
3. 비밀번호 설정...을 해야하는데 일단은 그대로 두고 추후에 추가할 예정.

Redis6을 실행할 때는 `sudo redis6-server /etc/redis6/redis6.conf`를 입력하면 되고, 따로 설정하지 않아도 백그라운드에서 돌아간다. 또한 redis-cli는 `redis6-cli`로 입력해야한다.

## 서버 배포 완료!
앱에서 소셜 로그인까지 성공했다! 데이터베이스에 데이터도 들어오고 레디스에 토큰이 저장되는 것도 확인했다. 앞으로 flutter에서 api 통신은 서버랑 직접적으로 하면 된다!  
AWS가 두려워서 시작조차 못하고 있었는데 생각보다 할만했다. 약 5일정도 걸렸는데 2주는 걸릴줄 알았다. 이 모든 영광을 스프링 부트와 aws로 혼자 구현하는 웹 서비스 작가님과 구글에게 돌립니다!!