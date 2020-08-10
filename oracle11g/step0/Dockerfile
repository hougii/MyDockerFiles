FROM centos:centos7
LABEL MAINTAINER="howard chiang"

RUN yum upgrade -y
RUN yum install -y oracle-rdbms-server-11gR2-preinstall unzip sysstat smartmontools vim dos2unix
RUN groupadd oinstall && groupadd dba && useradd -g oinstall -G dba -m oracle
RUN mkdir -p /software/oracle /u01/oracle /u01/oracle/product/11.2.0/db_1 /u01/oracle/dpdump /u01/oracle/oradata/mydata /u01/oracle/oradata/datafile \
 && chown oracle:oinstall -R /u01 && chmod 755 -R /u01/oracle

ADD p13390677_112040_Linux-x86-64_1of7.zip /software/oracle/p13390677_112040_Linux-x86-64_1of7.zip
ADD p13390677_112040_Linux-x86-64_2of7.zip /software/oracle/p13390677_112040_Linux-x86-64_2of7.zip
RUN cd /software/oracle && unzip p13390677_112040_Linux-x86-64_1of7.zip && unzip p13390677_112040_Linux-x86-64_2of7.zip && rm *.zip

ENV ORACLE_BASE /u01/oracle
ENV ORACLE_HOME /u01/oracle/product/11.2.0/db_1

ADD db_install.rsp /software/oracle/db_install.rsp
ADD install /software/oracle/install
RUN dos2unix /software/oracle/install