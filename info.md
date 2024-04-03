# SQL Injection Overview – Getting Started 

Useful External Sources

•	SQL injection concepts and code – I recommend this for more in depth explanation https://book.hacktricks.xyz/pentesting-web/sql-injection

•	SQL Injection explanation with diagrams and tutorial: https://portswigger.net/web-security/sql-injection

Payloads: https://github.com/payloadbox/sql-injection-payload-list

SQL Syntax – Structuring an attack

1.	Take the following query:
SELECT username, passwords FROM users WHERE userID = userIDVariable

# # What is happening?
o	select any data from username and password columns where userID matches the userIDVariable
•	"username" and "passwords" are the columns the SQL statement is operating on
•	"FROM users" specifies from the users table
•	WHERE "userid" is specifying a match on the userID column where a value matches that userID

# # How can we attack this – Union Based Attack?

SELECT username, passwords FROM users WHERE userID = userIDVariable

•	It is a SELECT query

•	There are two columns: username, passwords

•	The table is users

We will need to supply a payload that contains two columns. How can we do this?

# # Beginning our attack
For this example we already know that there are two columns in the query. In real world scenarios we will have to enumerate this.

So that our payload is compliant with the syntax of the query we will need to provide two columns. We can do this following the structure of the query itself in multiple ways. 

Two common examples are

a.	‘ SELECT NULL, NULL FROM users --	

b.	‘ SELECT 1, 2 FROM users –

# # # Note the following:
‘ SELECT NULL, NULL, NULL FROM users –

This would not be compliant with the UNION SELECT statement as it contains three columns. If the default MySQL error response is returned to us on the web application we would commonly see the error:

The used SELECT statements have a different number of columns

Therefore in our example we need to prefix any payloads with one of our previous two column examples:

‘ SELECT NULL, NULL 

# # Enumerating the MySQL Server – About Information Schema

We do not know any additional column names, tables or databases. So how can we enumerate this?

The information schema!

Reading regarding information schema: https://dev.mysql.com/doc/refman/8.0/en/information-schema-introduction.html

The information schema provides metadata regarding what the MySQL server maintains.

•	It is represented in an SQL statement as information_schema

Querying the Information Schema

To query the information schema for tables we can use the following statement:

•	table_name from information_schema.tables – 

Combining this with our prefix provides the following:

•	SELECT NULL, table_name from information_schema.tables – 

Notice that we have omitted one of the NULLs. This is because table_name is a column in the tables table of the information schema
 
With the environment setup on your system you can login and view the database information schema for a better overview if you are unsure. Or just ask me.

# # # Finding the columns
We will assume another table called named customers is returned 

How can we find the column values of this table?

We can do this with the following syntax:

•	table_name, column_name from information_schema.columns

This will loop through each table and return any relevant columns for that table.
 
The above figure shows there is a customers table and a customerNumber column

# # # Dealing with column limits

What if we have a single column in our original query? 

Well if we try table_name, column_name the server will return an error about the invalid number of columns supplied. 

How can we bypass this? Concatenation!

Take the following query:

•	‘ SELECT CONCAT(table_name, column_name) from information_schema.columns – c

The table_name, column_name is concatenated as a single column but returns both values. 
 
Notice how in the above figure the string is merged. We can separate this with a whitespace within our CONCAT. 

I prefer to use ‘identifiers’ in my CONCAT statements that contain a random symbol so that I can parse them via regex queries. 
Makes it easier to identify within HTML responses. Could be a better way to do it but that’s how I have always worked

Example: CONCAT(“||”,table_name,“||”,column_name,“||”) 

# # # Summary
That is a quick overview on how to do a basic UNION SELECT and apply it to the site. I recommend following the guides for a more in depth explanation with code.
Steps:

1.	Find your delimiter and correct comment.
   
a.	This could be ‘ or “ for the delimiter

b.	It could be:	--	 /*	# 

c.	Enumerate the columns: 	

i.	Keep adding NULL or numbers to your column values until the system stops returning errors

d.	Enumerate the tables by querying the information schema

e.	Enumerate the columns by querying the information schema – This can be done overall using table_name, column_name  or by specifying the specific table:

i.	e.g. FROM information_schema.columns WHERE table_name = “customers”

f.	See if you can find the flag in the returned response


 







