---
title: Oracle 实用 SQL
date: 2018-07-22 22:22:22
categories: 数据库
tags:
- Oracle
summary: 记录一些不经常用但是需要的时候很有用的查询语句，不定期更新
---

记录一些不经常用但是需要的时候很有用的查询语句，不定期更新

<!--more-->

## 数据库编码
```sql
SELECT USERENV('LANGUAGE') FROM DUAL;
SELECT * FROM NLS_DATABASE_PARAMETERS T WHERE T.PARAMETER IN ('NLS_LANGUAGE', 'NLS_TERRITORY', 'NLS_CHARACTERSET');
SELECT * FROM NLS_DATABASE_PARAMETERS T WHERE T.PARAMETER LIKE '%LANGUAGE%' OR T.PARAMETER LIKE '%TERRI%' OR T.PARAMETER LIKE '%CHAR%' UNION ALL (SELECT 'USERENV', USERENV('LANGUAGE') FROM DUAL);
SELECT * FROM NLS_DATABASE_PARAMETERS T WHERE T.PARAMETER = 'NLS_CHARACTERSET' OR PARAMETER='NLS_LANGUAGE' OR PARAMETER = 'NLS_NCHAR_CHARACTERSET';
```

## 表相关

### 查询表名

```sql
SELECT TABLE_NAME,TABLESPACE_NAME,TEMPORARY FROM USER_TABLES WHERE TABLE_NAME LIKE '%USER%'
```

### 查询列名

```sql
SELECT COLUMN_NAME,DATA_TYPE ,DATA_LENGTH,DATA_PRECISION,DATA_SCALE FROM USER_TAB_COLUMNS [WHERE TABLE_NAME=表名];
```

例如查询列名包含 `DEPT` 的表：

```sql
SELECT T.TABLE_NAME, T.COLUMN_NAME FROM USER_TAB_COLUMNS T WHERE T.COLUMN_NAME LIKE '%DEPT%'
```

### 查询指定值所在表和列

```sql
DECLARE
  CURSOR CUR_QUERY IS
    SELECT TABLE_NAME, COLUMN_NAME, DATA_TYPE FROM USER_TAB_COLUMNS;
  SQL_HARD   VARCHAR2(2000);
  FIND_VALUE VARCHAR2(100);
  DATE_FMT   VARCHAR2(20);
  VV         NUMBER;
BEGIN
  DBMS_OUTPUT.ENABLE(
      BUFFER_SIZE   => NULL
  );
  FIND_VALUE := '20180716';
  DATE_FMT := 'yyyymmdd hh24:mi:ss';
  SQL_HARD := '';
  FOR REC1 IN CUR_QUERY LOOP
    IF REC1.DATA_TYPE = 'VARCHAR2'
    THEN
      SQL_HARD := 'SELECT COUNT(*) FROM  ' || REC1.TABLE_NAME
                  || ' WHERE '
                  || REC1.COLUMN_NAME || ' LIKE ''%' || FIND_VALUE || '%''';
    ELSIF REC1.DATA_TYPE = 'NUMBER'
      THEN
        SQL_HARD := 'SELECT COUNT(*) FROM  ' || REC1.TABLE_NAME
                    || ' WHERE TO_CHAR('
                    || REC1.COLUMN_NAME || ') LIKE ''%' || FIND_VALUE || '%''';

    ELSIF REC1.DATA_TYPE = 'TIMESTAMP(6)' OR REC1.DATA_TYPE = 'DATE'
      THEN
        SQL_HARD := 'SELECT COUNT(*) FROM  ' || REC1.TABLE_NAME
                    || ' WHERE TO_CHAR('
                    || REC1.COLUMN_NAME || ',''' || DATE_FMT || ''') LIKE ''%' || FIND_VALUE || '%''';
    --     ELSE
    --       DBMS_OUTPUT.PUT_LINE(REC1.DATA_TYPE);
    END IF;
    EXECUTE IMMEDIATE SQL_HARD INTO VV;
    IF VV > 0
    THEN
      DBMS_OUTPUT.PUT_LINE(REC1.DATA_TYPE || ' 类型字段值 ''' || FIND_VALUE || ''' 在 '
                           || REC1.TABLE_NAME || ' 表中的 ' || REC1.COLUMN_NAME || ' 字段');
    END IF;
  END LOOP;
END;

```

>`Oracle SQL Developer` 中 View → Dbms Output 选项显示输出内容
>`IntelliJ IDEA` 中在 `Database Console` 窗口中左上角点击 ![](http://wx4.sinaimg.cn/large/a6e9cb00ly1fuibejxbvaj20160163yd.jpg) 图标显示输出内容

## 函数用法

### 字符串长度——LENGTH

```sql
SELECT LENGTH('2018-07-0415:22:00') FROM DUAL;
```

### 截取字符串——SUBSTR

```sql
SELECT SUBSTR('2018-07-0415:22:00', 0, 10) FROM DUAL;
```

### 合并字符串——||

```sql
SELECT SUBSTR('2018-07-0415:22:00', 0, 10)||' '||SUBSTR('2018-07-0415:22:00', 11, 8) FROM DUAL;
```

### 字符串数字转换

```sql
SELECT TO_CHAR(2) || '1' FROM DUAL;
SELECT TO_NUMBER('2') + 1 FROM DUAL;
```