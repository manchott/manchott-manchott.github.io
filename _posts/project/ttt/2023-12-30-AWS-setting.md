---
layout: post
title: AWS 프리 티어 배포 환경 구성
sitemap: true
categories:
  - project
  - Table Top Tracker
hide_last_modified: true
---
**AWS 하나도 몰라서 삽질하며 설정한 기록**  

11월 초에 무슨 자신감인지 AWS 계정을 만들었고 1년 무료 사용에서 벌써 2개월이나 날려버렸다.  
너무 아깝기도 하고 이제 슬슬 서버를 건드려야 할 것 같아서 AWS 설정을 시작했다.  
[스프링 부트와 aws로 혼자 구현하는 웹 서비스](https://m.yes24.com/Goods/Detail/83849117) 책에 정말 잘 구현이 되어 있었어서 기본적으로 책을 참고했다. 하지만 19년도 발매된 책인만큼, 버전같은 건 내가 나름 근거를 찾아가며 바꾸었다.  

1. this ordered seed list will be replaced by the toc
{:toc}

# EC2 환경 구성
EC2(Elastic Compute Cloud)는 AWS에서 제공하는 성능, 용량 등을 유동적으로 사용할 수 있는 서버이다.  
아마존 서비스에서 EC2를 검색하고 인스턴스 시작을 통해 새로운 인스턴스를 만들 수 있다.  
## 인스턴스 생성
### 이름 및 태그
이전에는 이름 대신 태그가 있었던 것 같다. 이제는 이름 따로 태그 따로라서 이름만 잘 설정하면 되었다. 태그는 선택사항으로 보인다. 

### AMI 선택
AMI는 EC2 인스턴스를 시작하는 데 필요한 정보를 **이미지로 만들어 둔 것**이다. 인스턴스는 가상 머신에 운영체제 등을 설치할 수 있게 구워 넣은 이미지이다.  
책에서는 국내 자료가 아마존 리눅스 1이 많기 떄문에 Amazon Linux AMI 2018(AL1)을 선택했지만 이제는 다르다. Amazon Linux 2(AL2)가 나온 뒤에 이제는 Amazon Linux 2023 AMI(AL2023)까지도 나왔다. 찾아보니, AL2023을 쓰지 않을 이유가 없었다.
> Now, Amazon Linux 2023 (AL2023) combines the best of AL1 and AL2 with added security benefits and a predictable release cycle, buffered by scheduled long-term support.[출처](https://coralogix.com/blog/why-were-moving-amazon-linux-2023/)  

이전 AMI과의 [차이점에 유의하며](https://cmakkaya.medium.com/amazon-linux-2-vs-amazon-linux-2023-10-key-differences-we-need-to-know-to-avoid-errors-9d0aa4b2bd0d) 설정하면 될 것이라고 생각했기에 AMI는 Amazon Linux 2023 AMI으로 선택했다.  
![이미지](https://manchott.github.io/assets/img/AWS-setting/AL2023.png "AL2023")

### 인스턴스 유형
프리 티어로 하려면 애초에 **t2.micro**밖에 선택지가 없다.  
책에 나와있는 부가설명에 의하면 프리 티어는 월 750시간의 제한이 있는데 1대의 t2.micro만 사용하면 24일*31일=744시간이 되어서 딱 24시간을 사용할 수 있다고 한다.  
추후에 서비스가 더 커지면 다양한 사양을 도전해볼 수 있을 것 같다.

### 키 페어
인스턴스의 보안을 위한 열쇠라고 생각하면 된다. 로컬에서 서버로 접속할 때, 키의 유무를 확인하게 된다. 그렇기에 키를 잃어버리면 앞으로 서버에 절대 접속할 수 없어지고, 유출이 된다면 누군가가 내 서버에서 열심히 코인을 채굴할 것이니 **안전한 곳에 보관**해야 한다.  

새 키 페어를 생성할 때 pem과 ppk 형식중에 선택할 수 있는데, 윈도우면 ppk를 선택하는게 편한 것 같다. 맥은 터미널에서 바로 서버에 연결할 수 있는데 윈도우는 **PuTTY**라는 프로그램을 사용해야 하고, 거기서는 ppk 형식만 지원하기 때문이다. 나는 맥과 윈도우 환경 모두 사용하기 때문에 **pem**으로 받았다.  

키 페어 생성 버튼을 누르면 바로 키가 다운로드 받아진다. 로컬의 안전한 곳에 저장했다.  
![이미지](https://manchott.github.io/assets/img/AWS-setting/key-pair.png "키 페어 생성")

### 네트워크 설정
오른쪽의 **편집** 버튼을 누르면 세부 설정이 가능하다. 다른 것은 건드릴 필요가 없고 보안 그룹을 설정하면 된다.  
보안 그룹 이름, 설명을 유의미하게 수정하고 인바운드 유형 규칙을 설정했다. 인바운드 유형 규칙은 3개를 설정한다.
1. SSH(22번 포트): AWS EC2 터미널로 접속하는 길. pem 키가 있으니 모든 IP를 허용하는 경우가 있는데 만약 pem키가 노출되면 끝장이다. 고로 소스 유형은 내 IP로 설정한다. 만약 집이 아니라 다른 곳에서 일한다면 SSH에 IP를 추가하면 된다.
2. 8080번 포트: 프로젝트의 기본 포트. 8080은 열어두어도 문제가 없다.
3. HTTPS(443번 포트): 웹 트래픽을 암호화하는 프로토콜. 책에는 따로 설명이 없어서 찾아보니 HTTPS를 추가하는 것은 보안 측면에서 중요한 조치라고 한다. 내가 웹을 할지는 모르겠지만 일단 설정했다. 이 포트도 열어두어도 문제가 없다.  

소스 유형에 위치 무관을 하면 경고가 뜨지만 상관없으니 진행했다.  
![이미지](https://manchott.github.io/assets/img/AWS-setting/security_group.png "보안 그룹 설정")

### 스토리지 구성
바로 아래에 설명에도 나와있듯이 프리 티어 최대 한도인 30GiB까지 늘여줬다.  
![이미지](https://manchott.github.io/assets/img/AWS-setting/storage.png "스토리지 구성")

### 인스턴스 시작
고급 세부 정보에서는 건드릴 것이 없다. 인스턴스를 시작하면 인스턴스 탭에서 인스턴스를 확인할 수 있다.

### EIP 할당
인스턴스도 하나의 서버이기에 IP가 존재한다. 그러나 인스턴스를 재시작할 때마다 **새 IP가 할당된다.** 그렇기에 고정 IP(EIP)를 할당해야한다.  
AWS 왼쪽 메뉴에서 탄력적 IP를 선택하고 탄력적 IP 주소를 할당하면 IP 주소가 생성된다. 해당 주소를 선택한 뒤 작업에서 탄력적 IP 주소 연결을 하면 된다. EC2 인스턴스와 연결하면 끝. 굉장히 간단했다.  
탄력적 IP를 생성하고 나서 **EC2에 연결하지 않으면 비용이 발생하니 주의**!

### 인스턴스 사용시 주의사항
인스턴스를 중지하고 재사용하고 싶은 경우가 있을 것이다.  이 때 인스턴스 목록에서 인스턴스를 선택하고 인스턴스 상태를 바꿀 수 있는데 **인스턴스 종료는 인스턴스 삭제나 다름없다.** 영어로는 Terminate Instance라서 좀 더 와닿는데 한국어로 종료라서 그냥 컴퓨터 종료하고 다시 키는거라고 생각해서 종료했다가... 인스턴스가 말 그대로 종료되었다. 결국 이 과정을 다시 그대로 거쳐야했다,,,,, 다시 시작하고 싶다면 인스턴스 재부팅을 선택하자.  
![이미지](https://manchott.github.io/assets/img/AWS-setting/terminate_instance.png "인스턴스 종료")

## EC2 서버 접속하기
### Mac
맥은 터미널에 명령어를 입력해서 서버에 접속할 수 있다. 하지만 매번 긴 명령어를 치는건 귀찮으니 쉽게 ssh 접속을 할 수 있도록 설정했다.
1. pem 파일을 ~/.ssh 디렉토리로 복사  
`cp [pem 키가 있는 위치] ~/.ssh/`
2. pem 키의 권한 변경  
`chmod 600 ~/.ssh/[pem 키 이름]`
3. pem 키가 있는 ~/.ssh 디렉토리에 config 파일 생성  
`vim ~/.ssh/config`
4. Host 등록: 앞으로 접속할 키 값 등록  
~~~shell
Host [본인이 원하는 서비스명]
  HostName [EC2의 탄력적 IP 주소]
  User ec2-user
  IdentityFIle ~/.ssh/[pem 키 이름]
~~~
5. config 파일 권한 변경  
`chmod 700 ~/.ssh/config`
6. 서버 접속  
`ssh [config에 등록한 서비스명]`  
터미널에 `The authenticitiy of host [IP 주소] can't be established.` 라는 말이 뜨고 yes를 입력하면 EC2에 접속이 성공된다.  
이후 접속할 때 위의 명령어를 입력했을 때 `ssh: connect to host [IP 주소] port 22: Connection refused`이나 `ssh: connect to host [IP 주소] port 22: Operation timed out`이 뜨면 몇 번 더 명령어를 입력하니 연결 되었다.

### Windows
윈도우는 PuTTY라는 프로그램을 사용해야한다. [공식 사이트](https://www.putty.org/)에 접속해서 다운받을 수 있다. Package Files에서 실행파일을 받고, 만약 키를 pem으로 다운받았다면 스크롤을 내려서 `puttygen.exe`도 다운받아야한다. puttygen.exe에서 pem키를 ppk파일로 변환할 수 있다.  
Connection > SSH > Auth > Credentials에서 Private key를 입력하면 된다.

## EC2 서버 설정하기
### Java 설치하기
책과는 다르게 나는 자바 17을 사용했고 AL2023이라서 자바 설치 명령어가 달랐다.
~~~shell
# JDK 17
sudo dnf install java-17-amazon-corretto-devel
# Java version 확인
java -version
~~~

### 타임존 변경
~~~shell
sudo rm /etc/localtime

sudo ln -s /usr/share/zoneinfo/Asia/Seoul /etc/localtime
~~~

### Hostname 변경
이 단계가 굳이 필요한지는 모르겠다. 그러나 언젠간 여러 서버를 운영하게 될 수도 있으니 해서 나쁠건 없을 것 같기도 하다...  
책에 나와있는 방법은 통하지 않아서 [공식문서](https://docs.aws.amazon.com/ko_kr/AWSEC2/latest/UserGuide/set-hostname.html)를 참고하여 변경했다.  
1. cloud-init 설정 편집  
`sudo vi /etc/cloud/cloud.cfg`
2. preserve_hostname 설정 변경  
`preserve_hostname: true`
3. hostnamectl 명령으로 호스트 이름을 설정(퍼블릭 DNS 이름 없이 시스템 호스트 이름을 변경)  
`sudo hostnamectl set-hostname [새로운 호스트 이름].localdomain`  
4. /etc/hosts 파일을 열고 127.0.0.1로 시작되는 항목을 변경  
~~~shell  
sudo vim /etc/hosts
127.0.0.1 [새로운 호스트 이름].localdomain [새로운 호스트 이름] localhost4 localhost4.localdomain4
~~~

1. 인스턴스를 재부팅하여 새 호스트 이름을 적용  
`sudo reboot`
1. 호스트 이름이 업데이트되었는지 확인  
~~~shell
# 호스트 이름 확인 명령어
hostname
#결과
[ec2-user@[새로운 호스트 이름] ~]$ hostname
[새로운 호스트 이름].localdomain
~~~

EC2 서버 설정완료! 이제 AWS의 데이터베이스 서비스인 RDS를 생성하고 설정해야한다.

# RDS로 데이터베이스 환경 구축
RDS(Relational Database Service)는 AWS에서 지원하는 클라우드 기반 관계형 데이터베이스이다. 하드웨어 프로비저닝, 데이터베이스 설정, 패치 및 맥업 등 잦은 운영 작업을 자동화하여 개발자가 개발에 집중할 수 있게 지원하는 서비스이다.
아마존 서비스에서 RDS를 검색하고 데이터베이스를 생성할 수 있다.
## RDS 인스턴스 생성
### 엔진 옵션
내 서비스는 MariaDB를 기준으로 만들었으니 MariaDB를 선택했다. 이 때, 엔진 버전을 확인하고 기억해두자.

### 템플릿
프리 티어는 선택지가 없다... 무조건 프리 티어 템플릿을 선택한다.

### 설정
DB 인스턴스 식별자는 DB 인스턴스의 이름이다.  
자격 증명 설정에서 마스터 사용자 이름과 암호를 설정하고 꼭 잊지 말자. 잊어버리면 어떻게 될지는 모르겠지만 귀찮아질게 분명하다... 또한 사용자 이름 admin 비밀번호 admin 이렇게 하면 큰일날 수도 있으니 꼭 성실하게 설정해야 한다.  
![이미지](https://manchott.github.io/assets/img/AWS-setting/rds_config.png "RDS 인스턴스 설정")

### 인스턴스 구성, 스토리지
프리 티어를 선택했다면 딱히 건드릴게 없다.  
혹시 모르니 스토리지 자동 조정에서 `스토리지 자동 조정 활성화`를 비활성화해두었다. 임계값을 넘어가면 자동으로 스토리지를 늘리는 설정이다.

### 연결
퍼블릭 액세스를 `예`로 하고 후에 보안 그룹에서 정해진 IP만 접근하도록 막을 것이다.

### 데이터베이스 생성 완료
더이상 건드릴건 없다. 데이터베이스 생성을 누르면 생성될 때까지 시간이 좀 걸린다. 그 사이에 몇 가지 설정을 하면 된다.

## RDS 파라미터 설정하기
### 파라미터 그룹 생성
RDS 왼쪽의 메뉴에서 파라미터 그룹으로 들어가 파라미터 그룹을 생성한다. 위의 엔진 옵션에서 기억해둔 MariaDB 버전을 선택한다. 이름도 적절히 지어준다.
![이미지](https://manchott.github.io/assets/img/AWS-setting/parameter_group.png "파라미터 그룹 생성")

### 파라미터 편집
목록에서 파라미터 그룹을 선택하고 오른쪽의 편집 버튼으로 수정 가능하다.  
1. time_zone을 Asia/Seoul로 설정
2. char로 검색해서 나온 모든 항목은 `utf8mb4`로 변경. utf8mb4는 데이터베이스에 이모지도 저장할 수 있다.  
![이미지](https://manchott.github.io/assets/img/AWS-setting/char.png "char")
1. collation으로 검색해서 collation_connection, collation_server 항목 모두 `utf8mb4_general_ci`로 변경  
2. 책에서는 max_connections를 150개로 설정했는데 나는 그냥 기본 값으로 놔두었다.  

### 파라미터 그룹을 데이터베이스에 연결
RDS 왼쪽의 메뉴에서 데이터베이스로 들어가 생성 완료된 데이터베이스를 수정한다. 아래로 쭉 내리면 추가 구성에서 DB 파라미터 그룹을 변경할 수 있다.  
![이미지](https://manchott.github.io/assets/img/AWS-setting/db_parameter_group.png "DB 파라미터 그룹 변경")  
여기서 계속을 누르면 변경사항 요약이 나오는데 **즉시 적용**을 선택하고 DB 인스턴스를 수정해야한다. 아니면 수정사항이 새벽 5시에 반영된다...  
또한 파라미터가 제대로 반영되지 않는 경우가 있다(경험담). 타임존이 적용 안되어서 헤맸었는데 그냥 데이터베이스 재부팅 한두번 해주면 되는 것 같다.(한 번 했었는데 적용 안되었음)

## 로컬에서 RDS 접속하기
### RDS, 개인 PC, EC2 연동 설정
로컬에서 RDS에 접근하기 위해서는 RDS의 보안 그룹에 내 PC의 IP를 추가해야한다. EC2의 보안 그룹 설정과 마찬가지로 작업하는 장소가 바뀌면 RDS의 보안 그룹에 바뀐 IP를 등록해야한다.
1. EC2에 사용된 보안 그룹의 그룹 ID를 복사
2. RDS의 보안 그룹으로 들어가서 인바운드 규칙에 추가. 유형을 `MySQL/Aurora`로 선택하면 자동으로 3306 포트로 선택된다.
   1. 내 PC의 IP
   2. EC2의 보안 그룹 ID  
![이미지](https://manchott.github.io/assets/img/AWS-setting/db_security_group.png "DB 보안 그룹 설정")  

### Intellij에서 RDS 접속
먼저 RDS 정보 페이지로 들어가서 엔드포인트를 복사해둔다. `rds.amazonaws.com`로 끝나는 주소이다.  
나는 Intellij 유료 버전을 사용하고 있어서 기본 내장된 Database 기능을 사용했다.  
오른쪽 탭에 있는 데이터베이스 메뉴를 통해 DataSource를 추가했다.
1. Name: 이름 설정
2. Host: RDS의 엔드포인트
3. User와 Password: RDS 설정할 때 입력했던 마스터 User와 비밀번호
![이미지](https://manchott.github.io/assets/img/AWS-setting/intellij_database.png "인텔리제이 데이터베이스 설정")  

위의 과정이 끝나면 자동으로 Query Console이 켜진다. 여기에서 추가적으로 설정해야할 것이 있다.
~~~sql
# RDS 파라미터 그룹으로는 MariaDB에 적용 안되는 파라미터 설정
ALTER DATABASE [데이터베이스 이름]
CHARACTER SET = 'utf8mb4'
COLLATE = 'utf8mb4_general_ci';
~~~
그리고 `select @@time_zone, now();`을 통해서 타임존이 Asia/Seoul로 바뀌었는지 확인한다. 만약 안바뀌었다면 RDS를 재부팅해보면 아마도 될 것이다(나는 그랬다).

### EC2에서 RDS 접속
1. EC2 서버 접속
2. MySQL 설치(책과 다름)
~~~shell
sudo dnf install https://dev.mysql.com/get/mysql80-community-release-el9-1.noarch.rpm
sudo dnf install mysql-community-server
~~~
3. 계정, 비밀번호, 호스트 주소로 접속  
`mysql -u [계정이름] -p -h [호스트 주소]`  
엔터 치면 비밀번호를 입력하라고 한다. **만약 무한 로딩에 걸린다면 보안 그룹의 IP 설정을 다시 확인해본다**
4. `show databases;`를 통해 데이터베이스 목록 확인

# 마무리
EC2 서버, RDS 설정까지 완료하며 배포 환경을 구성했다. 다음으로는 서비스 배포다!
