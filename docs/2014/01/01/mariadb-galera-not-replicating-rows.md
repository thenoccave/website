# MariaDB Galera Not Replicating Rows
After recently getting a MariaDB Galera cluster up and running (https://www.thenoccave.com/2013/12/30/mariadb-galera-cluster-ha-proxy-keepalived-centos-6/) we decided to move some none essential databases across to it as a test.

After moving a couple of Drupal sites without issue we decided to move our Piwik Analytics database and that is when we started hitting issues. We were just using mysqldump to dump to a file then importing into into MariaDB.

With the Piwik database the web interface starting throwing database errors. Using the mysql client we could connect no problem and list all the tables. After some more digging we found that although the database tables and all their structures were on both servers the rows only appeared on one.

For instance doing a query against the sites table showed all the rows one one server and no rows on the other:
```
MariaDB [analytics]> select * from piwik_site;
Empty set (0.00 sec)
 
MariaDB [analytics]> select * from piwik_site;
6 rows in set (0.00 sec)
```
After pulling my hair out all day I finally found the issue. One of the known limitations of MariaDB Galera ([https://mariadb.com/kb/en/mariadb-galera-cluster-known-limitations/](https://mariadb.com/kb/en/mariadb-galera-cluster-known-limitations/)) is that it can only replicate InnoDB tables

Having a look at the Piwik site table

```
MariaDB [analytics]> SHOW TABLE STATUS WHERE Name = 'piwik_site';
+------------+--------+
| Name       | Engine |
+------------+--------+
| piwik_site | MyISAM |
+------------+--------+
```

Finally a clue. Using the instructions at [http://rocketmodule.com/blog/convert-your-mysql-database-myisam-innodb-and-get-ready-drupal-7-same-time](http://rocketmodule.com/blog/convert-your-mysql-database-myisam-innodb-and-get-ready-drupal-7-same-time) I converted the database tables to InnoDB on the source server, redumped it using mysqldump and imported it back into MariaDB and all was working as it should