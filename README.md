bigdatabasic
============

#### 최초 접속 후 (root / bigdata) 3대 VM에 동일한 작업 반복
>##### hadoop 계정 생성

    adduser  hadoop
    passwd  hadoop
    패스워드 입력 :

>##### java 확인 및 다운로드 설치

    java –version 
        
    wget --no-check-certificate -c --header "Cookie: oraclelicense=accept-securebackup-cookie"  http://download.oracle.com/otn-pub/java/jdk/7u55-b13/jdk-7u55-linux-x64.tar.gz
    
    mv jdk-7u55-linux-x64.tar.gz?AuthParam=xxxxxxxxxxxxxxxx   /usr/local/jdk-7u55-linux-x64.tar.gz
    cd  /usr/local
    tar  -xzvf  jdk-7u55-linux-x64.tar.gz

>##### java 링크

    which  java
    unlink   /usr/bin/java  (기존에 연결된 링크가 있다면 연결 해제)
    
    ln -s  /usr/local/jdk1.7.0_55/bin/java   /usr/bin/java  (새로 설치한 자바로 링크)
    
    java  -version


>##### 하둡 설치 폴더 생성 

    mkdir   /usr/local/hadoop/
    chown  hadoop:hadoop   /usr/local/hadoop/ 


========

hadoop 계정으로 이동 후

==============

#### ssh 설치 : name node에서 수행 
>##### ssh key 생성 (ssh-keygen) 

    su - hadoop
    
    cd ~
    ls  -la 
    
    ssh-keygen  
    
>##### ssh public key 복사 
    
    ssh-copy-id  -i  hadoop@bigdata01-01  (namenode에서 1번 데이터 노드로 접속 위해)
    ssh-copy-id  -i  hadoop@bigdata01-02  (namenode에서 2번 데이터 노드로 접속 위해)
    ssh-copy-id  -i  hadoop@bigdata01-03  (namenode에서 3번 데이터 노드로 접속 위해) 
    
    
    ssh bigdata01-02  (패스워드 없이 접속 가능한 지 확인)

#### hadoop 설치 : name node에만 설치 후 나중에 전체 내용 복사
>##### 하둡 패키지 다운로드 

    su - hadoop
    
    wget   http://mirror.apache-kr.org/hadoop/common/hadoop-1.2.1/hadoop-1.2.1.tar.gz

    mv   hadoop-1.2.1.tar.gz    /usr/local/hadoop/hadoop-1.2.1.tar.gz 
    cd   /usr/local/hadoop
    tar  xvzf  hadoop-1.2.1.tar.gz

#### .bashrc 설정 : Windows의 환경 변수와 동일 기능
>##### vi  ~/.bashrc 

    export JAVA_HOME=/usr/local/jdk1.7.0_55
    export HADOOP_INSTALL=/usr/local/hadoop/hadoop-1.2.1
    export PATH=$PATH:/usr/local/hadoop/hadoop-1.2.1/bin:$JAVA_HOME/bin
    
>##### ~/.bashrc 복사

    scp  ~/.bashrc  bigdata01-02:
    scp  ~/.bashrc  bigdata01-03:

#### HADOOP 설정 파일 수정
>##### HADOOP_HOME으로 이동
 
    cd  /usr/local/hadoop/hadoop-1.2.1


>##### vi  conf/hadoop-env.sh

    # export JAVA_HOME=/usr/lib/j2sdk1.5-sun
    export JAVA_HOME=/usr/local/jdk1.7.0_55


>##### vi  conf/core-site.xml

    <property>	
      <name>fs.default.name</name>	
      <value>hdfs://192.168.122.201:9000</value>
      <description>IP는 각자의 네임노드 IP를 쓰세요. 192.168.122.201라고 쓰시면 앙대요</description>
    </property>


>##### vi conf/hdfs-site.xml 

    <property>	
      <name>dfs.name.dir</name>	
      <value>/home/hadoop/namenode</value>
    </property>
    <property>	
      <name>dfs.data.dir</name>	
      <value>/home/hadoop/data1</value>
    </property>
    <property>	
      <name>fs.checkpoint.dir</name>	
      <value>/home/hadoop/checkpoint</value>
    </property>
    <property>	
      <name>dfs.replication</name>	
      <value>2</value>
    </property>


>##### vi  conf/mapred-site.xml 

    <property>	
      <name>mapred.job.tracker</name>	
      <value>192.168.122.201:9001</value>
      <description>IP는 각자의 네임노드 IP를 쓰세요. 192.168.122.201라고 쓰시면 앙대요</description>
    </property>

>##### vi conf/masters 

    bigdata01-02

>##### vi  conf/slaves 

    bigdata01-02
    bigdata01-03


#### 설정된 구성 파일을 모든 데이터 노드에 복사

    scp  -r  /usr/local/hadoop/hadoop-1.2.1  bigdata01-02:/usr/local/hadoop/
    scp  -r  /usr/local/hadoop/hadoop-1.2.1  bigdata01-03:/usr/local/hadoop/


bigdatabasic (Ambari)
============

#### Ambari 설치 사전 설정

>##### password-less 설정 

    ssh-keygen 후 엔터키, 엔터키, 엔터키
    ssh-copy-id -i .ssh/id_rsa.pub <각 3개 host> 후 연결 확인
    
>##### ntp 설정(minimal 버전에는 추가 설치가 필요함)

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
  
>##### openssl upgrade (1.0.1e-15 에서 1.0.1e-e16 으로)

    yum upgrade openssl


#### Ambari 서버 설치 - 1번 서버에서 수행
>##### Repos 설정

    wget http://public-repo-1.hortonworks.com/ambari/centos6/1.x/updates/1.6.0/ambari.repo
    cp ambari.repo /etc/yum.repos.d
    yum repolist 해서 Ambari 1.x, HDP-UTILS 가 나오는지 확인

>##### Ambari Install

    yum install ambari-server
    ambari-server setup 후 엔터, 엔터, JDK 설정 시 3. Custom JDK 선택 후 /usr/java/jdk 적어준다. 그 뒤 엔터, 엔터

>##### Ambari Start

    ambari-server start
    ps -ef | grep Ambari 로 프로세스 확인    


#### Ambari 서버 설치 - 1번 서버에서 수행
>##### Log into Amabari
  - 브라우저에서 http://{ambari-server}:8080 혹은 ssh tunneling 이 되어 있다면 http://localhost:8080 으로 접속 한다.
  - admin/admin 으로 접속

  - cluster이름을 넣어준다. (예: bigdata10)
  - HDP 2.1 선택

  - Target Host에는 3개 hostname으로 다 넣어준다. 
    (예: bigdata10-01 
         bigdata10-02 
         bigdata10-03)

  -  SSH Private Key에는 ambari-server가 설치된 서버(1번 서버)의 ~/.ssh/id_rsa 파일 내용을 copy and paste 로 넣어준다.

  - 필요한 서비스 선택: HDFS, YARN, Hive, Tez, Nagios, Zookeeper 등 선택
  
  - Assign Masters
    - 1번 서버: NameNode, History Server, Resource Manager, Zookeeper Server, 
    - 2번 서버: SNameNode, App Timeline Server, Nagios Server, Zookeeper Server
    - 3번 서버: Ganglia Server, Zookeeper Server

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
    
    
    
    
    
    
