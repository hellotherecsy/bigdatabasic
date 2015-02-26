bigdatabasic (Ambari)
============

#### Ambari 설치 사전 설정

>##### password-less 설정 

    ssh-keygen 후 엔터키, 엔터키, 엔터키
    ssh-copy-id -i .ssh/id_rsa.pub <각 3개 host> 후 연결 확인
    
>##### ntp 설정(minimal 버전에는 추가 설치가 필요함)
 : 01 ~ 03 모두 실행 
    service ntpd restart
    chkconfig ntpd on


>##### ntp 설정 (설정 오류시)

    yum install ntp
    vim /etc/ntp.conf 후
    -- 1번 서버에서 기존 server 들은 주석 처리 후 아래를 추가
      server 0.kr.pool.ntp.org
      server 1.asia.pool.ntp.org
      server 2.asia.pool.ntp.org
    -- 2,3번 서버에서는 마찬가지로 다른 server 들은 주석 처리 후 1번 서버 IP혹은 hostname 추가
      server bigdata10-01 (1번 서버 hostname)

>##### selinux disable 

    setenforce 0
    vim /etc/yum/pluginconf.d/refresh-packagekit.conf 후 enabled=0 으로 수정

>##### iptables off

    chkconfig iptables off
    /etc/init.d/iptables stop
  
>##### openssl upgrade (1.0.1e-15 에서 1.0.1e-e16 으로) --> e30 )

    yum upgrade openssl


#### Ambari 서버 설치 - 1번 서버에서 수행
>##### Repos 설정

    wget http://public-repo-1.hortonworks.com/ambari/centos6/1.x/updates/1.6.0/ambari.repo
    wget http://public-repo-1.hortonworks.com/ambari/centos6/1.x/updates/1.7.0/ambari.repo
    cp ambari.repo /etc/yum.repos.d
    yum repolist 해서 Ambari 1.x가 나오는지 확인

>##### Ambari Install

    yum install ambari-server
    -- y 선택
    ambari-server setup
    -- 1) Ambari 계정 ( default : N ) 
    -- 2) JDK ( default : 1 ) 
    -- 2-1 ) download : (default : y ) 
    -- 3) database configuration ( default : n ) 
    -- 엔터, 엔터, 엔터, 엔터
    
>##### Ambari Start

    ambari-server start
    ps -ef | grep Ambari 로 프로세스 확인    

#### Ambari 서버 설치 - 1번 서버에서 수행
>##### Log into Amabari
  - 브라우저에서 http://{ambari-server}:8080 혹은 ssh tunneling 이 되어 있다면 http://localhost:8080 으로 접속 한다.
  - admin/admin 으로 접속

  - cluster이름을 넣어준다. (ex: bigdata10)
  - HDP 2.2 선택

  - Target Host에는 3개 hostname으로 다 넣어준다. 
    (예: bigdata10-01 
         bigdata10-02 
         bigdata10-03)

  -  SSH Private Key에는 ambari-server가 설치된 서버(01번 서버)의 ~/.ssh/id_rsa 파일 내용을 copy and paste 로 넣어준다.
  ex) $> cat ~/.ssh/id_rsa
       ( 출력내용을 복사하여 붙여넣기

  - 필요한 서비스 선택: HDFS, YARN, Hive, Tez, Nagios, Zookeeper 등 선택
  
  - Assign Masters
    - 1번 서버: NameNode, History Server, Resource Manager, Ganglia Server, Zookeeper Server, 
    - 2번 서버: SNameNode, App Timeline Server, Nagios Server, Zookeeper Server
    - 3번 서버: Hive Server, Zookeeper Server

  - Assign Slaves and Clients
    - 2번, 3번 서버에 DataNode, NodeManager, Client 설치 한다.

  - Customize Services 
    - Hive 탭에서 Database password 설정
    - Nagios 탭에서 Nagios Admin Passwrd와 Hadoop Admin email 설정
    - Yarn 탭에서 yarn.timeline-service.store-class=org.apache.hadoop.yarn.server.timeline.LeveldbTimelineStore


실습 파일 다운로드 
============

#### 1번 서버에 PuTTy접속
>##### Log into hdfs

    su - hdfs
    
>##### 실습 파일 다운로드

    wget http://data.dft.gov.uk.s3.amazonaws.com/road-accidents-safety-data/Stats19-Data2005-2013.zip
    unzip Stats19-Data2005-2013.zip
    
>##### HDFS 폴더 생성

    hadoop fs -mkdir /upload
    hadoop fs -mkdir /upload/acc
    hadoop fs -mkdir /upload/cas
    hadoop fs -mkdir /upload/veh
    
>##### HDFS로 파일 복사

    hadoop fs -copyFromLocal  Accidents0513.csv  /upload/acc/
    hadoop fs -copyFromLocal  Casualties0513.csv  /upload/cas/
    hadoop fs -copyFromLocal  Vehicles0513.csv  /upload/veh/
    
