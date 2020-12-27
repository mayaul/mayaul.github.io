---
layout: post
title: "Spring Webflux & react.js"
excerpt: "Spring Webflux 환경에서 react.js 개발하기"
date: 2020-12-26 00:00:00 +0900
comments: true
published : true
tag:
- spring boot
- spring boot webflux
- react
- type-script
---
### 들어가면서
* 백엔드 개발이 99%인 팀에서 가끔씩 admin 을 개발을 할 필요가 있습니다. 
* 쿠팡에서 재직을 하고있었을 때는 jquery 기반에서 개발을 했었는데요, 지금 여기에서는 react, typescript 를 통해서 개발을 하고 있습니다. 
* front-end 개발을 안한지가 너무 오래 되어서 그런지 처음에 적응이 너무 힘들었습니다. 
  - [인프런](https://www.inflearn.com/) 과 [유튜브](https://www.youtube.com/) 를 통해서 좀 학습을 했습니다.
  - 저 같은 경우는 유튜브로 통해서 무료강좌를 하나 듣고, 인프런으로 2개강좌를 결제해서 들었습니다.
* 강좌를 많이 보면 많은 강사들이 VisualStudioCode 를 사용을 하고 있는데요, 저는 주로 백엔드 개발자이라서 사용하는 툴이 IntelliJ 입니다.

### 일단은 react 기본 예제 프로그램 만들기
* 일단 기본적으로 Gradle 을 이용해서 프로젝트 구조를 잡은 다음에 `/src/main` 하위에 `front` 라는 디렉토리를 생성을 했습니다.
  ![[project directory]](/assets/img/posts/spring_webflux_react/web-flux-with-react-1.png)
* CRA 를 이용해서 샘플 react 페이지를 만들었습니다.
  - command 창에서 `/src/main/front` 에서 아래 명령어를 실행을 합니다.
```shell
npx create-react-app ./ --template typescript
```
* 이제 `npm run start` 명령어를 실행을 하면 `http://localhost:3000`으로 웹페이지가 정상적으로 열릴거에요.
  ![npm run start](/assets/img/posts/spring_webflux_react/web-flux-with-react-2.png)
  
### 이제는 Spring Boot 에서 react 페이지 수행하기
* 대부분의 웹페이지도 정적인 페이지만 제공을 하지는 않습니다. 
* admin 도 마찬가지로, API 호출을 통해서 `GET/PUT/POST/DELETE` 등을 요청을 처리해야합니다.
* build.gradle

``` groovy
plugins {
    id 'java'
    id 'org.springframework.boot' version '2.4.1'
    id 'com.github.node-gradle.node' version '2.2.4'
}

apply plugin: 'java-library'
apply plugin: 'org.springframework.boot'
apply plugin: 'io.spring.dependency-management'

dependencies {
    implementation('org.springframework.boot:spring-boot-starter-webflux')
    implementation('org.projectlombok:lombok')
    annotationProcessor('org.projectlombok:lombok')
}

task adminNpmInstall(type: NpmTask) {
  workingDir = file("src/main/front/")
  args = ['install']
}

task npmBuild(type: NpmTask, dependsOn: adminNpmInstall) {
  workingDir = file("src/main/front/")
  args = ['run', 'build']
}


task staticResourcesSync(type: Sync, dependsOn: npmBuild) {
  from 'src/main/front/build'
  into 'src/main/resources/static'
}

compileJava.dependsOn staticResourcesSync

task staticResourcesSyncForDeploy(type: Sync) {
  from 'src/main/front/build'
  into 'build/resources/main/static'
}

bootJar {
  archiveFileName = "mayaul-admin.jar"
  dependsOn 'staticResourcesSyncForDeploy'
}
```
  - 이번주제에서 다룰 webflux 환경으로 설정을 하였습니다. 
  - compile 을 했을 때, react 를 build 후 결과물을 resources 디렉토리로 copy 하는 gradle 명령어를 추가를 했습니다.

* 이제 SpringBootApplication 을 실행해보면...
  - 아래 이미지와 같은 404 에러 페이지를 만나게 됩니다.
    ![404 error](/assets/img/posts/spring_webflux_react/web-flux-with-react-3.png)
    
### 오류 해결하기 
* Controller 를 추가해서 `/resource/static/` 에 있는 `index.html` 로 redirect 를 시켜봅니다.
  - 이전 servlet 환경에서는 아래와 같이 했었습니다
```java
@Controller
public class AdminWebController {

    private static final String INDEX_PAGE = "forward:/index.html";

    @GetMapping(path = {"/"})
    public String home() { return INDEX_PAGE; }
}
```

  - 이제는 404 오류는 아니만 500에러가 발생하고 아래와 같은 오류메시지를 보게 됩니다.
``` 
java.lang.IllegalStateException: Could not resolve view with name 'forward:/index.html'.
	at org.springframework.web.reactive.result.view.ViewResolutionResultHandler.lambda$resolveViews$3(ViewResolutionResultHandler.java:278) ~[spring-webflux-5.3.2.jar:5.3.2]
	Suppressed: reactor.core.publisher.FluxOnAssembly$OnAssemblyException: 
Error has been observed at the following site(s):
	|_ checkpoint ⇢ Handler io.github.mayaul.controller.HomeController#home() [DispatcherHandler]
	|_ checkpoint ⇢ HTTP GET "/" [ExceptionHandlingWebHandler]
Stack trace:
		at org.springframework.web.reactive.result.view.ViewResolutionResultHandler.lambda$resolveViews$3(ViewResolutionResultHandler.java:278) ~[spring-webflux-5.3.2.jar:5.3.2]
		at reactor.core.publisher.FluxMapFuseable$MapFuseableSubscriber.onNext(FluxMapFuseable.java:113) ~[reactor-core-3.4.1.jar:3.4.1]
		at reactor.core.publisher.Operators$MonoSubscriber.complete(Operators.java:1784) ~[reactor-core-3.4.1.jar:3.4.1]
		at reactor.core.publisher.MonoCollectList$MonoCollectListSubscriber.onComplete(MonoCollectList.java:128) ~[reactor-core-3.4.1.jar:3.4.1]
		at reactor.core.publisher.FluxConcatMap$ConcatMapImmediate.drain(FluxConcatMap.java:366) ~[reactor-core-3.4.1.jar:3.4.1]
		at reactor.core.publisher.FluxConcatMap$ConcatMapImmediate.onComplete(FluxConcatMap.java:275) ~[reactor-core-3.4.1.jar:3.4.1]
		at reactor.core.publisher.Operators.complete(Operators.java:135) ~[reactor-core-3.4.1.jar:3.4.1]
		at reactor.core.publisher.FluxIterable.subscribe(FluxIterable.java:147) ~[reactor-core-3.4.1.jar:3.4.1]
		at reactor.core.publisher.FluxIterable.subscribe(FluxIterable.java:86) ~[reactor-core-3.4.1.jar:3.4.1]
		at reactor.core.publisher.InternalMonoOperator.subscribe(InternalMonoOperator.java:64) ~[reactor-core-3.4.1.jar:3.4.1]
		at reactor.core.publisher.MonoFlatMap$FlatMapMain.onNext(MonoFlatMap.java:157) ~[reactor-core-3.4.1.jar:3.4.1]
		at reactor.core.publisher.FluxSwitchIfEmpty$SwitchIfEmptySubscriber.onNext(FluxSwitchIfEmpty.java:73) ~[reactor-core-3.4.1.jar:3.4.1]
		at reactor.core.publisher.Operators$ScalarSubscription.request(Operators.java:2346) ~[reactor-core-3.4.1.jar:3.4.1]
		at reactor.core.publisher.Operators$MultiSubscriptionSubscriber.set(Operators.java:2154) ~[reactor-core-3.4.1.jar:3.4.1]
		at reactor.core.publisher.Operators$MultiSubscriptionSubscriber.onSubscribe(Operators.java:2028) ~[reactor-core-3.4.1.jar:3.4.1]
		at reactor.core.publisher.MonoJust.subscribe(MonoJust.java:54) ~[reactor-core-3.4.1.jar:3.4.1]
		at reactor.core.publisher.InternalMonoOperator.subscribe(InternalMonoOperator.java:64) ~[reactor-core-3.4.1.jar:3.4.1]
		at reactor.core.publisher.MonoFlatMap$FlatMapMain.onNext(MonoFlatMap.java:157) ~[reactor-core-3.4.1.jar:3.4.1]
		at reactor.core.publisher.Operators$MonoSubscriber.complete(Operators.java:1784) ~[reactor-core-3.4.1.jar:3.4.1]
		at reactor.core.publisher.MonoFlatMap$FlatMapInner.onNext(MonoFlatMap.java:249) ~[reactor-core-3.4.1.jar:3.4.1]
		at reactor.core.publisher.FluxOnErrorResume$ResumeSubscriber.onNext(FluxOnErrorResume.java:79) ~[reactor-core-3.4.1.jar:3.4.1]
		at reactor.core.publisher.FluxPeekFuseable$PeekFuseableSubscriber.onNext(FluxPeekFuseable.java:210) ~[reactor-core-3.4.1.jar:3.4.1]
		at reactor.core.publisher.FluxPeekFuseable$PeekFuseableSubscriber.onNext(FluxPeekFuseable.java:210) ~[reactor-core-3.4.1.jar:3.4.1]
		at reactor.core.publisher.Operators$MonoSubscriber.complete(Operators.java:1784) ~[reactor-core-3.4.1.jar:3.4.1]
		at reactor.core.publisher.MonoIgnoreThen$ThenAcceptInner.onNext(MonoIgnoreThen.java:305) ~[reactor-core-3.4.1.jar:3.4.1]
		at reactor.core.publisher.Operators$ScalarSubscription.request(Operators.java:2346) ~[reactor-core-3.4.1.jar:3.4.1]
		at reactor.core.publisher.MonoIgnoreThen$ThenAcceptInner.onSubscribe(MonoIgnoreThen.java:294) ~[reactor-core-3.4.1.jar:3.4.1]
		at reactor.core.publisher.FluxFlatMap.trySubscribeScalarMap(FluxFlatMap.java:191) ~[reactor-core-3.4.1.jar:3.4.1]
		at reactor.core.publisher.MonoFlatMap.subscribeOrReturn(MonoFlatMap.java:53) ~[reactor-core-3.4.1.jar:3.4.1]
		at reactor.core.publisher.InternalMonoOperator.subscribe(InternalMonoOperator.java:57) ~[reactor-core-3.4.1.jar:3.4.1]
		at reactor.core.publisher.MonoDefer.subscribe(MonoDefer.java:52) ~[reactor-core-3.4.1.jar:3.4.1]
		at reactor.core.publisher.MonoIgnoreThen$ThenIgnoreMain.drain(MonoIgnoreThen.java:154) ~[reactor-core-3.4.1.jar:3.4.1]
		at reactor.core.publisher.MonoIgnoreThen.subscribe(MonoIgnoreThen.java:56) ~[reactor-core-3.4.1.jar:3.4.1]
		at reactor.core.publisher.InternalMonoOperator.subscribe(InternalMonoOperator.java:64) ~[reactor-core-3.4.1.jar:3.4.1]
		at reactor.core.publisher.MonoFlatMap$FlatMapMain.onNext(MonoFlatMap.java:157) ~[reactor-core-3.4.1.jar:3.4.1]
		at reactor.core.publisher.FluxSwitchIfEmpty$SwitchIfEmptySubscriber.onNext(FluxSwitchIfEmpty.java:73) ~[reactor-core-3.4.1.jar:3.4.1]
		at reactor.core.publisher.MonoNext$NextSubscriber.onNext(MonoNext.java:82) ~[reactor-core-3.4.1.jar:3.4.1]
		at reactor.core.publisher.FluxConcatMap$ConcatMapImmediate.innerNext(FluxConcatMap.java:281) ~[reactor-core-3.4.1.jar:3.4.1]
		at reactor.core.publisher.FluxConcatMap$ConcatMapInner.onNext(FluxConcatMap.java:860) ~[reactor-core-3.4.1.jar:3.4.1]
		at reactor.core.publisher.FluxMapFuseable$MapFuseableSubscriber.onNext(FluxMapFuseable.java:127) ~[reactor-core-3.4.1.jar:3.4.1]
		at reactor.core.publisher.MonoPeekTerminal$MonoTerminalPeekSubscriber.onNext(MonoPeekTerminal.java:180) ~[reactor-core-3.4.1.jar:3.4.1]
		at reactor.core.publisher.Operators$ScalarSubscription.request(Operators.java:2346) ~[reactor-core-3.4.1.jar:3.4.1]
		at reactor.core.publisher.MonoPeekTerminal$MonoTerminalPeekSubscriber.request(MonoPeekTerminal.java:139) ~[reactor-core-3.4.1.jar:3.4.1]
		at reactor.core.publisher.FluxMapFuseable$MapFuseableSubscriber.request(FluxMapFuseable.java:169) ~[reactor-core-3.4.1.jar:3.4.1]
		at reactor.core.publisher.Operators$MultiSubscriptionSubscriber.set(Operators.java:2154) ~[reactor-core-3.4.1.jar:3.4.1]
		at reactor.core.publisher.Operators$MultiSubscriptionSubscriber.onSubscribe(Operators.java:2028) ~[reactor-core-3.4.1.jar:3.4.1]
		at reactor.core.publisher.FluxMapFuseable$MapFuseableSubscriber.onSubscribe(FluxMapFuseable.java:96) ~[reactor-core-3.4.1.jar:3.4.1]
		at reactor.core.publisher.MonoPeekTerminal$MonoTerminalPeekSubscriber.onSubscribe(MonoPeekTerminal.java:152) ~[reactor-core-3.4.1.jar:3.4.1]
		at reactor.core.publisher.MonoJust.subscribe(MonoJust.java:54) ~[reactor-core-3.4.1.jar:3.4.1]
		at reactor.core.publisher.Mono.subscribe(Mono.java:4046) ~[reactor-core-3.4.1.jar:3.4.1]
		at reactor.core.publisher.FluxConcatMap$ConcatMapImmediate.drain(FluxConcatMap.java:448) ~[reactor-core-3.4.1.jar:3.4.1]
		at reactor.core.publisher.FluxConcatMap$ConcatMapImmediate.onSubscribe(FluxConcatMap.java:218) ~[reactor-core-3.4.1.jar:3.4.1]
		at reactor.core.publisher.FluxIterable.subscribe(FluxIterable.java:164) ~[reactor-core-3.4.1.jar:3.4.1]
		at reactor.core.publisher.FluxIterable.subscribe(FluxIterable.java:86) ~[reactor-core-3.4.1.jar:3.4.1]
		at reactor.core.publisher.InternalMonoOperator.subscribe(InternalMonoOperator.java:64) ~[reactor-core-3.4.1.jar:3.4.1]
		at reactor.core.publisher.MonoDefer.subscribe(MonoDefer.java:52) ~[reactor-core-3.4.1.jar:3.4.1]
		at reactor.core.publisher.Mono.subscribe(Mono.java:4046) ~[reactor-core-3.4.1.jar:3.4.1]
		at reactor.core.publisher.MonoIgnoreThen$ThenIgnoreMain.drain(MonoIgnoreThen.java:173) ~[reactor-core-3.4.1.jar:3.4.1]
		at reactor.core.publisher.MonoIgnoreThen.subscribe(MonoIgnoreThen.java:56) ~[reactor-core-3.4.1.jar:3.4.1]
		at reactor.core.publisher.InternalMonoOperator.subscribe(InternalMonoOperator.java:64) ~[reactor-core-3.4.1.jar:3.4.1]
		at reactor.netty.http.server.HttpServer$HttpServerHandle.onStateChange(HttpServer.java:632) ~[reactor-netty-http-1.0.2.jar:1.0.2]
		at reactor.netty.ReactorNetty$CompositeConnectionObserver.onStateChange(ReactorNetty.java:612) ~[reactor-netty-core-1.0.2.jar:1.0.2]
		at reactor.netty.transport.ServerTransport$ChildObserver.onStateChange(ServerTransport.java:453) ~[reactor-netty-core-1.0.2.jar:1.0.2]
		at reactor.netty.http.server.HttpServerOperations.onInboundNext(HttpServerOperations.java:510) ~[reactor-netty-http-1.0.2.jar:1.0.2]
		at reactor.netty.channel.ChannelOperationsHandler.channelRead(ChannelOperationsHandler.java:94) ~[reactor-netty-core-1.0.2.jar:1.0.2]
		at io.netty.channel.AbstractChannelHandlerContext.invokeChannelRead(AbstractChannelHandlerContext.java:379) ~[netty-transport-4.1.55.Final.jar:4.1.55.Final]
		at io.netty.channel.AbstractChannelHandlerContext.invokeChannelRead(AbstractChannelHandlerContext.java:365) ~[netty-transport-4.1.55.Final.jar:4.1.55.Final]
		at io.netty.channel.AbstractChannelHandlerContext.fireChannelRead(AbstractChannelHandlerContext.java:357) ~[netty-transport-4.1.55.Final.jar:4.1.55.Final]
		at reactor.netty.http.server.HttpTrafficHandler.channelRead(HttpTrafficHandler.java:208) ~[reactor-netty-http-1.0.2.jar:1.0.2]
		at io.netty.channel.AbstractChannelHandlerContext.invokeChannelRead(AbstractChannelHandlerContext.java:379) ~[netty-transport-4.1.55.Final.jar:4.1.55.Final]
		at io.netty.channel.AbstractChannelHandlerContext.invokeChannelRead(AbstractChannelHandlerContext.java:365) ~[netty-transport-4.1.55.Final.jar:4.1.55.Final]
		at io.netty.channel.AbstractChannelHandlerContext.fireChannelRead(AbstractChannelHandlerContext.java:357) ~[netty-transport-4.1.55.Final.jar:4.1.55.Final]
		at io.netty.channel.CombinedChannelDuplexHandler$DelegatingChannelHandlerContext.fireChannelRead(CombinedChannelDuplexHandler.java:436) ~[netty-transport-4.1.55.Final.jar:4.1.55.Final]
		at io.netty.handler.codec.ByteToMessageDecoder.fireChannelRead(ByteToMessageDecoder.java:324) ~[netty-codec-4.1.55.Final.jar:4.1.55.Final]
		at io.netty.handler.codec.ByteToMessageDecoder.channelRead(ByteToMessageDecoder.java:296) ~[netty-codec-4.1.55.Final.jar:4.1.55.Final]
		at io.netty.channel.CombinedChannelDuplexHandler.channelRead(CombinedChannelDuplexHandler.java:251) ~[netty-transport-4.1.55.Final.jar:4.1.55.Final]
		at io.netty.channel.AbstractChannelHandlerContext.invokeChannelRead(AbstractChannelHandlerContext.java:379) ~[netty-transport-4.1.55.Final.jar:4.1.55.Final]
		at io.netty.channel.AbstractChannelHandlerContext.invokeChannelRead(AbstractChannelHandlerContext.java:365) ~[netty-transport-4.1.55.Final.jar:4.1.55.Final]
		at io.netty.channel.AbstractChannelHandlerContext.fireChannelRead(AbstractChannelHandlerContext.java:357) ~[netty-transport-4.1.55.Final.jar:4.1.55.Final]
		at io.netty.channel.DefaultChannelPipeline$HeadContext.channelRead(DefaultChannelPipeline.java:1410) ~[netty-transport-4.1.55.Final.jar:4.1.55.Final]
		at io.netty.channel.AbstractChannelHandlerContext.invokeChannelRead(AbstractChannelHandlerContext.java:379) ~[netty-transport-4.1.55.Final.jar:4.1.55.Final]
		at io.netty.channel.AbstractChannelHandlerContext.invokeChannelRead(AbstractChannelHandlerContext.java:365) ~[netty-transport-4.1.55.Final.jar:4.1.55.Final]
		at io.netty.channel.DefaultChannelPipeline.fireChannelRead(DefaultChannelPipeline.java:919) ~[netty-transport-4.1.55.Final.jar:4.1.55.Final]
		at io.netty.channel.nio.AbstractNioByteChannel$NioByteUnsafe.read(AbstractNioByteChannel.java:166) ~[netty-transport-4.1.55.Final.jar:4.1.55.Final]
		at io.netty.channel.nio.NioEventLoop.processSelectedKey(NioEventLoop.java:719) ~[netty-transport-4.1.55.Final.jar:4.1.55.Final]
		at io.netty.channel.nio.NioEventLoop.processSelectedKeysOptimized(NioEventLoop.java:655) ~[netty-transport-4.1.55.Final.jar:4.1.55.Final]
		at io.netty.channel.nio.NioEventLoop.processSelectedKeys(NioEventLoop.java:581) ~[netty-transport-4.1.55.Final.jar:4.1.55.Final]
		at io.netty.channel.nio.NioEventLoop.run(NioEventLoop.java:493) ~[netty-transport-4.1.55.Final.jar:4.1.55.Final]
		at io.netty.util.concurrent.SingleThreadEventExecutor$4.run(SingleThreadEventExecutor.java:989) ~[netty-common-4.1.55.Final.jar:4.1.55.Final]
		at io.netty.util.internal.ThreadExecutorMap$2.run(ThreadExecutorMap.java:74) ~[netty-common-4.1.55.Final.jar:4.1.55.Final]
		at io.netty.util.concurrent.FastThreadLocalRunnable.run(FastThreadLocalRunnable.java:30) ~[netty-common-4.1.55.Final.jar:4.1.55.Final]
		at java.base/java.lang.Thread.run(Thread.java:834) ~[na:na]
```
![500 error](/assets/img/posts/spring_webflux_react/web-flux-with-react-4.png)

* webflux 환경에서 static resource 를 이용하기
  - webflux 에서 지원하는 몇가지 [view engine](https://docs.spring.io/spring-framework/docs/current/reference/html/web-reactive.html#webflux-view) 들이 있습니다. 
  - 이번에는 이런 view engine 은 필요가 없습니다. 
    + 저희는 react.js 를 사용할거니까 webpack 으로 만들어진 index.html 비롯한 resource 만 잘 읽어와 주면 됩니다.
  - 관련해서 몇가지를 찾아봤지만, 제가 생각하기에 제일 괜찮은 방법은 아래와 같습니다.
    + index.html 의 resource 를 읽어온 다음, 그걸 바로 응답값으로 전달을 하는 `RouterFunction` 을 만드는 방법이었습니다. 
  
* htmlRouter bean 등록하기
```java
@SpringBootApplication
public class Application {

    public static void main(String... args) {
        SpringApplication.run(Application.class, args);
    }

    @Bean
    public RouterFunction<ServerResponse> htmlRouter(@Value("classpath:/static/index.html") Resource html) {
        return RouterFunctions.route(
                GET("/"),
                request -> ServerResponse.ok().contentType(MediaType.TEXT_HTML).bodyValue(html)
        );
    }
}
```

* application.yml 에 static 경로 추가
``` groovy
spring.webflux.static-path-pattern: /static/**
```

* 이제는 404, 500 에러는 안나지만 뭔가 이상합니다. `npm run build`로 봤던 화면이 나오지 않고 개발자 도구로 보면 `css, js` 를 404 에러가 발생되면서 가지고 오지 못합니다.
  - index.html 페이지를 보면 `css, js` 경로들이 `/static/**` 이렇게 시작을 하고 있습니다. 
  - 실제로 해당 resource 경로를 브라우저에 호출해도 404가 발생을 하고, `/static/static/**` 이렇게 앞에 /static 을 한번 더 넣어줘야 해결이 됩니다. 

* 이것을 해결하기 위해서 저는 webpack 설정을 override 하는 방법으로 진행을 했습니다. 
  - package.json 수정하기
``` javascript
{
  //... 
  "dependencies": {
    //...
    "react-app-rewired": "^2.1.6"
    //...
  },
  "scripts": {
    "start": "react-app-rewired start",
    "build": "react-app-rewired build ",
    "test": "react-app-rewired test"
  },
  //... 
}
```
  - config-overrides.js 추가
``` javascript
module.exports = {
  webpack: function(config, env) {
    config.output.publicPath = env === 'development' ? '/' : '/static/';
    return config;
  }
};
```
* 이제 다시 빌드를 하고 띄워보면 `npm run start` 를 했을때와 같은 정상적인 화면을 볼 수 있습니다.
![success](/assets/img/posts/spring_webflux_react/web-flux-with-react-5.png)
### 참고 자료
> https://www.baeldung.com/spring-webflux-static-content