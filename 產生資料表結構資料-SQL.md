SQL結構ToModel_工具

[TOC]

參考文章：https://stackoverflow.com/questions/5873170/generate-class-from-database-table



##### 產生資料表結構資料

```sql
--USE 'DBNAME';
SELECT a.TABLE_SCHEMA +'.'+a.TABLE_NAME   as 表格名稱  
       ,b.COLUMN_NAME                     as 欄位名稱
	   ,b.ORDINAL_POSITION 
       ,b.DATA_TYPE                       as 資料型別,  
	    case b.DATA_TYPE 
            when 'bigint' then 'long'
            when 'binary' then 'byte[]'
            when 'bit' then 'bool'
            when 'char' then 'string'
            when 'date' then 'DateTime'
            when 'datetime' then 'DateTime'
            when 'datetime2' then 'DateTime'
            when 'datetimeoffset' then 'DateTimeOffset'
            when 'decimal' then 'decimal'
            when 'float' then 'float'
            when 'image' then 'byte[]'
            when 'int' then 'int'
            when 'money' then 'decimal'
            when 'nchar' then 'string'
            when 'ntext' then 'string'
            when 'numeric' then 'decimal'
            when 'nvarchar' then 'string'
            when 'real' then 'double'
            when 'smalldatetime' then 'DateTime'
            when 'smallint' then 'short'
            when 'smallmoney' then 'decimal'
            when 'text' then 'string'
            when 'time' then 'TimeSpan'
            when 'timestamp' then 'DateTime'
            when 'tinyint' then 'byte'
            when 'uniqueidentifier' then 'Guid'
            when 'varbinary' then 'byte[]'
            when 'varchar' then 'string'
            else 'UNKNOWN_' + b.DATA_TYPE 
		 end as c#資料型別 
       ,isnull(b.CHARACTER_MAXIMUM_LENGTH,'') as 長度   
       ,isnull(b.COLUMN_DEFAULT,'')           as 預設值   
       ,b.IS_NULLABLE                         as 是否允許空值   
       ,( SELECT value   
          FROM fn_listextendedproperty (NULL, 'schema', a.TABLE_SCHEMA, 'table', a.TABLE_NAME, 'column', default)   
          WHERE name='MS_Description' and objtype='COLUMN'    
          and objname Collate Chinese_Taiwan_Stroke_CI_AS = b.COLUMN_NAME   
        ) as 欄位描述   
FROM INFORMATION_SCHEMA.TABLES  a   
 LEFT JOIN INFORMATION_SCHEMA.COLUMNS b ON a.TABLE_NAME = b.TABLE_NAME   
WHERE a.TABLE_NAME='TABLENAME'
ORDER BY a.TABLE_NAME , b.ORDINAL_POSITION 
```

##### 產生純文字 - class_model，含屬性成員標籤是否唯獨及長度

```sql
DECLARE @TableName VARCHAR(MAX) = 'tablename' -- Replace 'tablename' with your table name
DECLARE @NameSpace VARCHAR(MAX) = 'namespace' -- Replace 'namespace' with your class namespace
DECLARE @TableSchema VARCHAR(MAX) = 'dbo' -- Replace 'dbo' with your schema name
DECLARE @result varchar(max) = ''

    SET @result = @result + 'using System;' + CHAR(13)
    SET @result = @result + 'using System.ComponentModel.DataAnnotations;' + CHAR(13) + CHAR(13) 

    IF (@TableSchema IS NOT NULL) 
    BEGIN
        SET @result = @result + 'namespace ' + @NameSpace  + CHAR(13) + '{' + CHAR(13) 
    END

    SET @result = @result + 'public class ' + @TableName + CHAR(13) + '{' + CHAR(13) 

    SET @result = @result + '#region Instance Properties' + CHAR(13)  

    SELECT @result = @result + CHAR(13)     
        + ' [Display(Name = "' + ColumnName + '")] ' + CHAR(13) 
        + CASE bRequired WHEN 'NO' 
        THEN 
        CASE WHEN Len(MaxLen) > 0 THEN ' [Required, StringLength(' + MaxLen + ')]' + CHAR(13) ELSE ' [Required] ' + CHAR(13)  END   
        ELSE
        CASE WHEN Len(MaxLen) > 0 THEN ' [StringLength(' + MaxLen + ')]' + CHAR(13) ELSE '' END  
        END
        + ' public ' + ColumnType + ' ' + ColumnName + ' { get; set; } ' + CHAR(13) 
    FROM
    (
        SELECT  c.COLUMN_NAME   AS ColumnName 
            , CASE c.DATA_TYPE   
                WHEN 'bigint' THEN
                    CASE c.IS_NULLABLE
                        WHEN 'YES' THEN 'Int64?' ELSE 'Int64' END
                WHEN 'binary' THEN 'Byte[]'
                WHEN 'bit' THEN 
                    CASE c.IS_NULLABLE
                        WHEN 'YES' THEN 'Boolean?' ELSE 'Boolean' END            
                WHEN 'char' THEN 'String'
                WHEN 'date' THEN
                    CASE c.IS_NULLABLE
                        WHEN 'YES' THEN 'DateTime?' ELSE 'DateTime' END                        
                WHEN 'datetime' THEN
                    CASE c.IS_NULLABLE
                        WHEN 'YES' THEN 'DateTime?' ELSE 'DateTime' END                        
                WHEN 'datetime2' THEN  
                    CASE c.IS_NULLABLE
                        WHEN 'YES' THEN 'DateTime?' ELSE 'DateTime' END                        
                WHEN 'datetimeoffset' THEN 
                    CASE c.IS_NULLABLE
                        WHEN 'YES' THEN 'DateTimeOffset?' ELSE 'DateTimeOffset' END                                    
                WHEN 'decimal' THEN  
                    CASE c.IS_NULLABLE
                        WHEN 'YES' THEN 'Decimal?' ELSE 'Decimal' END                                    
                WHEN 'float' THEN 
                    CASE c.IS_NULLABLE
                        WHEN 'YES' THEN 'Single?' ELSE 'Single' END                                    
                WHEN 'image' THEN 'Byte[]'
                WHEN 'int' THEN  
                    CASE c.IS_NULLABLE
                        WHEN 'YES' THEN 'Int32?' ELSE 'Int32' END
                WHEN 'money' THEN
                    CASE c.IS_NULLABLE
                        WHEN 'YES' THEN 'Decimal?' ELSE 'Decimal' END                                                
                WHEN 'nchar' THEN 'String'
                WHEN 'ntext' THEN 'String'
                WHEN 'numeric' THEN
                    CASE c.IS_NULLABLE
                        WHEN 'YES' THEN 'Decimal?' ELSE 'Decimal' END                                                            
                WHEN 'nvarchar' THEN 'String'
                WHEN 'real' THEN 
                    CASE c.IS_NULLABLE
                        WHEN 'YES' THEN 'Double?' ELSE 'Double' END                                                                        
                WHEN 'smalldatetime' THEN 
                    CASE c.IS_NULLABLE
                        WHEN 'YES' THEN 'DateTime?' ELSE 'DateTime' END                                    
                WHEN 'smallint' THEN 
                    CASE c.IS_NULLABLE
                        WHEN 'YES' THEN 'Int16?' ELSE 'Int16'END            
                WHEN 'smallmoney' THEN  
                    CASE c.IS_NULLABLE
                        WHEN 'YES' THEN 'Decimal?' ELSE 'Decimal' END                                                                        
                WHEN 'text' THEN 'String'
                WHEN 'time' THEN 
                    CASE c.IS_NULLABLE
                        WHEN 'YES' THEN 'TimeSpan?' ELSE 'TimeSpan' END                                                                                    
                WHEN 'timestamp' THEN 
                    CASE c.IS_NULLABLE
                        WHEN 'YES' THEN 'DateTime?' ELSE 'DateTime' END                                    
                WHEN 'tinyint' THEN 
                    CASE c.IS_NULLABLE
                        WHEN 'YES' THEN 'Byte?' ELSE 'Byte' END                                                
                WHEN 'uniqueidentifier' THEN 'Guid'
                WHEN 'varbinary' THEN 'Byte[]'
                WHEN 'varchar' THEN 'String'
                ELSE 'Object'
            END AS ColumnType,
                c.IS_NULLABLE AS bRequired,
                CASE c.DATA_TYPE             
                WHEN 'char' THEN  CONVERT(varchar(10),c.CHARACTER_MAXIMUM_LENGTH)
                WHEN 'nchar' THEN  CONVERT(varchar(10),c.CHARACTER_MAXIMUM_LENGTH)
                WHEN 'nvarchar' THEN  CONVERT(varchar(10),c.CHARACTER_MAXIMUM_LENGTH)
                WHEN 'varchar' THEN  CONVERT(varchar(10),c.CHARACTER_MAXIMUM_LENGTH)
                ELSE ''
            END AS MaxLen,
            c.ORDINAL_POSITION 
    FROM    INFORMATION_SCHEMA.COLUMNS c
    WHERE   c.TABLE_NAME = @TableName and ISNULL(@TableSchema, c.TABLE_SCHEMA) = c.TABLE_SCHEMA  
    ) t
    ORDER BY t.ORDINAL_POSITION

    SET @result = @result + CHAR(13) + '#endregion Instance Properties' + CHAR(13)  

    SET @result = @result  + '}' + CHAR(13)

    IF (@TableSchema IS NOT NULL) 
    BEGIN
        SET @result = @result + CHAR(13) + '}' 
    END

    PRINT @result
```



##### 產生純文字 - 單屬性成員model

```sql
declare @TableName sysname = 'TABLENAME'
declare @Result varchar(max) = 'public class ' + @TableName + '
{'

select @Result = @Result + '
    public ' + ColumnType + NullableSign + ' ' + ColumnName + ' { get; set; }
'
from
(
    select 
        replace(col.name, ' ', '_') ColumnName,
        column_id ColumnId,
        case typ.name 
            when 'bigint' then 'long'
            when 'binary' then 'byte[]'
            when 'bit' then 'bool'
            when 'char' then 'string'
            when 'date' then 'DateTime'
            when 'datetime' then 'DateTime'
            when 'datetime2' then 'DateTime'
            when 'datetimeoffset' then 'DateTimeOffset'
            when 'decimal' then 'decimal'
            when 'float' then 'float'
            when 'image' then 'byte[]'
            when 'int' then 'int'
            when 'money' then 'decimal'
            when 'nchar' then 'string'
            when 'ntext' then 'string'
            when 'numeric' then 'decimal'
            when 'nvarchar' then 'string'
            when 'real' then 'double'
            when 'smalldatetime' then 'DateTime'
            when 'smallint' then 'short'
            when 'smallmoney' then 'decimal'
            when 'text' then 'string'
            when 'time' then 'TimeSpan'
            when 'timestamp' then 'DateTime'
            when 'tinyint' then 'byte'
            when 'uniqueidentifier' then 'Guid'
            when 'varbinary' then 'byte[]'
            when 'varchar' then 'string'
            else 'UNKNOWN_' + typ.name
        end ColumnType,
        case 
            when col.is_nullable = 1 and typ.name in ('bigint', 'bit', 'date', 'datetime', 'datetime2', 'datetimeoffset', 'decimal', 'float', 'int', 'money', 'numeric', 'real', 'smalldatetime', 'smallint', 'smallmoney', 'time', 'tinyint', 'uniqueidentifier') 
            then '?' 
            else '' 
        end NullableSign
    from sys.columns col
        join sys.types typ on
            col.system_type_id = typ.system_type_id AND col.user_type_id = typ.user_type_id
    where object_id = object_id(@TableName)
) t
order by ColumnId

set @Result = @Result  + '
}'

print @Result
```

##### 產生純文字 -  Interface

```sql
declare @script nvarchar(max);
declare @table nvarchar(256);
set @table = 'TABLENAME'

set @script = 'public interface I' + @table + ' {  ' + char(10);
SELECT
    @script = @script + 
        CASE
            WHEN st.Name IN ('int') AND c.is_nullable = 0 THEN '  int'
            WHEN st.name in ('smallint')  AND c.is_nullable = 0 THEN '  short'
            WHEN st.name IN ('bigint') AND c.is_nullable = 0  THEN '  long'
            WHEN st.name IN ('varchar','nvarchar','sysname') THEN '  string'
            WHEN st.Name IN ('datetime') AND c.is_nullable = 0 THEN '  DateTime'
            WHEN st.Name IN ('bit') AND c.is_nullable = 0 THEN '  bool'
            WHEN st.Name IN ('decimal') AND c.is_nullable = 0 THEN '  decimal'
            /* NULLABLE VALUES */
            WHEN st.Name IN ('int') AND c.is_nullable = 1 THEN '  int?'
            WHEN st.name in ('smallint')  AND c.is_nullable = 1 THEN '  short?'
            WHEN st.name IN ('bigint') AND c.is_nullable = 1  THEN '  long?'
            WHEN st.name IN ('varchar','nvarchar','sysname') AND c.is_nullable = 1 THEN '  string?'
            WHEN st.Name IN ('datetime') AND c.is_nullable = 1 THEN '  DateTime?'
            WHEN st.Name IN ('bit') AND c.is_nullable = 1 THEN '  bool?'
            WHEN st.Name IN ('decimal') AND c.is_nullable = 1 THEN '  decimal?'   
            --WHEN st.name IN('sysname') AND c.is_nullable = 1 THEN 'string?'       
            ELSE 'UNKOWN-' + st.name
        END
    + ' ' + c.name + '{get;set;}' + char(10)
FROM sys.tables t
INNER JOIN sys.columns c
ON t.object_id = c.object_id
INNER JOIN sys.types st
ON st.system_type_id = c.system_type_id
WHERE t.name = @table

print @script + '}'
```

##### 產生純文屬性成員帶描述註解-SP

```sql
CREATE PROCEDURE [dbo].[createCode]
(   
   @TableName sysname = '',
   @befor varchar(max)='public class  @TableName  
{',
   @templet varchar(max)=' 
     // @ColumnDesc
     public @ColumnType @ColumnName   { get; set; } 
	 ',
   @after varchar(max)='
}'

)
AS
BEGIN 


declare @result varchar(max)

set @befor =replace(@befor,'@TableName',@TableName)

set @result=@befor

select @result = @result 
+ replace(replace(replace(replace(replace(@templet,'@ColumnType',ColumnType) ,'@ColumnName',ColumnName) ,'@ColumnDesc',ColumnDesc),'@ISPK',ISPK),'@max_length',max_length)

from  
(
    select 
    column_id,
    replace(col.name, ' ', '_') ColumnName,
    typ.name as sqltype,
    typ.max_length,
    is_identity,
    pkk.ISPK, 
        case typ.name 
            when 'bigint' then 'long'
            when 'binary' then 'byte[]'
            when 'bit' then 'bool'
            when 'char' then 'String'
            when 'date' then 'DateTime'
            when 'datetime' then 'DateTime'
            when 'datetime2' then 'DateTime'
            when 'datetimeoffset' then 'DateTimeOffset'
            when 'decimal' then 'decimal'
            when 'float' then 'float'
            when 'image' then 'byte[]'
            when 'int' then 'int'
            when 'money' then 'decimal'
            when 'nchar' then 'char'
            when 'ntext' then 'string'
            when 'numeric' then 'decimal'
            when 'nvarchar' then 'String'
            when 'real' then 'double'
            when 'smalldatetime' then 'DateTime'
            when 'smallint' then 'short'
            when 'smallmoney' then 'decimal'
            when 'text' then 'String'
            when 'time' then 'TimeSpan'
            when 'timestamp' then 'DateTime'
            when 'tinyint' then 'byte'
            when 'uniqueidentifier' then 'Guid'
            when 'varbinary' then 'byte[]'
            when 'varchar' then 'string'
            else 'UNKNOWN_' + typ.name
        END + CASE WHEN col.is_nullable=1 AND typ.name NOT IN ('binary', 'varbinary', 'image', 'text', 'ntext', 'varchar', 'nvarchar', 'char', 'nchar') THEN '?' ELSE '' END ColumnType,
      isnull(colDesc.colDesc,'') AS ColumnDesc 
    from sys.columns col
        join sys.types typ on
            col.system_type_id = typ.system_type_id AND col.user_type_id = typ.user_type_id
            left join
            (
                SELECT c.name  AS 'ColumnName', CASE WHEN dd.pk IS NULL THEN 'false' ELSE 'true' END ISPK           
                FROM        sys.columns c
                    JOIN    sys.tables  t   ON c.object_id = t.object_id    
                    LEFT JOIN (SELECT   K.COLUMN_NAME , C.CONSTRAINT_TYPE as pk  
                        FROM INFORMATION_SCHEMA.KEY_COLUMN_USAGE AS K 
                            LEFT JOIN INFORMATION_SCHEMA.TABLE_CONSTRAINTS AS C
                        ON K.TABLE_NAME = C.TABLE_NAME
                            AND K.CONSTRAINT_NAME = C.CONSTRAINT_NAME
                            AND K.CONSTRAINT_CATALOG = C.CONSTRAINT_CATALOG
                            AND K.CONSTRAINT_SCHEMA = C.CONSTRAINT_SCHEMA            
                        WHERE K.TABLE_NAME = @TableName) as dd
                     ON dd.COLUMN_NAME = c.name
                 WHERE       t.name = @TableName       
            ) pkk  on ColumnName=col.name

    OUTER APPLY (
    SELECT TOP 1 CAST(value AS NVARCHAR(max)) AS colDesc
    FROM
       sys.extended_properties
    WHERE
       major_id = col.object_id
       AND
       minor_id = COLUMNPROPERTY(major_id, col.name, 'ColumnId')
    ) colDesc      
    where object_id = object_id(@TableName)

    ) t

    set @result=@result+@after

    --select @result
    print @result

END
```

##### 產生純文字屬性成員含註解model

```sql
declare @TableName sysname = 'branch';
declare @befor varchar(max)='public class  @TableName  
{';
declare  @templet varchar(max)=' 
     // @ColumnDesc
     public @ColumnType @ColumnName   { get; set; } 
	 ';
declare  @after varchar(max)='
}';

declare @result varchar(max)

set @befor =replace(@befor,'@TableName',@TableName)

set @result=@befor

select @result = @result 
+ replace(replace(replace(replace(replace(@templet,'@ColumnType',ColumnType) ,'@ColumnName',ColumnName) ,'@ColumnDesc',ColumnDesc),'@ISPK',ISPK),'@max_length',max_length)

from  
(
    select 
    column_id,
    replace(col.name, ' ', '_') ColumnName,
    typ.name as sqltype,
    typ.max_length,
    is_identity,
    pkk.ISPK, 
        case typ.name 
            when 'bigint' then 'long'
            when 'binary' then 'byte[]'
            when 'bit' then 'bool'
            when 'char' then 'String'
            when 'date' then 'DateTime'
            when 'datetime' then 'DateTime'
            when 'datetime2' then 'DateTime'
            when 'datetimeoffset' then 'DateTimeOffset'
            when 'decimal' then 'decimal'
            when 'float' then 'float'
            when 'image' then 'byte[]'
            when 'int' then 'int'
            when 'money' then 'decimal'
            when 'nchar' then 'char'
            when 'ntext' then 'string'
            when 'numeric' then 'decimal'
            when 'nvarchar' then 'String'
            when 'real' then 'double'
            when 'smalldatetime' then 'DateTime'
            when 'smallint' then 'short'
            when 'smallmoney' then 'decimal'
            when 'text' then 'String'
            when 'time' then 'TimeSpan'
            when 'timestamp' then 'DateTime'
            when 'tinyint' then 'byte'
            when 'uniqueidentifier' then 'Guid'
            when 'varbinary' then 'byte[]'
            when 'varchar' then 'string'
            else 'UNKNOWN_' + typ.name
        END + CASE WHEN col.is_nullable=1 AND typ.name NOT IN ('binary', 'varbinary', 'image', 'text', 'ntext', 'varchar', 'nvarchar', 'char', 'nchar') THEN '?' ELSE '' END ColumnType,
      isnull(colDesc.colDesc,'') AS ColumnDesc 
    from sys.columns col
        join sys.types typ on
            col.system_type_id = typ.system_type_id AND col.user_type_id = typ.user_type_id
            left join
            (
                SELECT c.name  AS 'ColumnName', CASE WHEN dd.pk IS NULL THEN 'false' ELSE 'true' END ISPK           
                FROM        sys.columns c
                    JOIN    sys.tables  t   ON c.object_id = t.object_id    
                    LEFT JOIN (SELECT   K.COLUMN_NAME , C.CONSTRAINT_TYPE as pk  
                        FROM INFORMATION_SCHEMA.KEY_COLUMN_USAGE AS K 
                            LEFT JOIN INFORMATION_SCHEMA.TABLE_CONSTRAINTS AS C
                        ON K.TABLE_NAME = C.TABLE_NAME
                            AND K.CONSTRAINT_NAME = C.CONSTRAINT_NAME
                            AND K.CONSTRAINT_CATALOG = C.CONSTRAINT_CATALOG
                            AND K.CONSTRAINT_SCHEMA = C.CONSTRAINT_SCHEMA            
                        WHERE K.TABLE_NAME = @TableName) as dd
                     ON dd.COLUMN_NAME = c.name
                 WHERE       t.name = @TableName       
            ) pkk  on ColumnName=col.name

    OUTER APPLY (
    SELECT TOP 1 CAST(value AS NVARCHAR(max)) AS colDesc
    FROM
       sys.extended_properties
    WHERE
       major_id = col.object_id
       AND
       minor_id = COLUMNPROPERTY(major_id, col.name, 'ColumnId')
    ) colDesc      
    where object_id = object_id(@TableName)

    ) t

    set @result=@result+@after

    --select @result
    print @result

```

##### 產生純文字屬性成員帶註解、DisplayName

```sql
declare @TableName sysname = 'TABLENAME';
declare @befor varchar(max)='public class  @TableName  
{';
declare  @templet varchar(max)=' 
     // @ColumnDesc
	 [Display(Name = "@ColumnDesc")]
     public @ColumnType @ColumnName   { get; set; } 
	 ';
declare  @after varchar(max)='
}';

declare @result varchar(max)

set @befor =replace(@befor,'@TableName',@TableName)

set @result=@befor

select @result = @result 
+ replace(replace(replace(replace(replace(@templet,'@ColumnType',ColumnType) ,'@ColumnName',ColumnName) ,'@ColumnDesc',ColumnDesc),'@ISPK',ISPK),'@max_length',max_length)

from  
(
    select 
    column_id,
    replace(col.name, ' ', '_') ColumnName,
    typ.name as sqltype,
    typ.max_length,
    is_identity,
    pkk.ISPK, 
        case typ.name 
            when 'bigint' then 'long'
            when 'binary' then 'byte[]'
            when 'bit' then 'bool'
            when 'char' then 'String'
            when 'date' then 'DateTime'
            when 'datetime' then 'DateTime'
            when 'datetime2' then 'DateTime'
            when 'datetimeoffset' then 'DateTimeOffset'
            when 'decimal' then 'decimal'
            when 'float' then 'float'
            when 'image' then 'byte[]'
            when 'int' then 'int'
            when 'money' then 'decimal'
            when 'nchar' then 'char'
            when 'ntext' then 'string'
            when 'numeric' then 'decimal'
            when 'nvarchar' then 'String'
            when 'real' then 'double'
            when 'smalldatetime' then 'DateTime'
            when 'smallint' then 'short'
            when 'smallmoney' then 'decimal'
            when 'text' then 'String'
            when 'time' then 'TimeSpan'
            when 'timestamp' then 'DateTime'
            when 'tinyint' then 'byte'
            when 'uniqueidentifier' then 'Guid'
            when 'varbinary' then 'byte[]'
            when 'varchar' then 'string'
            else 'UNKNOWN_' + typ.name
        END + CASE WHEN col.is_nullable=1 AND typ.name NOT IN ('binary', 'varbinary', 'image', 'text', 'ntext', 'varchar', 'nvarchar', 'char', 'nchar') THEN '?' ELSE '' END ColumnType,
      isnull(colDesc.colDesc,'') AS ColumnDesc 
    from sys.columns col
        join sys.types typ on
            col.system_type_id = typ.system_type_id AND col.user_type_id = typ.user_type_id
            left join
            (
                SELECT c.name  AS 'ColumnName', CASE WHEN dd.pk IS NULL THEN 'false' ELSE 'true' END ISPK           
                FROM        sys.columns c
                    JOIN    sys.tables  t   ON c.object_id = t.object_id    
                    LEFT JOIN (SELECT   K.COLUMN_NAME , C.CONSTRAINT_TYPE as pk  
                        FROM INFORMATION_SCHEMA.KEY_COLUMN_USAGE AS K 
                            LEFT JOIN INFORMATION_SCHEMA.TABLE_CONSTRAINTS AS C
                        ON K.TABLE_NAME = C.TABLE_NAME
                            AND K.CONSTRAINT_NAME = C.CONSTRAINT_NAME
                            AND K.CONSTRAINT_CATALOG = C.CONSTRAINT_CATALOG
                            AND K.CONSTRAINT_SCHEMA = C.CONSTRAINT_SCHEMA            
                        WHERE K.TABLE_NAME = @TableName) as dd
                     ON dd.COLUMN_NAME = c.name
                 WHERE       t.name = @TableName       
            ) pkk  on ColumnName=col.name

    OUTER APPLY (
    SELECT TOP 1 CAST(value AS NVARCHAR(max)) AS colDesc
    FROM
       sys.extended_properties
    WHERE
       major_id = col.object_id
       AND
       minor_id = COLUMNPROPERTY(major_id, col.name, 'ColumnId')
    ) colDesc      
    where object_id = object_id(@TableName)

    ) t

    set @result=@result+@after

    --select @result
    print @result

```

