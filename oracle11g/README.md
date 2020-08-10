# Oracle Database 11g Release 2(11.2.0.4.0)
---





---

- ORCL database on port 1521
- Enterprise Manager on HTTPS port 1158

## 環境面資訊

```shell
# OS 版本使用的Docker Image
centos:centos7
# Oracle 的版本
Oracle Database 11g Release 2(11.2.0.4.0)	#這官方來源載不到，向各DBA人員取得。
	p13390677_112040_Linux-x86-64_1of7.zip
	p13390677_112040_Linux-x86-64_2of7.zip


```

群組及使用者

```shell
#建立群組:oinstall + dba
groupadd oinstall
groupadd dba
#建立使用者：oracle (並加入上述群組)
useradd -g oinstall -G dba -m oracle	#-g=主要群組 -G=隸屬多個群組
```

Oracle 設定資訊

```shell
SID=ldrdev
#ORACLE_BASE 
/u01/oracle
#ORACLE_HOME 
/u01/oracle/product/11.2.0/db_1
```





## Build

#### Step 0
**目標：**

---

- 設定好Group / User
- 安裝檔放至目標位置並解壓縮。
- 設定好ORACLE_BASE 、ORACLE_HOME

最終IMAGES大小： `5.69G`

---

調整以下的檔案內容，目前範例值如下：

##### update file : db_install.rsp

此檔案為安裝時所需的Repsonse file，修正以下區段內容為自已所需資訊：

```shell
oracle.install.option=INSTALL_DB_SWONLY
ORACLE_HOSTNAME=twqcasdb01	#這是目前機器的hostName
UNIX_GROUP_NAME=oinstall
INVENTORY_LOCATION=/u01/oracle/oraInventory
SELECTED_LANGUAGES=en
ORACLE_HOME=/u01/oracle/product/11.2.0/db_1
ORACLE_BASE=/u01/oracle
oracle.install.db.InstallEdition=EE
oracle.install.db.isCustomInstall=false
oracle.install.db.DBA_GROUP=dba
oracle.install.db.OPER_GROUP=oinstall
oracle.install.db.CLUSTER_NODES=
oracle.install.db.config.starterdb.type=GENERAL_PURPOSE
oracle.install.db.config.starterdb.globalDBName=ldrdev #(!!你要設定的DBName)
oracle.install.db.config.starterdb.SID=ldrdev   #【!!!要改】(!!設定你要的SID值)
......
oracle.install.db.config.starterdb.fileSystemStorage.dataLocation=/oradata	#(!!儲存位置)

```

##### update file : Dockerfile

這部份主要是目錄所在位置資訊，可調整為自已所需 (但後續相關的要一併修改)

```shell
#我會將安裝資訊放至
/software/oracle/
	#install rsp 也會放至此目錄下
	/software/oracle/db_install.rsp
	
#ORACLE_BASE 
/u01/oracle
#ORACLE_HOME 
/u01/oracle/product/11.2.0/db_1

#Dbdump File 預計位置 (預計採用Volumes所要替換的位置)
/u01/oracle/dpdump

#datafile 預計位置 (預計採用Volumes所要替換的位置)
/u01/oracle/oradata/datafile

```



##### 執行Build

1. get oracle install zip files.

2. Put the 2 zip files in the `step1` directory

3. `cd` to the `oracle11g` repo directory

  ```shell
  cd oracle11g
  ```

4. `$ docker build -t oracle11g:step0 step0`  或 `docker build --pull --rm -f "step0\Dockerfile" -t oracle11g:step0 "step0"`

  ```shell
  docker build -t oracle11g:step0 step0
  ```

5. 完成step0。

**注意**：若都是Local Repository操作的話，不要使用--pull (VSCode預設是使用pull的)，不然接下來的DockerFile > FROM 都會至遠端check；

所以將造成接下來的docker build 會得到此Error訊息 `pull access denied for oracle11g, repository does not exist or may require 'docker login': denied: requested access to the resource is denied `



##### 問題處理

```shell
# 錯誤訊息
/bin/sh: /software/oracle/install: /bin/bash^M: bad interpreter: No such file or directory
The command '/bin/sh -c /software/oracle/install' returned a non-zero code: 126
```

這是編碼的問題，Window下的編碼，在Linux上會有問題  ( https://is.gd/u9tOce )

解決方式：

1. 在yum install 補上 `dos2unix` ；

2. 將DOS 轉為UNIX： `dos2unix /software/oracle/install`

   (此段會補進docker file下)



#### Step 1

**目標：**

---

- 手動啟用images，執行安裝動作
- 最後commit image ：oracle11g:installed`
- 最終images大小 9.6GB

---

##### 啟動images

```shell
#啟動
docker run --privileged  --name step1 -itd oracle11g:step0 bash
#進入linux
docker exec -it step1

```

##### 執行cmd

```shell
#執行
sh -c /software/oracle/install
#(開始安裝oracle)
```



##### commit container

```shell
docker commit step1 oracle11g:installed
```



# (調整到這~未完待續)



#### Step 2
1) `$ docker build -t oracle-11g:step2 step2`

2) `$ docker run --privileged -ti --name step2 oracle-11g:step2 bash`

3) ` # /tmp/intall/create` (takes about 15m)

```
[root@3cccfeafca0c /]# /tmp/install/create
Mon Oct 26 10:44:29 UTC 2015
Creating listener...   

Parsing command line arguments:
    Parameter "silent" = true
    Parameter "responsefile" = /tmp/install/database/response/netca.rsp
Done parsing command line arguments.
Oracle Net Services Configuration:
Profile configuration complete.
Oracle Net Listener Startup:
    Running Listener Control:
      /opt/oracle/product/11.2.0/dbhome_1/bin/lsnrctl start LISTENER
    Listener Control complete.
    Listener started successfully.
Listener configuration complete.
Oracle Net Services configuration successful. The exit code is 0
Mon Oct 26 10:44:30 UTC 2015
Creating database...
/bin/cat: /proc/sys/net/core/wmem_default: No such file or directory
/bin/cat: /proc/sys/net/core/wmem_default: No such file or directory
/bin/cat: /proc/sys/net/core/wmem_default: No such file or directory
Copying database files
1% complete
3% complete
11% complete
18% complete
26% complete
37% complete
Creating and starting Oracle instance
40% complete
45% complete
50% complete
55% complete
56% complete
60% complete
62% complete
Completing Database Creation
66% complete
70% complete
73% complete
85% complete
96% complete
100% complete
Look at the log file "/opt/oracle/cfgtoollogs/dbca/orcl/orcl.log" for further details.

Mon Oct 26 10:51:43 UTC 2015
Creating password file...

Mon Oct 26 10:51:43 UTC 2015
Running catalog.sql...

Mon Oct 26 10:53:17 UTC 2015
Running catproc.sql...

Mon Oct 26 10:58:16 UTC 2015
Running pupbld.sql...

Finalizing install and shutting down the database...

SQL*Plus: Release 11.2.0.1.0 Production on Mon Oct 26 10:58:16 2015

Copyright (c) 1982, 2009, Oracle.  All rights reserved.


Connected to:
Oracle Database 11g Enterprise Edition Release 11.2.0.1.0 - 64bit Production
With the Partitioning, OLAP, Data Mining and Real Application Testing options

Database closed.
Database dismounted.   
ORACLE instance shut down.
Disconnected from Oracle Database 11g Enterprise Edition Release 11.2.0.1.0 - 64bit Production
With the Partitioning, OLAP, Data Mining and Real Application Testing options

Mon Oct 26 10:58:30 UTC 2015
Done! Commit the container now.
```
4) ` # exit`

5) `$ docker commit step2 oracle-11g:created`

#### Step 3
1) `$ docker build -t oracle-11g step3`

## Run
Create and run a container named oracle-11g with **mandatory hostname *oradb11g***:
```
$ docker run --privileged -dP -h oradb11g --name oracle-11g oracle-11g
```

## Connect
- `$ sqlplus sys/change_on_install as sysdba`
- `$ sqlplus system/manager`

## License
[GNU Lesser General Public License (LGPL)](http://www.gnu.org/licenses/lgpl-3.0.txt) for the contents of this GitHub repo; for Oracle's database software, see their [Licensing Information](http://docs.oracle.com/cd/E11882_01/license.112/e47877/toc.htm)
