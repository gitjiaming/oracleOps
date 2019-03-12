# Oracle 11 备份脚本

该脚本功能包括

1. 自动生成备份路基，可以自定义；
2. 自动创建dump\_dir；
3. 自动识别oracle环境变量，有时安装了oracle client会对ORACLE\_HOME和path产生影响；
4. 自动保留7天备份数据；
5. 需手动，自己把脚本添加到windows定时任务中（怎么添加自己百度）

      我的文件名称db\_backup.bat

1. ```
   cd /d %~dp0

   rem 设置数据文件备份路径，可以手动修改
   Set backupDir=e:\oradata_backup

   rem 设置数据文件名称前缀
   Set filenamePrefix=sop_week_

   rem 设置数据库用户
   Set schema=sop

   rem 设置oracle的bin默认路径
   set ORACLE_HOME=E:\app\%USERNAME%\product\11.2.0\dbhome_1
   set path=%ORACLE_HOME%\BIN;%path%

   rem 检查是否已经创建了路径backupDir
   if not exist %backupDir% (
     md %backupDir%
    )

   rem 检查oracle中是否已经设置了数据文件路径backupDir
   (
   echo connect / as sysdba
   echo select rownum from dba_directories where directory_name='SOP_BACKUP_DIR';
   echo quit;
   ) | sqlplus -s -l /nolog >dbrow.txt


   set ifexist=0
   for /f %%i in (dbrow.txt) do @set ifexist=%%i

   echo current usered:"%ifexist%"
   del dbrow.txt

   if %ifexist% == 1 GOTO :backup

   rem ---------------- cteate dump directory ----------
   (
   echo connect / as sysdba
   echo create directory SOP_BACKUP_DIR as '%backupDir%';
   echo grant read,write on directory SOP_BACKUP_DIR to %schema% ;
   echo quit;
   ) | sqlplus -s -l /nolog

   :backup

   rem 计算出今天是周几，把汉字转为数字
   set no=%date:~12,13%
   if %no%==一 set kk=1
   if %no%==二 set kk=2
   if %no%==三 set kk=3
   if %no%==四 set kk=4
   if %no%==五 set kk=5
   if %no%==六 set kk=6
   if %no%==日 set kk=7
   Del %backupDir%%filenamePrefix%%kk%.log
   Del %backupDir%%filenamePrefix%%kk%.dmpdp

   expdp \"/ as sysdba\" schemas=%schema% job_name=sop_EXPORT exclude=table:\" in ('T_ALARM_DETAILS','T_ALARM_SMART','T_INDICATOR_SIGNAL' )\" directory=SOP_BACKUP_DIR dumpfile=%filenamePrefix%%kk%.dmpdp logfile=%filenamePrefix%%kk%.log  compression=ALL parallel=4

   rem  查看导出进度  job_name=sop_EXPORT
   rem  select job_name,state from dba_datapump_jobs;
   rem  expdp \"/ as sysdba\" attach=sop_EXPORT
   rem  Export> stop_job=immediate 

   ```



