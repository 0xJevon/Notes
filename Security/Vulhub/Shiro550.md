返回包携带`rememberMe=deleteMe`确认使用Shiro
![[Shiro550-20260910.png]]
通过ysoserial生成恶意序列化包，使用CB链
`java -jar ysoserial.jar CommonsBeanutils1 "touch /tmp/Jevon" > shiro.ser`
Shiro框架解密Cookie通过base64解码->AES解密->反序列化，当AES密钥为默认密钥或可爆破时即可实现反序列化利用（序列化->AES加密->base64编码），编写代码，借助Shiro框架实现AES加密
![[Shiro550-20260910-1.png]]
将编码后的内容发送，Shiro框架硬编码的Cookie名称为rememberMe
![[Shiro550-20260910-2.png]]
实现Shiro反序列化攻击
![[Shiro550-20260910-3.png]]