---
title:  "[Netty] Discard 서버 생성해보기"
excerpt: "Netty로 간단한 Discard 서버 구현"

categories:
  - Netty
tags:
  - [Netty, JAVA, Socket]
last_modified_at: 2021-11-24T00:06:00+09:00

toc: true
toc_sticky: true

share: true
comments: true
---


💡 작성자는 모든 과정을 IntelliJ 로 진행 하였습니다.

## 1. Project 생성하기

### 빌드 도구 설정

![Untitled](/assets/image/netty/created-netty-discard-server/Untitled.png)

New Project → `Gradle` 선택 → SDK와 사용 언어 선택 후 Next를 눌러주세요.

💡 꼭 `Gradle`로 선택 하지 않아도 되고, `Java`나 `Maven`을 사용 하셔도 됩니다.

### 프로젝트 명 설정

![Untitled](/assets/image/netty/created-netty-discard-server/Untitled%201.png)

Project Name을 설정 해주고 `Finish`를 눌러주세요.

### Netty 패키지 의존성 추가

![Untitled](/assets/image/netty/created-netty-discard-server/Untitled%202.png)

아래 링크에 접속해서 패키지 확인 후 `build.gradle`에서 implementation 시켜주세요.

> [https://mvnrepository.com/artifact/io.netty/netty-all](https://mvnrepository.com/artifact/io.netty/netty-all)
> 

## 2. DiscardServer 클래스 생성하기

```java
package discard;

import io.netty.bootstrap.ServerBootstrap;
import io.netty.channel.ChannelFuture;
import io.netty.channel.ChannelInitializer;
import io.netty.channel.ChannelPipeline;
import io.netty.channel.EventLoopGroup;
import io.netty.channel.nio.NioEventLoopGroup;
import io.netty.channel.socket.SocketChannel;
import io.netty.channel.socket.nio.NioServerSocketChannel;

public class DiscardServer {

    public static void main(String[] args) {
        EventLoopGroup bossGroup = new NioEventLoopGroup(1);
        EventLoopGroup workerGroup = new NioEventLoopGroup();

        try {
            ServerBootstrap serverBootstrap = new ServerBootstrap();
            serverBootstrap.group(bossGroup, workerGroup)
                    .channel(NioServerSocketChannel.class)
                    .childHandler(new ChannelInitializer<SocketChannel>() {
                        @Override
                        protected void initChannel(SocketChannel ch) throws Exception { // [4]
                            ChannelPipeline pipeline = ch.pipeline();
                            pipeline.addLast(new DiscardServerHandler());
                        }
                    });

            ChannelFuture future = serverBootstrap.bind(8888);
            future.channel().closeFuture().sync();

        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            bossGroup.shutdownGracefully();
            workerGroup.shutdownGracefully();
        }
    }

}
```

## 3. DiscardServerHandler 클래스 생성하기

```java
package ex1.discard;

import io.netty.channel.ChannelHandlerContext;
import io.netty.channel.SimpleChannelInboundHandler;

public class DiscardServerHandler extends SimpleChannelInboundHandler<Object> {

    @Override
    public void channelRead0(ChannelHandlerContext ctx, Object msg) throws Exception {
				// 아무 행동도 하지 않음.
    }

    @Override
    public void exceptionCaught(ChannelHandlerContext ctx, Throwable cause) throws Exception {
        cause.printStackTrace();
        ctx.close();
    }
}
```

## 4. 연결 해보기

![Untitled](/assets/image/netty/created-netty-discard-server/Untitled%203.png)

<aside>
💡 Discard 서버에 접속 실패 했을때

</aside>

![Untitled](/assets/image/netty/created-netty-discard-server/Untitled%204.png)

<aside>
💡 Discard 서버에 접속 하였을때

</aside>

## 5. 마치며

이번엔 단순히 SocketServer를 오픈 했지만 그 외 특별한 동작은 없는 DiscardServer에 대해 작성을 해보았고, 다음번엔 EchoServer, EchoClient에 대해서 작성해보겠습니다.