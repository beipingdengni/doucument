# sip 协议

js

https://github.com/versatica/JsSIP

https://juejin.cn/post/7336093196354551827



java博客：

JAIN SIP API详解与GB28181服务器实现：Java编写SIP协议  https://blog.csdn.net/qq_41519442/article/details/140757660

https://blog.csdn.net/qq_27890899/article/details/130968649

java使用sip： https://blog.51cto.com/u_16175432/6908341

https://blog.51cto.com/u_16175454/11506902

GB28181:基于JAVA的Catalog目录获取: https://www.jianshu.com/p/83b568929e56

```xml
<!-- GB28181 规范了实现 SIP 监控域与非SIP 监控域互联 -->
<dependency>
  <groupId>javax.sip</groupId>
  <artifactId>jain-sip-api</artifactId>
  <version>1.3.0-91</version>
</dependency>
<!-- sip协议栈 -->
<dependency>
  <groupId>javax.sip</groupId>
  <artifactId>jain-sip-ri</artifactId>
  <version>1.3.0-91</version>
</dependency>
```

```java
import javax.sip.*;
import javax.sip.address.*;
import javax.sip.header.*;
import javax.sip.message.*;

public class SipClient {
    private static final String SIP_SERVER_IP = "127.0.0.1";
    private static final int SIP_SERVER_PORT = 5060;
    private static final String FROM_NAME = "JavaClient";
    private static final String FROM_ADDRESS = "sip:" + FROM_NAME + "@127.0.0.1";
    private static final String TO_NAME = "JavaServer";
    private static final String TO_ADDRESS = "sip:" + TO_NAME + "@127.0.0.1";

    public static void main(String[] args) {
        try {
            SipFactory sipFactory = SipFactory.getInstance();
            SipStack sipStack = sipFactory.createSipStack(null);

            ListeningPoint listeningPoint = sipStack.createListeningPoint(SIP_SERVER_IP, SIP_SERVER_PORT, "UDP");

            AddressFactory addressFactory = sipFactory.createAddressFactory();
            SipURI fromUri = addressFactory.createSipURI(FROM_NAME, FROM_ADDRESS);
            SipURI toUri = addressFactory.createSipURI(TO_NAME, TO_ADDRESS);

            Address fromAddress = addressFactory.createAddress(fromUri);
            Address toAddress = addressFactory.createAddress(toUri);

            ApplicationSession applicationSession = sipStack.createApplicationSession(listeningPoint);

            ClientTransaction inviteTid = applicationSession.createClientTransaction(
                    createInvite(fromAddress, toAddress, applicationSession)
            );

            System.out.println("Invite sent");

            sipStack.start();
        } catch (Exception e) {
            e.printStackTrace();
        }
    }

    private static Message createInvite(Address fromAddress, Address toAddress, ApplicationSession applicationSession) throws Exception {
        Request inviteRequest = applicationSession.createRequest(fromAddress, toAddress, "INVITE");
        inviteRequest.setHeader("Contact", FROM_ADDRESS);
        inviteRequest.setHeader("Content-Type", "application/sdp");
        inviteRequest.setHeader("Expires", "60");

        byte[] contentBytes = "v=0\r\n" +
                "o=- 0 0 IN IP4 127.0.0.1\r\n" +
                "s=-\r\n" +
                "c=IN IP4 127.0.0.1\r\n" +
                "t=0 0\r\n" +
                "m=audio 49170 RTP/AVP 0 8".getBytes();
        inviteRequest.setContent(contentBytes, "application/sdp");

        return inviteRequest;
    }
}
```

流媒体

https://github.com/ZLMediaKit/ZLMediaKit

官网：https://docs.zlmediakit.com/

- 基于C++11开发，避免使用裸指针，代码稳定可靠，性能优越。
- 支持多种协议(RTSP/RTMP/HLS/HTTP-FLV/WebSocket-FLV/GB28181/HTTP-TS/WebSocket-TS/HTTP-fMP4/WebSocket-fMP4/MP4/WebRTC),支持协议互转。

```
#此镜像为github action 持续集成自动编译推送，跟代码(master分支)保持最新状态
docker run -id -p 1935:1935 -p 8080:80 -p 8443:443 -p 8554:554 -p 10000:10000 -p 10000:10000/udp -p 8000:8000/udp -p 9000:9000/udp zlmediakit/zlmediakit:master
```