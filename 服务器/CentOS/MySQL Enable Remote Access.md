# MySQL Enable Remote Access

If you try to connect to your mysql server from remote machine, and run into error like below, this article is for you.

ERROR 1130 (HY000): Host ‘1.2.3.4’ is not allowed to connect to this MySQL server

### Change mysql config

Start with editing mysql config file

```
vim /etc/mysql/my.cnf
```

Comment out following lines.

```
#bind-address           = 127.0.0.1
#skip-networking
```

If you do not find skip-networking line, add it and comment out it.

Restart mysql server.

```
service mysql restart
```

### Change GRANT privilege

You may be surprised to see even after above change you are not getting remote access or getting access but not able to all databases.

By default, mysql username and password you are using is allowed to access mysql-server locally. So need to update privilege.

Run a command like below to access from all machines.

```
mysql> GRANT ALL PRIVILEGES ON *.* TO 'USERNAME'@'%' IDENTIFIED BY 'PASSWORD' WITH GRANT OPTION;
```

Run a command like below to give access from specific IP.

```
mysql> GRANT ALL PRIVILEGES ON *.* TO 'USERNAME'@'1.2.3.4' IDENTIFIED BY 'PASSWORD' WITH GRANT OPTION;
```

You can replace `1.2.3.4` with your IP. You can run above command many times to GRANT access from multiple IPs.

You can also specify a separate `USERNAME` & `PASSWORD` for remote access.

You can check final outcome by:

```
SELECT * from information_schema.user_privileges where grantee like "'USERNAME'%";
```

Finally, you may also need to run:

```
mysql> FLUSH PRIVILEGES;
```

### Test Connection

From terminal/command-line:

```
mysql -h HOST -u USERNAME -pPASSWORD
```

If you get a mysql shell, don’t forget to run `show databases;` to check if you have right privileges from remote machines.

### Bonus-Tip: Revoke Access

If you accidentally grant access to a user, then better have revoking option handy.

Following will revoke all options for USERNAME from all machines:

```
mysql> REVOKE ALL PRIVILEGES, GRANT OPTION FROM 'USERNAME'@'%';
```

Following will revoke all options for USERNAME from particular IP:

```
mysql> REVOKE ALL PRIVILEGES, GRANT OPTION FROM 'USERNAME'@'1.2.3.4';
```

Its better to check `information_schema.user_privileges` table after running REVOKE command.

If you see USAGE privilege after running REVOKE command, its fine. It is as good as no privilege at all. I am not sure if it can be revoked.

```
+-------------------------+---------------+-------------------------+--------------+
| GRANTEE                 | TABLE_CATALOG | PRIVILEGE_TYPE          | IS_GRANTABLE |
+-------------------------+---------------+-------------------------+--------------+
| 'USERNAME'@'%'          | def           | USAGE                   | NO           |
+-------------------------+---------------+-------------------------+--------------+
```

If we missed anything, feel free to let us know using comment form below.