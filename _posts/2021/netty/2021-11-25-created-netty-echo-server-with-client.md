---
title:  "[Netty] EchoServer, EchoClient 생성해보기"
excerpt: "Netty EchoServer, Client 생성과 사족"

categories:
  - Netty
tags:
  - [Netty, JAVA, Socket]
last_modified_at: 2021-11-25T23:32:00+09:00

toc: true
toc_sticky: true

share: true
comments: true
---
## 1. 들어가며

이번에는 이전 Netty로 DiscardServer 를 작성 해본것에 조금 더 응용하여 EchoServer와 EchoClient를 구현 해보겠습니다.

![Untitled](/assets/image/netty/created-netty-echo-server-with-client//Untitled.png)

## 2. EchoServer

```java
package echo;

import io.netty.bootstrap.ServerBootstrap;
import io.netty.channel.ChannelFuture;
import io.netty.channel.ChannelInitializer;
import io.netty.channel.ChannelPipeline;
import io.netty.channel.EventLoopGroup;
import io.netty.channel.nio.NioEventLoopGroup;
import io.netty.channel.socket.SocketChannel;
import io.netty.channel.socket.nio.NioServerSocketChannel;

public class EchoServer {

    public static void main(String[] args) {
        EventLoopGroup bossGroup = new NioEventLoopGroup(1);                        // [1]
        EventLoopGroup workerGroup = new NioEventLoopGroup();                                  // [2]

        try {
            ServerBootstrap bootstrap = new ServerBootstrap();
            bootstrap.group(bossGroup, workerGroup)
                    .channel(NioServerSocketChannel.class)
                    .childHandler(new ChannelInitializer<SocketChannel>() {
                        @Override
                        protected void initChannel(SocketChannel ch) throws Exception {        // [3]
                            ChannelPipeline pipeline = ch.pipeline();
                            pipeline.addLast(new EchoServerHandler());
                        }
                    });

            ChannelFuture future = bootstrap.bind(8888);                             // [4]
            future.channel().closeFuture().sync();

        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            workerGroup.shutdownGracefully();
            bossGroup.shutdownGracefully();
        }
    }

}
```

<p>[1]: 실제 요청만 받아줄 EventLoopGroup 1개를 선언해줍니다.</p>

<p>[2]: bossGroup에서 요청 받은 것을 처리해 주는 workerGroup을 선언해줍니다. NioEventLoopGroup을 객체 생성 할때 아무 값도 넣지 않는경우. NettyRuntime.*availableProcessors*() * 2) 의 갯수로 스레드가 생성이 됩니다.</p>

<p>[3]: workerGroup에 Channel이 초기화 될 경우 EchoServerHandler라는 Class를 생성하며 Channel의 Pipeline에 추가합니다.</p>

<p>[4]: 8888 포트로 Socket 접속 할 수 있도록, ServerBootstarp에 포트를 바인딩 시켜줍니다.</p>

## 3. EchoServerHandler

```java
package echo;

import io.netty.buffer.ByteBuf;
import io.netty.channel.ChannelHandlerContext;
import io.netty.channel.SimpleChannelInboundHandler;

import java.nio.charset.Charset;

public class EchoServerHandler extends SimpleChannelInboundHandler<Object> {

    @Override
    protected void channelRead0(ChannelHandlerContext ctx, Object msg) throws Exception {
        String clientMessage = ((ByteBuf) msg).retain().toString(Charset.defaultCharset());         // [1]
        System.out.printf("클라이언트 메세지: %s%n", clientMessage);

        ctx.writeAndFlush(msg);                                                                     // [2]
    }

    @Override
    public void exceptionCaught(ChannelHandlerContext ctx, Throwable cause) throws Exception {
        cause.printStackTrace();
        ctx.close();
    }
}
```

<p>[1]: EchoClient에서 수신받은 ByteBuf의 사용 횟수를 retain메서드를 통하여 1번더 증가시켜줍니다.</p>

<p>[2]: EchoClient에서 수신받은 메세지를 그대로 다시 전달 해줍니다.</p>

- ChannelRead0 Method의 경우 채널에 입력이 들어 왔을때 수행되는 메서드 입니다.

## 4. EchoClient

```java
package echo;

import io.netty.bootstrap.Bootstrap;
import io.netty.channel.ChannelFuture;
import io.netty.channel.ChannelInitializer;
import io.netty.channel.ChannelPipeline;
import io.netty.channel.EventLoopGroup;
import io.netty.channel.nio.NioEventLoopGroup;
import io.netty.channel.socket.SocketChannel;
import io.netty.channel.socket.nio.NioSocketChannel;

public class EchoClient {

    public static void main(String[] args) {
        EventLoopGroup group = new NioEventLoopGroup();
        try {
            Bootstrap bootstrap = new Bootstrap();                                          // [1]
            bootstrap.group(group)
                    .channel(NioSocketChannel.class)
                    .handler(new ChannelInitializer<SocketChannel>() {
                        @Override
                        protected void initChannel(SocketChannel ch) throws Exception {
                            ChannelPipeline pipeline = ch.pipeline();
                            pipeline.addLast(new EchoClientHandler());                      // [2]
                        }
                    });

            ChannelFuture future = bootstrap.connect("localhost" , 8888).sync ();       // [3]
            future.channel().closeFuture().sync();

        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            group.shutdownGracefully();
        }
    }

```

<p>[1]: SocketClient측에선 ServerBootstrap객체가 아닌, Bootstrap객체로 소켓 서버에 접근 해야 합니다.</p>

<p>[2]: EchoClient의 Channel이 초기화 되었을때 Handler를 Pipeline에 추가해 줍니다.</p>

<p>[3]: EchoServer의 접속 정보를 EchoClient를 실행할때 Connect 할 수 있도록 접속 정보를 입력해줍니다.</p>

## 5. EchoClientHandler

```java
package echo;

import io.netty.buffer.ByteBuf;
import io.netty.buffer.Unpooled;
import io.netty.channel.ChannelHandlerContext;
import io.netty.channel.SimpleChannelInboundHandler;

import java.nio.charset.Charset;
import java.nio.charset.StandardCharsets;

public class EchoClientHandler extends SimpleChannelInboundHandler<Object> {

    @Override
    public void channelActive(ChannelHandlerContext ctx) throws Exception {                // [1]
        String message = "Echo";
        ByteBuf byteBuf = Unpooled.buffer();
        byteBuf.writeBytes(message.getBytes(StandardCharsets.UTF_8));
        ctx.writeAndFlush(byteBuf);                                                        
    }

    @Override
    protected void channelRead0(ChannelHandlerContext ctx, Object msg) throws Exception {
        String serverMessage = ((ByteBuf) msg).toString(Charset.defaultCharset());
        System.out.println(serverMessage);                                                 // [2]
    }

    @Override
    public void channelReadComplete(ChannelHandlerContext ctx) throws Exception {          // [3]
        System.out.println("클라이언트를 종료할게요.");
        ctx.close();
    }

    @Override
    public void exceptionCaught(ChannelHandlerContext ctx, Throwable cause) throws Exception {
        cause.printStackTrace();
        ctx.close();
    }

}
```

<p>[1]: EchoClient의 Channel이 활성화 되었을때 Echo라는 메세지를 ByteBuf에 담아서 EchoServer로 전달 시킵니다.</p>

<p>[2]: EchoServer에서 전달받은 메세지를 system.out.println으로 콘솔에 출력합니다.</p>

<p>[3]: channelRead0 메서드 실행 후 실행 되는 이벤트이며 종료 안내 메세지를 띄우고 Channel을 종료시킵니다.</p>

## 6. 실행 결과

### EchoServer

![Untitled](/assets/image/netty/created-netty-echo-server-with-client//Untitled%201.png)

### EchoClient

![Untitled](/assets/image/netty/created-netty-echo-server-with-client//Untitled%202.png)

## 7. 마치며

이번에는 EchoServer, EchoClient를 작성하여 실행해보았는데요, 다음 글엔 왜 ByteBuf를 사용하는지 channelActive, channelRead0 등의 이벤트가 발생하는 시점이 언제인지, Netty 아키텍쳐를 웹 채팅을 구현하며 같이 다른 게시글로 작성 해 보겠습니다.