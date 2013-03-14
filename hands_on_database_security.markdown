I wrote these notes for a student-led presentation I gave to
[UAF's](http://www.uaf.edu/)
[Pacific Rim CCDC](http://ciac.ischool.washington.edu/) team.

This information should be doubly useful should UAF host its own capture the flag event,
but hopefully more on that later.

# overview

These notes list some important security issues that people working with databases
should be aware of.

Hands-on exercises and examples are provided because playing with real code is much
more fun and educational than merely discussing these vulnerabilities.

# What do databases do?

![malicious user](/images/malicious-user.png)

Relational databases are everywhere. They keep track of your health records,
your online sock puppet purchasing history, your favorite government agency's
clandestine domestic spying program, and your state or province's electoral
process even if you don't use an electronic voting machine.
Despite or perhaps because of all these uses, databases are often exploited by
malicious parties in an increasingly lucrative black market for personal
information, trade secrets, and government intelligence.

Relational databases pretty much all use some form or another of
[SQL](http://en.wikipedia.org/wiki/SQL).

You can [learn more about SQL](http://www.w3schools.com/SQl/default.asp)
and play with an [SQL Sandbox](/projects/database-security/sandbox/)
if you haven't used it before or don't remember it very well.

# database connections

The default configurations of many databases will pass queries unencrypted over
the network. To demonstrate with a default installation of MySQL 5.1 on Debian
GNU/Linux,

```
# tshark -Tfields -e mysql.user -e mysql.passwd -i lo -R mysql.user
```

```
$ mysql -h 127.0.0.1 -u root -p
```

after entering the password into the mysql client produces the output

```
Running as user "root" and group "root". This could be dangerous.
Capturing on lo
root    \x7fr\xe9ArY?7w4\xa5\xb2\x8e>\xa3\xff\x03j\x99\xc3
```

which thankfully is hashed, but the username is sent unencrypted over the
network. The queries themselves are not encrypted by default, as this next
example demonstrates.

For this session,

```
$ mysql -h 127.0.0.1 -u root -p
Enter password: 
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 52
Server version: 5.1.37-2 (Debian)
 
Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.
 
mysql> use ocelot;
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A
 
Database changed
mysql> insert into friends (id, name, description) values (1337, 'turtle', 'c/,,\\');
Query OK, 1 row affected (0.12 sec)
 
mysql> select * from friends;
+------+--------+-------------+
| id   | name   | description |
+------+--------+-------------+
|    0 | cow    | it says moo | 
| 1337 | turtle | c/,,\       | 
+------+--------+-------------+
2 rows in set (0.02 sec)

mysql>
```

Wireshark is able to capture

```
# tshark -Tfields -e mysql.query -i lo -R 'mysql.query'
# tshark -Tfields -e mysql.query -i lo -R 'mysql.query'
Running as user "root" and group "root". This could be dangerous.
Capturing on lo
select @@version_comment limit 1
SELECT DATABASE()
show databases
show tables
insert into friends (id, name, description) values (1337, 'turtle', 'c/,,\\\\')
select * from friends
```

Admittedly, for an attacker to capture this information, they would need to
control a system between the database and the client.
However, if a database is storing confidential information such as credit card
numbers (which should only be stored by the credit card companies but I digress),
employment information,
or health records,
basing security of the databases upon the security of the network sees unwise
when it's not too much work to configure the database use SSL in most cases.

Here's how to
[configure MySQL with SSL](http://dev.mysql.com/doc/refman/5.1/en/secure-using-ssl.html)
and since version 8, PostgreSQL enables SSL by default.
You can double-check that your postgres database is using ssl by looking for
`"ssl = true"` in `/etc/postgresql/$PG_VERSION/main/postgresql.conf`
(or a similar place depending on how postgres was installed).

If your database and the applications that use it are on the same system,
consider disabling networking altogether and using a
[UNIX socket](http://en.wikipedia.org/wiki/Unix_domain_socket) instead.

For more information about wireshark filters, consult the
[manual](http://www.wireshark.org/docs/man-pages/wireshark-filter.html).

# unsafe functions

Some database servers, notably Microsoft SQL Server when poorly configured,
expose interfaces to unsafe system calls from SQL in order to write more
powerful stored procedures.

Functions such as
[xp_cmdshell](http://msdn.microsoft.com/en-us/library/ms175046.aspx)
are particularly
[infamous](http://www.databasejournal.com/features/mssql/article.php/3372131/Using-xpcmdshell.htm).

MySQL also allows
[select statements](http://dev.mysql.com/doc/refman/5.0/en/select.html)
to load data into and out of files on disk.

Other databases come with their own security caveats that programmers should be aware of.

# roles

For databases that support it, setting up roles with the lowest permissions
necessary to get the job done is important so that a compromise of one system
limits the damage or disclosure of the intrusion.

To list the roles and users on a system,

## in a PostgreSQL shell:

* `\du`
* `select * from pg_roles;`
* `select * from pg_user;`

## in a MySQL shell:

* `show grants;`
* `select * from mysql.user;`
* `select * from mysql.tables_priv;`

To grant or revoke roles, consult the

[MySQL](http://dev.mysql.com/doc/refman/5.1/en/grant.html) and
[PostgreSQL](http://www.postgresql.org/docs/8.1/static/sql-grant.html)
documentation.

There is some
[good information at this site](http://database-programmer.blogspot.com/2008/05/introducing-database-security.html)
too.
 
# sql injection

![malicious user attacks but is denied in a cliché full-screen error message](/images/malicious-attack.png)

Suppose a website uses usernames and passwords to do authentication. A hapless
web developer might write a PHP script that queries a database like this:

``` php
$username = $_POST['username'];
$password = $_POST['password'];
$result = sqlite_query($db, "
    select * from users where
        username = '$username' and password = '$password'
");

if ($row = sqlite_fetch_array($result)) {
    echo "Welcome " . $row[0] . "!";
}
else {
    echo "Authentication failure";
}
```

For a username of `"caligula"` and a password of `"tyranny"`, the query would be

``` sql
select * from users where username = 'caligula' and password = 'tyranny'
```

But if a malicious user enters a username of `"caligula' --"` and doesn't enter
a password, the query becomes

``` sql
select * from users where username = 'caligula' --' and password = ''
```

which for databases where `"--"` is configured as a comment will successfully
authenticate as the user `"caligula"` without the malicious user needing to know
the password.

Worse still, the password could be retreived by submitting a username of

```
' union select password from users where username='caligula' --
```

would perform an SQL union on the database, allowing an attacker to see
any tables and columns that the configured database user can see.

``` sql
select * from users where
    username = '' union select password from users where username='caligula' --'
    and password = ''
```

The
[SQL union operator](http://www.w3schools.com/sql/sql_union.asp)
combines two or more select statements with the same
number of columns into one row-set.

If the attacker doesn't know how many columns the query had originally, they
can just use database constants such as numbers instead of column names in their
union until the query succeeds.
By using different numbers, they can even see where in the output the columns
that they want to see will show up at.

The attacker might not know the names of the columns or table names, but this
information can be obtained through the database schema which is stored in the
database itself.

In both
[MySQL](http://dev.mysql.com/doc/refman/5.0/en/information-schema.html)
and
[PostgreSQL](http://www.postgresql.org/docs/8.4/static/information-schema.html),
this information is contained in the
information_schema database.
Of particular interest are the "tables" and "columns" tables.
In SQLite this information is available in the
[sqlite_master](http://www.sqlite.org/faq.html#q7) table.

Re-visiting the previous example, setting the username to

```
' union select sql from sqlite_master --
```
gives back the result

```
Welcome CREATE TABLE users (username text, password text)!
```

# protecting against sql injection

There are several approaches for protecting against SQL injection.
Escaping the user input manually is one option. From the authentication
example from earlier in the document:

``` php
$username = sqlite_escape_string($_POST['username']);
$password = sqlite_escape_string($_POST['password']);
```

A cleaner way to protect against SQL injection is by using placeholders,
which bind parameters to a query and handle all the string escaping.
The authentication example can be made to use placeholders:

``` php
$sth = $db-&gt;prepare("select username from users
    where username = '?' and password = '?'
");

$sth-&gt;execute($_POST["username"], $_POST["password"])
  or die("Error: " . sqlite_error_string(sqlite_last_error($db)));

if ($row = $sth-&gt;fetch()) {
    echo "Welcome " . $row[0] . "!";
}
else {
    echo "Authentication failure";
}
```

For more information about placeholders and prepared statements, see

* [placeholders in php](http://php.net/manual/en/pdo.prepared-statements.php)
* [placeholders in perl](http://search.cpan.org/~timb/DBI/DBI.pm#Placeholders_and_Bind_Values)
* [ruby dbi synopsis](http://ruby-dbi.rubyforge.org/rdoc/index.html)

Object-relational mappers provide a more elegant way to protect against SQL injection,
since the actual queries are abstracted behind objects instead of written directly.

* [orm libraries in php](http://stackoverflow.com/questions/108699/good-php-orm-library)
* [comparison of perl orm modules](http://www.perlmonks.org/?node_id=700283)
* [orm solutions for python](http://stackoverflow.com/questions/53428/what-are-some-good-python-orm-solutions)
* ruby has [ActiveRecord](http://ar.rubyonrails.org/),
[DataMapper](http://datamapper.org/), and
[Sequel](http://sequel.rubyforge.org/), to name a few

# exercises

Try to exploit these vulnerable example scripts.
There is a single md5 sum hiding somewhere in each SQLite database.
You shouldn't need to look at the source or the database for any of these to exploit them,
but the source especially comes in handy if you are stuck.
There are also writeups provided if you like spoilers.

* [auth](/projects/database-security/auth/)
* [index.php](/projects/database-security/auth/index.phps">)
* [db.sqlite](/projects/database-security/auth/db.sqlite)
* [writeup.txt](/projects/database-security/auth/writeup.txt)

This is the example from the previous section.
There is a hash hidden somewhere too.

* [quotes](/projects/database-security/quotes/)
* [index.pl](/projects/database-security/quotes/index.pl)
* [db.sqlite3](/projects/database-security/quotes/db.sqlite3)
* [writeup.txt](/projects/database-security/quotes/writeup.txt)

This script shows quotes. The hash is in a quote that isn't listed.

* [articles](/projects/database-security/articles/)
* [index.pl](/projects/database-security/articles/index.pl)
* [db.sqlite3](/projects/database-security/articles/db.sqlite3)
* [writeup.txt](/projects/database-security/articles/writeup.txt)

This script shows articles. The hash is in a secret table with a secret column
name.
