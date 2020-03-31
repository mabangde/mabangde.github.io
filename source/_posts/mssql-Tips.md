---
title: mssql 找指定库指定表
date: 2019-08-29 20:34:08
tags: RedTeam
---

### 找出包含关键字段的库和表
```sql
declare @i int,@id int,@dbname varchar(255),@sql varchar(255)
    set @i = 6
    set @id=(select count(*) from master..sysdatabases)

create table #t (
    dbname varchar(255),
    tablename varchar(255),
    columnname varchar(255)
)

while (@i < @id)
    begin
        set @i = @i + 1;
        set @dbname = (select name from master..sysdatabases where dbid= @i)
        set @sql = 'use '+ @dbname+';insert [#t] select table_catalog,table_name,column_name from information_schema.columns where column_name like ''%pass%'' or column_name like ''%pwd%'' or column_name like ''%mail%'''
        exec (@sql)
        print @sql
    end

select * from #t
drop table #t

go
```

### 所有库中找某个表
```sql
DECLARE @SQL NVARCHAR(max)
 
SET @SQL = stuff((
            SELECT '
UNION
SELECT ' + quotename(NAME, '''') + ' as Db_Name, Name collate SQL_Latin1_General_CP1_CI_AS as Table_Name
FROM ' + quotename(NAME) + '.sys.tables WHERE NAME LIKE ''%'' + @TableName + ''%'''
            FROM sys.databases
            ORDER BY NAME
            FOR XML PATH('')
                ,type
            ).value('.', 'nvarchar(max)'), 1, 8, '')
 
--PRINT @SQL;
 
EXECUTE sp_executeSQL @SQL
    ,N'@TableName varchar(30)'
    ,@TableName = 'admin'
```


