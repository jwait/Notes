# How to Install MySQL on CentOS 7

MySQL is a popular database management system used for web and server applications. However, MySQL is no longer in CentOS’s repositories and MariaDB has become the default database system offered. MariaDB is considered a [drop-in replacement ](https://mariadb.com/kb/en/mariadb/mariadb-vs-mysql-compatibility/)for MySQL and would be sufficient if you just need a database system in general. See our [MariaDB in CentOS 7](https://www.linode.com/docs/databases/mariadb/how-to-install-mariadb-on-centos-7) guide for installation instructions.

If you nonetheless prefer MySQL, this guide will introduce how to install, configure and manage it on a Linode running CentOS 7.

> This guide is written for a non-root user. Commands that require elevated privileges are prefixed with `sudo`. If you’re not familiar with the `sudo` command, you can check our [Users and Groups](https://www.linode.com/docs/tools-reference/linux-users-and-groups) guide.

## Before You Begin