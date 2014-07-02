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

#### hadoop 설치: name node에만 설치 후 나중에 전체 내용 복사

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






