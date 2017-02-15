# How to install Oracle JDK 8 on CentOS

![java8-centos-example](http://www.mkyong.com/wp-content/uploads/2016/07/java8-centos-example.png)

In this tutorial, we will show you how to install Oracle JDK 8 On CentOS.

Environment :

1. CentOS 6.8
2. Oracle JDK 8u102

**Note**
This guide should work on Fedora and RedHat.

## 1. Get Oracle JDK 8

1.1 Visit [Oracle JDK download page](http://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html), look for `RPM` version.

1.2 Copy the download link of `jdk-8u102-linux-x64.rpm` and `wget` it.

```
$ pwd
/home/mkyong

$ wget --header "Cookie: oraclelicense=accept-securebackup-cookie" http://download.oracle.com/otn-pub/java/jdk/8u102-b14/jdk-8u102-linux-x64.rpm
```

## 2. Install Oracle JDK 8

2.1 Install with `yum localinstall`.

```
$ sudo yum localinstall jdk-8u102-linux-x64.rpm

//...
//...
//...
Installed:
  jdk1.8.0_102.x86_64 2000:1.8.0_102-fcs

Complete!
```

2.2 Now the JDK should be installed at `/usr/java/jdk1.8.0_102`

```
$ cd /usr/java
$ ls -lsah
total 12K
4.0K drwxr-xr-x   3 root root 4.0K Jul 21 09:58 ./
4.0K drwxr-xr-x. 15 root root 4.0K Jun 22 22:00 ../
   0 lrwxrwxrwx   1 root root   16 Jul 21 09:58 default -> /usr/java/latest/
4.0K drwxr-xr-x   9 root root 4.0K Jul 21 09:58 jdk1.8.0_102/
   0 lrwxrwxrwx   1 root root   22 Jul 21 09:58 latest -> /usr/java/jdk1.8.0_102/
```

2.3 Verification

```
$ java -version
java version "1.8.0_102"
Java(TM) SE Runtime Environment (build 1.8.0_102-b14)
Java HotSpot(TM) 64-Bit Server VM (build 25.102-b14, mixed mode)
```

2.4 Delete the RPM file

```
$ rm ~/jdk-8u102-linux-x64.rpm
```

Done. Oracle JDK 8 is installed on CentOS successfully.

## 3. JAVA_HOME Environment Variables

This is good practice to set the `JAVA_HOME` environment variable.

3.1 Edit the `.bash_profile`, and append the `export JAVA_HOME` at the end of the file, for example :

.bash_profile

```
# Get the aliases and functions
if [ -f ~/.bashrc ]; then
	. ~/.bashrc
fi

# User specific environment and startup programs

export JAVA_HOME=/usr/java/jdk1.8.0_102/
export JRE_HOME=/usr/java/jdk1.8.0_102/jre

PATH=$PATH:$HOME/bin:$JAVA_HOME/bin

export PATH
```

3.2 Test the `$JAVA_HOME` and `$PATH`

```
$ source .bash_profile

$ echo $JRE_HOME
/usr/java/jdk1.8.0_102/jre

$ echo $JAVA_HOME
/usr/java/jdk1.8.0_102/

$ echo $PATH
/...:/usr/local/bin:/usr/X11R6/bin:/home/mkyong/bin:/usr/java/jdk1.8.0_102//bin
```

## 4. Multiple JDK installed

If the CentOS has multiple JDK installed, you can use the `alternatives` command to set the default `java`

```
$ sudo alternatives --config java
[sudo] password for mkyong:

There are 2 programs which provide 'java'.

  Selection    Command
-----------------------------------------------
   1           /usr/lib/jvm/jre-1.6.0-openjdk.x86_64/bin/java
*+ 2           /usr/java/jdk1.8.0_102/jre/bin/java

Enter to keep the current selection[+], or type selection number:
```