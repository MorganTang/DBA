# DBA


-- $ cat dfiles.sql
set lines 1200 pages 1200
col file_name for a85
select file_id, file_name, bytes/1024/1024/1024 USED_GB, autoextensible, maxbytes/1024/1024/1024 MAX_GB from dba_data_files where tablespace_name like '%&TBS%' order by 1;

-- $ cat dfresize.sql -resize hwm
set heading off
set linesize 444
set echo on
set feedback off;
select ' '||Bytes/1024/1024/1024||' alter database datafile '''|| file_name||''' resize '|| ceil((nvl(hwm,1)* 8192)/1024/1024)||'m;' cmd
from dba_data_files a,
(select file_id, max(block_id+blocks-1) hwm
from dba_Extents group by file_id) b
where a.file_id=b.file_id(+)
and ceil(blocks*8192/1024/1024)-ceil((nvl(hwm,1)*8192)/1024/1024)>0   order by tablespace_name;
