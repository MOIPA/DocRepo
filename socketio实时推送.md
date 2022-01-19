title: socketio实时推送
author: tr
date: 2021-03-17 10:55:58
tags:
---
# socketio 实时推送

不仅是简单的建立连接响应，还要每隔一段时间推送数据到客户端
<!--more-->

## pom文件

添加依赖
```xml
<dependency>
  <groupId>com.corundumstudio.socketio</groupId>
  <artifactId>netty-socketio</artifactId>
  <version>1.7.17</version>
</dependency>
<dependency>
  <groupId>com.alibaba</groupId>
  <artifactId>fastjson</artifactId>
  <version>1.2.56</version>
</dependency>
```

## yml 配置

```yml
socketio:
  port: 9922		#监听端口
  # host: 127.0.0.1	#监听的ip
  # 设置最大每帧处理数据的长度，防止他人利用大数据来攻击服务器
  # maxFramePayloadLength: 1048576
  # 设置http交互最大内容长度
  # maxHttpContentLength: 1048576
  # socket连接数大小（如只监听一个端口boss线程组为1即可）
  # bossCount: 1
  # workCount: 100
  # allowCustomRequests: true
  # 协议升级超时时间（毫秒），默认10秒。HTTP握手升级为ws协议超时时间
  # upgradeTimeout: 1000000
  # Ping消息超时时间（毫秒），默认60秒，这个时间间隔内没有接收到心跳消息就会发送超时事件
  # pingTimeout: 6000000
  # Ping消息间隔（毫秒），默认25秒。客户端向服务器发送一条心跳消息间隔
  # pingInterval: 25000
```

## socketio 配置

@Configuration 和 @Bean 将spring管理的socketIOServer配置为我们自定义的server

```java

import cc.crrc.manage.mq.RabbitmqChannelListener;
import com.corundumstudio.socketio.SocketIOClient;
import com.corundumstudio.socketio.SocketIOServer;
import com.corundumstudio.socketio.listener.ExceptionListener;
import io.netty.channel.ChannelHandlerContext;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

import java.net.Inet4Address;
import java.net.UnknownHostException;
import java.util.List;


@Configuration
public class SocketIOConfiguration {
    @Value("${socketio.port:''}")
    private int socketIOPort;
    private static final Logger logger = LoggerFactory.getLogger(SocketIOConfiguration.class);

    @Bean("socketIOServer")
    public SocketIOServer socketIOServer() throws UnknownHostException {
        com.corundumstudio.socketio.Configuration configuration = new com.corundumstudio.socketio.Configuration();
        configuration.setPort(socketIOPort);
        configuration.setExceptionListener(new ExceptionListener() {
            @Override
            public void onEventException(Exception e, List<Object> list, SocketIOClient socketIOClient) {
                logger.error("客户端:" + socketIOClient.getRemoteAddress() + " EventException：" +e.getMessage());
            }

            @Override
            public void onDisconnectException(Exception e, SocketIOClient socketIOClient) {
                logger.error("客户端:" + socketIOClient.getRemoteAddress() + " DisconnectException：" +e.getMessage());
            }

            @Override
            public void onConnectException(Exception e, SocketIOClient socketIOClient) {
                logger.error("客户端:" + socketIOClient.getRemoteAddress() + " ConnectException：" +e.getMessage());
            }

            @Override
            public void onPingException(Exception e, SocketIOClient socketIOClient) {
                logger.error("客户端:" + socketIOClient.getRemoteAddress() + " PingException：" +e.getMessage());
            }

            @Override
            public boolean exceptionCaught(ChannelHandlerContext channelHandlerContext, Throwable throwable) throws Exception {
                return true;
            }
        });
//        configuration.setOrigin("*");
//        configuration.setSocketConfig();
//        configuration.setWorkerThreads(1);
//         configuration.setAllowCustomRequests(true);
//        configuration.setUpgradeTimeout(10000);
//        configuration.setPingTimeout(60000);
//        configuration.setPingInterval(25000);
//        configuration.setMaxHttpContentLength(2071738);
//        configuration.setMaxFramePayloadLength(2071738);
        return new SocketIOServer(configuration);
    }

```
`configuration.setExceptionListener(new ExceptionListener() {...`可以不用写 这里只是重写了异常监听，用于记录日志到系统。


## 定义线程处理接口

```java
public interface MonitorProcessor {
    Object process(String value);
}
```
## 实现线程处理接口

处理传递过来的message 产生返回消息
```java
@Service
public class GPSProcessor implements MonitorProcessor {

    private static final Logger logger = LoggerFactory.getLogger(GPSProcessor.class);

    @Autowired
    private MtrVehicleService vehicleService;

    @Override
    public Object process(String message) {
        List<Map<String, String>> result = new LinkedList<>();
        result.put("name","tr");
        return JSONObject.toJSONString(result);
    }
}
```
## 映射主题和处理类

将所有类根据名字存入map中

```java

public enum SocketRoute {

    LINE_MONITOR("lineMonitor", "/line", LineMonitorProcessor.class, 2000),
    VEHICLE_MONITOR("trainMonitor", "/vehicle", VehicleMonitorProcessor.class, 1000),
    VEHICLE_GPS("GPSMonitor", "/gps", GPSProcessor.class, 2000),
    VEHICLE_FAULT("faultInform", "/fault", GlobalFaultInformProcessor.class, 1000);
    private static final Map<String, SocketRoute> routes = new HashMap<>();

    static {
        for (SocketRoute route : SocketRoute.values()) {
            routes.put(route.getCmd(),route);
        }
    }

    private Class<? extends MonitorProcessor> monitorClass;
    private String nameSpace;
    private long intervalTime;
    private String cmd;

    SocketRoute(String cmd, String nameSpace, Class<? extends MonitorProcessor> monitorClass, int intervalTime) {
        this.cmd = cmd;
        this.nameSpace = nameSpace;
        this.monitorClass = monitorClass;
        this.intervalTime = intervalTime;
    }


    public Class<? extends MonitorProcessor> getMonitorClass() {
        return monitorClass;
    }

    public String getNameSpace() {
        return nameSpace;
    }

    public long getIntervalTime() {
        return intervalTime;
    }

    public String getCmd() {
        return cmd;
    }

    public  static SocketRoute getSocketRoute(String cmd){
        return routes.get(cmd);
    }
}

```


## 创建server

配置socketIOServer启动类，并且在spring容器初始化后启动，有两种方式实现自动启动 

1. 实现CommandLineRunner接口 实现里面的run方法

2. @PostConstruct标签描述方法，被描述的方法会自动在构建后执行

3. 以下这种实现是多线程共享一个socket通道，如果数据量大还是和前端建立多个通道好。

前端后端约定监听`talk`事件，且发送的json内容的两个key分别为`cmd`和`message`，定义一个SocketRoute的enum 保存不同cmd的处理事件和间隔时间。假设发送的json为`{"cmd":"GPSMonitor","message":{"vehicle":"t"}}`，通过枚举类找到 GPSMonitor为key的值的类，将该类的对象作为线程的内部对象，启动新的线程，首先根据key取消之前的线程，再重写启动新的线程，内部调用传递类的process方法。


处理线程内部根据是否循环loop和循环间隔interval 不停调用处理对象的process方法和`socketIOClient.sendEvent("message", returnVal);`返回客户端。

```java
@Component
public class SocketIOStartup implements CommandLineRunner {
    private static final Logger logger = LoggerFactory.getLogger(SocketIOStartup.class);
    // 保存当前运行的socket线程 一个key一个线程
    private static final ConcurrentHashMap<String, SocketThread> socketThreadMap = new ConcurrentHashMap<>();

    @Autowired
    private SocketIOServer socketIOServer;
    
    private void startUpServer() {
        bindListener();
        socketIOServer.startAsync();
    }

    private void bindListener() {
        connect();
        disconnect();
        bindEventListener();
    }
	// 可以不写，这里重写connect的监听方法用于日志记录
    private void connect() {
        socketIOServer.addConnectListener(new ConnectListener() {

            @Override
            public void onConnect(SocketIOClient socketIOClient) {
                logger.info("{}已连接", socketIOClient.getRemoteAddress().toString() + socketIOClient.getSessionId());
            }
        });
    }
	// 可以不写，这里重写disconnect的监听方法用于日志记录
    private void disconnect() {
        socketIOServer.addDisconnectListener(new DisconnectListener() {
            @Override
            public void onDisconnect(SocketIOClient socketIOClient) {
                SocketThread st = socketThreadMap.get(socketIOClient.getSessionId().toString());
                if (st != null) {
                    st.setLoop(false);
                    socketThreadMap.remove(socketIOClient.getSessionId().toString());
                }
                logger.info("{}已关闭", socketIOClient.getSessionId());
            }
        });
    }
	// 绑定监听事件：cmd 实现监听方法 前端也要发送该事件才能调用这个方法：socket.emit("cmd"..
    private void bindEventListener() {
        socketIOServer.addEventListener("talk", Object.class, new DataListener<Object>() {
            @Override
            public void onData(SocketIOClient socketIOClient, Object value, AckRequest ackRequest) throws Exception {
                sendMessage(socketIOClient, value);
            }
        });
    }
	// 消息处理方法
    private void sendMessage(SocketIOClient socketIOClient, Object object) throws JsonParseException {
        String cmd;
        String value;
        try {
            String jsonString = JSONObject.toJSONString(object);
            JSONObject cmdJson = JSONObject.parseObject(jsonString);
            cmd = cmdJson.getString("cmd");
            value = cmdJson.getString("message");
        } catch (Exception e) {
            throw new RestApiException(ExceptionInfoEnum.DATA_PARSE_EXCEPTION);
        }
//        } else {
//            throw new RestApiException(ExceptionInfoEnum.URL_PARAMETER_MISMATCH_EXCEPTION);
//        }
		//得到处理类
        SocketRoute route = SocketRoute.getSocketRoute(cmd);
        if (route == null) {
            throw new RuntimeException("找不到该cmd:" + cmd + " 对应的处理方法");
        }
        MonitorProcessor processor = SpringBeanUtils.getBean(route.getMonitorClass());
        // 将该处理类作为包装成线程启动
        startThread(socketIOClient, processor, value, route.getIntervalTime());
    }


    private void startThread(SocketIOClient socketIOClient, MonitorProcessor processor, String value, long interval) {
        SocketThread st = socketThreadMap.get(socketIOClient.getSessionId().toString());
        if (st != null) {
            st.setLoop(false);
            socketThreadMap.remove(socketIOClient.getSessionId().toString());
        }
        SocketThread st_new = new SocketThread(socketIOClient, processor, value, interval);
        socketThreadMap.put(socketIOClient.getSessionId().toString(), st_new);
        Thread t = new Thread(st_new);
        t.start();
    }

    class SocketThread implements Runnable {
        private SocketIOClient socketIOClient;
        private MonitorProcessor processor;
        private String value;
        private long interval;
        private boolean loop = true;

        public SocketThread(SocketIOClient socketIOClient, MonitorProcessor processor, String value, long interval) {
            this.socketIOClient = socketIOClient;
            this.processor = processor;
            this.value = value;
            this.interval = interval;
        }

        public void setLoop(boolean loop) {
            this.loop = loop;
        }

        @Override
        public void run() {
            while (loop && socketIOClient.isChannelOpen()) {
                Object returnVal = processor.process(value);
                if (loop && socketIOClient.isChannelOpen()) {
                    socketIOClient.sendEvent("message", returnVal);
                }
                try {
                    Thread.sleep(interval);
                } catch (InterruptedException e) {
                    logger.error("Socket clientId:{} {}", socketIOClient.getSessionId(), e.getMessage());
                }
            }
        }
    }

    @Override
    public void run(String... args) {
        startUpServer();
    }

}

```
## 前端测试

npm install socketio

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
    <script src="http://libs.baidu.com/jquery/2.0.0/jquery.min.js"></script>
    <!--<script src="/js/socketio/socket.io.js"></script>-->
    <script src="https://cdn.bootcss.com/socket.io/2.3.0/socket.io.js"></script>
    <script type="text/javascript">
        var first = true;
        $(function(){
         var socket ;
           $("#connectBtn").on("click",function(){
                if(first){
                socket = io.connect('http://localhost:9922');
                 console.log(socket)
                socket.on('connect', function () {
                    $("#content").text("连接成功")
                });
                 socket.on('disconnect', function () {
                   $("#content").text("连接断开")
                });
                socket.on("message",function(data){
                    $("#content").text(data);
                })
                first = false;
                }else{
                    socket.connect();
                }

           })


           $("#cancelBtn").on("click",function(){
                socket.disconnect();
                console.log(socket);
           });

           $("#sendBtn").on("click",function(){
                socket.emit("cmd",JSON.parse($("#message").val()));
           });

        });
    </script>
</head>
<body>
<input id="connectBtn" type="button" value="连接">
<button type="button" id="cancelBtn" class="btn">断开</button>
<button type="button" id="sendBtn" class="btn">发送</button>
<div id="content"></div>

<div>
    <textarea id="message"></textarea>
</div>
</body>
</html>
```