FROM oracle11g:step1
LABEL MAINTAINER="howard chiang"

ADD dbca.rsp /software/oracle/dbca.rsp
ADD conf_finish.sql $ORACLE_HOME/config/scripts/conf_finish.sql

ADD create /tmp/install/create

