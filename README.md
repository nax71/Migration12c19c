# Migration12c19c

# Install oracle binaries 
https://www.oracle.com/database/technologies/oracle-database-software-downloads.html

    ./runInstaller -silent -responseFile install/response/db_install.rsp

rsp_file example. 
--

    INVENTORY_LOCATION=/u01/app/oraInventory
    ORACLE_HOME=/u01/app/oracle/product/19.3.0/dbhome_1
    ORACLE_BASE=/u01/app/oracle
    oracle.install.db.InstallEdition=EE
    oracle.install.db.OSDBA_GROUP=dba
    oracle.install.db.OSOPER_GROUP=oper
    oracle.install.db.OSBACKUPDBA_GROUP=backupdba
    oracle.install.db.OSDGDBA_GROUP=dgdba
    oracle.install.db.OSKMDBA_GROUP=kmdba
    oracle.install.db.OSRACDBA_GROUP=racdba
    oracle.install.db.rootconfig.executeRootScript=true
    oracle.install.db.rootconfig.configMethod=ROOT
    oracle.install.db.config.starterdb.type=GENERAL_PURPOSE
    oracle.install.db.ConfigureAsContainerDB=false

rsp_file.rsp 
--

    oracle.install.responseFileVersion=/oracle/install/rspfmt_dbinstall_response_schema_v19.0.0
    oracle.install.option=INSTALL_DB_SWONLY
    UNIX_GROUP_NAME=dba
    INVENTORY_LOCATION=/oracle_base/oraInventory
    ORACLE_BASE=/oracle_base
    oracle.install.db.InstallEdition=EE
    oracle.install.db.OSDBA_GROUP=dba
    oracle.install.db.OSOPER_GROUP=dba
    oracle.install.db.OSBACKUPDBA_GROUP=dba
    oracle.install.db.OSDGDBA_GROUP=dba
    oracle.install.db.OSKMDBA_GROUP=dba
    oracle.install.db.OSRACDBA_GROUP=dba
    oracle.install.db.rootconfig.executeRootScript=false
    oracle.install.db.rootconfig.configMethod=
    oracle.install.db.rootconfig.sudoPath=
    oracle.install.db.rootconfig.sudoUserName=
    oracle.install.db.CLUSTER_NODES=
    oracle.install.db.config.starterdb.type=GENERAL_PURPOSE
    oracle.install.db.config.starterdb.globalDBName=
    oracle.install.db.config.starterdb.SID=
    oracle.install.db.ConfigureAsContainerDB=false
    oracle.install.db.config.PDBName=
    oracle.install.db.config.starterdb.characterSet=
    oracle.install.db.config.starterdb.memoryOption=false
    oracle.install.db.config.starterdb.memoryLimit=
    oracle.install.db.config.starterdb.installExampleSchemas=false
    oracle.install.db.config.starterdb.password.ALL=
    oracle.install.db.config.starterdb.password.SYS=
    oracle.install.db.config.starterdb.password.SYSTEM=
    oracle.install.db.config.starterdb.password.DBSNMP=
    oracle.install.db.config.starterdb.password.PDBADMIN=
    oracle.install.db.config.starterdb.managementOption=DEFAULT
    oracle.install.db.config.starterdb.omsHost=
    oracle.install.db.config.starterdb.omsPort=0
    oracle.install.db.config.starterdb.emAdminUser=
    oracle.install.db.config.starterdb.emAdminPassword=
    oracle.install.db.config.starterdb.enableRecovery=false
    oracle.install.db.config.starterdb.storageType=
    oracle.install.db.config.starterdb.fileSystemStorage.dataLocation=
	oracle.install.db.config.starterdb.fileSystemStorage.recoveryLocation=
    oracle.install.db.config.asm.diskGroup=


Upgrade database
--

    ORACLE_SID = CID 
    
    SQL> select count(*) from dba_objects where status='INVALID';
    
    $> /u01/app/oracle/product/12.2.0/dbhome_1/jdk/bin/java -jar /u01/app/oracle/product/19.0.0/dbhome_1/rdbms/admin/preupgrade.jar TERMINAL TEXT

or 

    $> /u01/app/oracle/product/12.2.0/dbhome_1/jdk/bin/java -jar /u01/app/oracle/product/19.0.0/dbhome_1/rdbms/admin/preupgrade.jar FILE DIR /home/oracle/CID/preupgrade
    
    . oraenv 
    CID


Verify tablespace sizes for upgrade
--

    SQL> SET ECHO ON;
    SQL> SET SERVEROUTPUT ON;
    SQL> EXECUTE DBMS_STATS.GATHER_DICTIONARY_STATS;
    SQL> PURGE DBA_RECYCLEBIN;

    @/home/oracle/CID/preupgrade/preupgrade_fixups.sql
    $> cat /home/oracle/CID/preupgrade/preupgrade.log

If Dataguard
--
-- on standby 
SQL> alter database recover managed standby cancel;
DGMGRL> disable fast_start failover
DGMGRL> disable configuration
SQL> select 'host cp ' || value || ' /tmp' as cmd from v$parameter where name like 'dg_broker_config_file%';
SQL> host ls /tmp/dr*.dat
SQL> alter system set dg_broker_start=false scope=both;

-- on primary
DGMGRL> disable fast_start failover
DGMGRL> disable configuration
SQL> select 'host cp ' || value || ' /tmp' as cmd from v$parameter where name like 'dg_broker_config_file%';
SQL> host ls /tmp/dr*.dat
SQL> alter system set dg_broker_start=false scope=both;
SQL> alter system set log_archive_dest_state_2 = defer


Stop LISTENER
--

    SQL> SHUTDOWN IMMEDIATE;
    
    $> cd /u01/app/oracle/product/12.2.0/dbhome_1/dbs
    $> cp orapwCID spfileCID.ora /u01/app/oracle/product/19.0.0/dbhome_1/dbs/
    $> ls -ltr /u01/app/oracle/product/19.0.0/dbhome_1/dbs/*CID*
    $> export ORACLE_HOME=/u01/app/oracle/product/19.0.0/dbhome_1
    $> export ORACLE_SID=CID
    $> PATH=/u01/app/oracle/product/19.0.0/dbhome_1/bin:$PATH; export PATH
    $> which sqlplus
    /u01/app/oracle/product/19.0.0/dbhome_1/bin/sqlplus
    $> sqlplus / as sysdba
    SQL> startup upgrade;
    SQL> select name,open_mode,cdb,version,status from v$database,v$instance;
    NAME      OPEN_MODE            CDB VERSION           STATUS
    --------- -------------------- --- ----------------- ------------
    CID       READ WRITE           NO  19.0.0.0.0        OPEN MIGRATE 


UPGRADE 
--

    $> cd /u01/app/oracle/product/19.0.0/dbhome_1/bin/
    $> ls -ltr dbupgrade
    ./dbupgrade
    
    export ORACLE_HOME=/u01/app/oracle/product/19.0.0/dbhome_1
    export ORACLE_SID=CID
    PATH=/u01/app/oracle/product/19.0.0/dbhome_1/bin:$PATH; export PATH
    which sqlplus
    sqlplus / as sysdba
    SQL> startup;
    SQL> select name,open_mode,cdb,version,status from v$database,v$instance;

    NAME      OPEN_MODE            CDB VERSION           STATUS
    --------- -------------------- --- ----------------- ------------
    CID       READ WRITE           NO  19.0.0.0.0        OPEN 

    SQL> col COMP_ID for a10
    SQL> col COMP_NAME for a40
    SQL> col VERSION for a15
    SQL> set lines 180
    SQL> set pages 999
    SQL> select COMP_ID,COMP_NAME,VERSION,STATUS from dba_registry;

Post upgrade
--

    $> cd /u01/app/oracle/product/19.0.0/dbhome_1/rdbms/admin
    sqlplus '/as sysdba'
    SQL> @utlrp.sql
    SQL> @utlrp.sql
    SQL> select count(*) from dba_objects where status='INVALID';
    SQL> @/home/oracle/$ORACLE_HOME/preupgrade/postupgrade_fixups.sql

Upgrade Timezone
--

    $> cd /u01/app/oracle/product/19.0.0/dbhome_1/rdbms/admin/
    $> ls -ltr utltz_countstats.sql utltz_countstar.sql utltz_upg_check.sql utltz_upg_apply.sql
    SQL> SELECT version FROM v$timezone_file;
       VERSION
    ----------
            26
    SQL> @/u01/app/oracle/product/19.0.0/dbhome_1/rdbms/admin/utltz_upg_check.sql
    SQL> @/u01/app/oracle/product/19.0.0/dbhome_1/rdbms/admin/utltz_upg_apply.sql
    SQL> SELECT version FROM v$timezone_file;

       VERSION
    ----------
            32
    
    SQL> @/u01/app/oracle/product/19.0.0/dbhome_1/rdbms/admin/utlusts.sql TEXT
    SQL> @/u01/app/oracle/product/19.0.0/dbhome_1/rdbms/admin/catuppst.sql
    SQL> @/home/oracle/CID/preupgrade/postupgrade_fixups.sql
    SQL> select count(*) from dba_objects where status='INVALID';
    SQL> show parameter COMPATIBLE
    SQL> ALTER SYSTEM SET COMPATIBLE = '19.0.0' SCOPE=SPFILE;
    
    SQL> col COMP_ID for a10
    SQL> col COMP_NAME for a40
    SQL> col VERSION for a15
    SQL> set lines 180
    SQL> set pages 999
    SQL> select COMP_ID,COMP_NAME,VERSION,STATUS from dba_registry;
    
    SQL> show parameter password

If Standby
--
-- primary
SQL> alter system set log_archive_dest_state_2 = enable;
-- stabndby
SQL> alter database recover managed standby database disconnect from session;
-- check rfs process 
SQL> select process, status sequence# from v$managed_standby;
-- waint until the dabases are sync

-- Enable Broker on primary and standby
$> cp /tmp/dr1$ORACLE_UNQNAME.dat $ORACLE_HOME/dbs
$> cp /tmp/dr2$ORACLE_UNQNAME.dat $ORACLE_HOME/dbs
SQL> alter system set dg_broker_start=true scope=both;
DGMGRL> show configuration
DGMGRL> enable configuration
DGMGRL> enable fast_start failover

END Upgrade
--
* Change TNS Entries 
* Change oratab
* Backup database







