# database

# Oracle

Oracle `VARCHAR2` only goes up to 4000 bytes. More than this, it's necessary to user a `CLOB`. 
This version uses `VARCHAR2(4000)`, so if it's necessary just change to a `CLOB`.


Dica para obter o DataSorce para o Oracle

https://docs.oracle.com/database/121/TDPJD/getconn.htm#TDPJD144


## Local Oracle Database for tests

### Oracle 12c R2

Oracle Database Server Docker Image contains the Oracle Database Server 12.2.0.1 Enterprise Edition running on Oracle Linux 7

https://hub.docker.com/
- Login on Oracle Database and registrate first;

Run:
```
docker login
docker run -d -it -p 1521:1521 --name oracle-db store/oracle/database-enterprise:12.2.0.1-slim
```

Connection from DBeaver:
- Host: localhost
- Port: 1521
- Database: ORCLCDB
- Type: SID
- Username: sys
- Password: Oradoc_db1
- Role: sysdba


The database server can be connected to by executing SQL*Plus,
```bash
docker exec -it oracle-db bash -c "source /home/oracle/.bashrc; sqlplus /nolog"
```

Status of the listener:
```bash
docker exec -it oracle-db bash -c "source /home/oracle/.bashrc; lsnrctl status"
```

Configuration files:
```bash
docker exec -it oracle-db bash -c "ls /u01/app/oracle/product/12.2.0/dbhome_1/admin/ORCLCDB/"
```

Allow commands to create user in sysdba session:
```sql
alter session set "_ORACLE_SCRIPT"=true;
```

Create QUICKFIX user:
```sql
create user QUICKFIX identified by QUICKFIX;
```

Grant all privileges to the user:
```
GRANT ALL PRIVILEGES TO QUICKFIX;
```

## DDL to create tables

#### sessions

```sql
CREATE TABLE sessions (
  beginstring VARCHAR2(8) NOT NULL,
  sendercompid VARCHAR2(64) NOT NULL,
  sendersubid VARCHAR2(64) NOT NULL,
  senderlocid VARCHAR2(64) NOT NULL,
  targetcompid VARCHAR2(64) NOT NULL,
  targetsubid VARCHAR2(64) NOT NULL,
  targetlocid VARCHAR2(64) NOT NULL,
  session_qualifier VARCHAR2(64) NOT NULL,
  creation_time TIMESTAMP NOT NULL,
  incoming_seqnum NUMBER(10) NOT NULL,
  outgoing_seqnum NUMBER(10) NOT NULL,
  PRIMARY KEY (beginstring, sendercompid, sendersubid, senderlocid,
               targetcompid, targetsubid, targetlocid, session_qualifier)
);
```


#### messages

```sql
CREATE TABLE messages (
  beginstring VARCHAR2(8) NOT NULL,
  sendercompid VARCHAR2(64) NOT NULL,
  sendersubid VARCHAR2(64) NOT NULL,
  senderlocid VARCHAR2(64) NOT NULL,
  targetcompid VARCHAR2(64) NOT NULL,
  targetsubid VARCHAR2(64) NOT NULL,
  targetlocid VARCHAR2(64) NOT NULL,
  session_qualifier VARCHAR2(64) NOT NULL,
  msgseqnum NUMBER(10) NOT NULL,
  message VARCHAR2(4000) NOT NULL,
  PRIMARY KEY (beginstring, sendercompid, sendersubid, senderlocid,
               targetcompid, targetsubid, targetlocid, session_qualifier, msgseqnum)
);
```

#### messages_log

```sql

CREATE TABLE messages_log (
id NUMBER GENERATED ALWAYS AS IDENTITY INCREMENT BY 1 START WITH 1 CACHE 1000,
time TIMESTAMP NOT NULL,
beginstring VARCHAR2(8) NOT NULL,
sendercompid VARCHAR2(64) NOT NULL,
sendersubid VARCHAR2(64) NOT NULL,
senderlocid VARCHAR2(64) NOT NULL,
targetcompid VARCHAR2(64) NOT NULL,
targetsubid VARCHAR2(64) NOT NULL,
targetlocid VARCHAR2(64) NOT NULL,
session_qualifier VARCHAR2(64) NOT NULL,
text VARCHAR2(4000) NOT NULL,
PRIMARY KEY (id));
```

#### event_log

```sql

CREATE TABLE event_log (
id NUMBER GENERATED ALWAYS AS IDENTITY INCREMENT BY 1 START WITH 1 CACHE 1000,
time TIMESTAMP NOT NULL,
beginstring VARCHAR2(8) NOT NULL,
sendercompid VARCHAR2(64) NOT NULL,
sendersubid VARCHAR2(64) NOT NULL,
senderlocid VARCHAR2(64) NOT NULL,
targetcompid VARCHAR2(64) NOT NULL,
targetsubid VARCHAR2(64) NOT NULL,
targetlocid VARCHAR2(64) NOT NULL,
session_qualifier VARCHAR2(64),
text VARCHAR2(4000) NOT NULL,
PRIMARY KEY (id));
```


### DML Data Manipulation Language

```sql
SELECT * FROM QUICKFIX.SESSIONS;

SELECT * FROM QUICKFIX.MESSAGES ORDER BY MSGSEQNUM  DESC;

SELECT * FROM QUICKFIX.EVENT_LOG ORDER BY ID DESC;

SELECT * FROM QUICKFIX.MESSAGES_LOG ORDER BY ID DESC;
```

