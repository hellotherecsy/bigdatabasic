bigdatabasic
============


다음과
같이
>사용이
>>가능합니다.
>###어렵지 않습니다.
코드 표현방법
————-
“`
var i = 0
“`

var foo = “bar”
<html> </html>
##리스트 표현방법
+ 플러스
* 별
- 하이픈
###숫자리스트
1. 숫자
1. 숫자
### 링크
[블로그](teragoon.wordpress.com)
#주의
1. 이렇게 해야합니다.

“`
var i = 0;
“`
2. 이건 안됩니다.

var i = 0;
* 숫자 리스트로 하면 숫자가 유지됩니다.

#### 하둡 실행 계정으로 수행(hadoop)
"     "
    
wget   http://mirror.apache-kr.org/hadoop/common/hadoop-1.2.1/hadoop-1.2.1.tar.gz
"

#### 하둡 설치 폴더 생성 (root 계정)
"`
mkdir   /usr/local/hadoop/    
chown  hadoop:hadoop   /usr/local/hadoop/ 
"`
####압축  해제

mv   hadoop-1.2.1.tar.gz    /usr/local/hadoop/hadoop-1.2.1.tar.gz 
tar  xvzf  hadoop-1.2.1.tar.gz

vi  .bashrc 

export JAVA_HOME=/usr/local/jdk1.7.0_55

export HADOOP_INSTALL=/usr/local/hadoop/hadoop-1.2.1

export PATH=$PATH:/usr/local/hadoop/hadoop-1.2.1/bin:$JAVA_HOME/bin

conf/hadoop-env.sh 파일 수정 : 아래 내용 추가

vi  conf/hadoop-env.sh

# export JAVA_HOME=/usr/lib/j2sdk1.5-sun
export JAVA_HOME =/usr/local/jdk1.7.0_55


conf/core-site.xml 파일 수정 : 아래 내용 추가

vi  conf/core-site.xml

<property>	
  <name>fs.default.name</name>	
  <value>hdfs://192.168.122.201:9000</value>
</property>


conf/ hdfs-site.xml 파일 수정 : 아래 내용 추가

vi conf/ hdfs-site.xml 

<property>	<name>dfs.name.dir</name>	<value>/home/hadoop/namenode</value></property>
<property>	<name>dfs.data.dir</name>	<value>/home/hadoop/data1</value></property>
<property>	<name>fs.checkpoint.dir</name>	<value>/home/hadoop/checkpoint</value></property>
<property>	<name>dfs.replication</name>	<value>2</value></property>


conf/ mapred-site.xml 파일 수정 : 아래 내용 추가

vi  conf/ mapred-site.xml 

<property>	<name>mapred.job.tracker</name>	<value> 192.168.122.201:9001</value></property>


설정된 구성 파일을 모든 데이터 노드에 복사

scp  conf/*  hadoop@192.168.122.202:/usr/local/hadoop/hadoop-1.2.1/conf/






