# FreeSWITCH ESL ALL

[![Codacy Badge](https://api.codacy.com/project/badge/Grade/23fb13f7487f4ccd985f09c96341dfab)](https://app.codacy.com/gh/zhouhailin/freeswitch-esl-all?utm_source=github.com&utm_medium=referral&utm_content=zhouhailin/freeswitch-esl-all&utm_campaign=Badge_Grade_Settings)
[![Jdk Version](https://img.shields.io/badge/JDK-8-green.svg)](https://img.shields.io/badge/JDK-8-green.svg)
[![Spring Boot](https://img.shields.io/badge/Spring%20Boot-2.x%20%7C%203.x%20%7C%204.x-brightgreen.svg)](https://spring.io/projects/spring-boot)
[![License](https://img.shields.io/badge/license-Apache%202-4EB1BA.svg)](https://www.apache.org/licenses/LICENSE-2.0)
[![Maven Central](https://img.shields.io/maven-central/v/link.thingscloud/freeswitch-externals)](https://mvnrepository.com/artifact/link.thingscloud/freeswitch-externals)
[![Gitter](https://badges.gitter.im/freeswitch-esl-all/community.svg)](https://gitter.im/freeswitch-esl-all/community?utm_source=badge&utm_medium=badge&utm_campaign=pr-badge)

## Summary

    JDK 1.8 +

    Netty used 4.1.x, 5.x is not supported

    Spring Boot 2.x / 3.x / 4.x, one starter for all of them, see Compatibility

## Features

    1. Support to connect FreeSWITCH Cluster
    2. Easy to integrate with FreeSWITCH ESL
    3. Support spring boot starter
    4. Support to dynamic add or remove remote server
    5. Support Spring Boot 2.x, 3.x and 4.x with the same artifact

## AI Skills (Qoder)

Two Qoder skills are maintained for this library, so an AI coding agent can wire it up correctly without reading the
whole source tree. They are documentation-only: no MCP server, no credentials, no network calls.

| Skill / plugin                                                            | Use it when                              | Covers                                                                                                                                                                                                                                                                                                 |
|---------------------------------------------------------------------------|------------------------------------------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `freeswitch-esl-spring-boot-starter` (FreeSWITCH ESL Spring Boot Starter) | the project is a Spring Boot application | properties binding under `link.thingscloud.freeswitch.esl.inbound`, the five auto-configured beans and what breaks when you replace them, `@EslEventName` handlers, `AbstractInboundClientOptionHandler`, runtime cluster endpoints, Spring Boot 2 / 3 / 4 differences, `@SpringBootTest` verification |
| `freeswitch-esl` (FreeSWITCH ESL 纯 Java 客户端)                              | the project has no Spring                | `InboundClientOption` + `InboundClient` bootstrap, `IEslEventListener`, `api` / `bgapi` commands and the `host:port` address rule, cluster add/remove, `ConnectState`, reconnect, thread and timeout tuning, FreeSWITCH-side checklist                                                                 |

Both also carry the failure modes that are easy to get wrong: `InboundClient` is a JVM-wide singleton,
`closeChannel(addr)` closes the ESL socket rather than a call, events you did not subscribe to are never delivered,
and sync commands are serialised per connection.

### Install

From the Qoder plugin marketplace, search `freeswitch-esl` and install the plugin you need, or install it by name:

```bash

qodercli plugin install freeswitch-esl-spring-boot-starter
```

Or install a local plugin directory / zip. `--scope` accepts `user` (default, global), `project` (shared and
committed to VCS) or `local` (personal, not committed):

```bash

qodercli plugin install --scope user /path/to/freeswitch-esl-spring-boot-starter
qodercli plugin validate /path/to/freeswitch-esl        # optional structure check
```

Or drop the skill folders into a project so the whole team gets them from git:

```text
<your-project>/.qoder/skills/freeswitch-esl/SKILL.md
<your-project>/.qoder/skills/freeswitch-esl/references/*.md
```

```text
<your-project>/.qoder/skills/freeswitch-esl-spring-boot-starter/SKILL.md
<your-project>/.qoder/skills/freeswitch-esl-spring-boot-starter/references/*.md
```

### Use

1. Reload skills (`/skills reload`) or restart the session, then confirm with `/skills list`.
2. Invoke explicitly with `/freeswitch-esl-spring-boot-starter` or `/freeswitch-esl`, or just describe the task in
   natural language - Chinese and English triggers both work (FreeSWITCH, ESL, event socket, originate, 事件订阅,
   呼叫控制, 话单).
3. Typical prompts: "add FreeSWITCH ESL inbound to this Boot 3 service with a hangup handler", "why does my
   `originate` block the thread", "expose an admin endpoint to add a switch node", "tune executor threads for
   200 events/s".

Keep the skills in step with the library: they pin examples at `2.2.0` and describe the current behaviour of
`InboundClient`, `InboundClientOption` and `FreeswitchEslAutoConfiguration`. After a breaking change, update both
skills together with this README.

## Maven

Latest release is `2.2.0`, see [Maven Central](https://mvnrepository.com/artifact/link.thingscloud/freeswitch-esl).

```xml

<dependency>
   <groupId>link.thingscloud</groupId>
   <artifactId>freeswitch-esl</artifactId>
   <version>${freeswitch-esl.version}</version>
</dependency>
```

## Quick Start

### freeswitch-esl-spring-boot-starter

[Maven Central](https://mvnrepository.com/artifact/link.thingscloud/freeswitch-esl-spring-boot-starter)

```xml

<dependency>
    <groupId>link.thingscloud</groupId>
    <artifactId>freeswitch-esl-spring-boot-starter</artifactId>
    <version>${freeswitch-esl.version}</version>
</dependency>
```

application.properties

```properties
link.thingscloud.freeswitch.esl.inbound.servers[0].host=127.0.0.1
link.thingscloud.freeswitch.esl.inbound.servers[0].port=8021
link.thingscloud.freeswitch.esl.inbound.servers[0].timeout-seconds=5
link.thingscloud.freeswitch.esl.inbound.servers[0].password=ClueCon
link.thingscloud.freeswitch.esl.inbound.servers[2].host=127.0.0.2
link.thingscloud.freeswitch.esl.inbound.servers[2].port=8021
link.thingscloud.freeswitch.esl.inbound.servers[2].timeout-seconds=5
link.thingscloud.freeswitch.esl.inbound.events=CHANNEL_CREATE CHANNEL_DESTORY 
# performance monitor - event driven - business logic processing time
link.thingscloud.freeswitch.esl.inbound.performance=true 
link.thingscloud.freeswitch.esl.inbound.performanceCostTime=200 
# event performance monitor - event driven - event generate time and event receive time
link.thingscloud.freeswitch.esl.inbound.eventPerformance=false 
link.thingscloud.freeswitch.esl.inbound.eventPerformanceCostTime=200
```

application.yml

```yaml
link:
  thingscloud:
    freeswitch:
      esl:
        inbound:
          defaultPassword: ClueCon
          performance: false
          performanceCostTime: 200
          #read-timeout-seconds: 0
          servers:
            - host: 127.0.0.1
              port: 8021
              timeoutSeconds: 5
            - host: 127.0.0.2
            - host: 127.0.0.3
          events:
            - CHANNEL_CREATE
            - CHANNEL_DESTORY
```

```java

@Slf4j
@Component
public class ExampleInboundClient {
    @Autowired
    private InboundClient inboundClient;

    @Autowired
    private InboundClientBootstrap inboundClientBootstrap;

    @PostConstruct
    public void startup() {
        System.out.println(inboundClientBootstrap);
    }

}
```

```java

@Slf4j
@Component
public class InboundClientOptionHandlerExample extends AbstractInboundClientOptionHandler {

    /**
     * {@inheritDoc}
     */
    @Override
    protected void intercept(InboundClientOption inboundClientOption) {
        List<ServerOption> serverOptions = inboundClientOption.serverOptions();
        log.info("serverOptions before : {}", serverOptions);
        serverOptions.clear();
        serverOptions.add(new ServerOption("127.0.0.8", 8021));
        log.info("serverOptions after  : {}", serverOptions);
    }
}
```

```java

@Slf4j
@Component
@EslEventName(EventNames.HEARTBEAT)
public class HeartbeatEslEventHandler implements EslEventHandler {
    /**
     * {@inheritDoc}
     */
    @Override
    public void handle(String addr, EslEvent event) {
        log.info("HeartbeatEslEventHandler handle addr[{}] EslEvent[{}].", addr, event);
    }
}
```

```java

@Slf4j
@Service
public class ServerConnectionListenerImpl implements ServerConnectionListener {
    /**
     * {@inheritDoc}
     */
    @Override
    public void onOpened(ServerOption serverOption) {
        log.info("onOpened serverOption : {}", serverOption);
    }

    /**
     * {@inheritDoc}
     */
    @Override
    public void onClosed(ServerOption serverOption) {
        log.info("onClosed serverOption : {}", serverOption);
    }
}
```

### freeswitch-esl

[Maven Central](https://mvnrepository.com/artifact/link.thingscloud/freeswitch-esl)

```xml

<dependency>
    <groupId>link.thingscloud</groupId>
    <artifactId>freeswitch-esl</artifactId>
    <version>${freeswitch-esl.version}</version>
</dependency>
```

```
InboundClientOption option = new InboundClientOption();

// set default esl password, if ServerOption not set password, use this default value
option.defaultPassword("ClueCon")
    .addServerOption(new ServerOption("127.0.0.1", 8021));

// set esl subscribe events
option.addEvents("all");

// ESL event listener
option.addListener(new IEslEventListener() {
    @Override
    public void eventReceived(String addr, EslEvent event) {
        System.out.println(addr);
        System.out.println(event);
    }

    @Override
    public void backgroundJobResultReceived(String addr, EslEvent event) {
        System.out.println(addr);
        System.out.println(event);
    }
});

option.serverConnectionListener(new ServerConnectionListener() {
    @Override
    public void onOpened(ServerOption serverOption) {
        System.out.println("---onOpened--");
    }

    @Override
    public void onClosed(ServerOption serverOption) {
        System.out.println("---onClosed--");
    }
});

InboundClient inboundClient = InboundClient.newInstance(option);

// start client
inboundClient.start();

// shutdown client, after shutdown, can't start again, shutdown with JVM
inboundClient.shutdown();
```

```
// Get InboundClient Instance
InboundClient inboundClient = InboundClient.getInstance();

// Get InboundClient FreeSWITCH API
InboundClient.getBootstrap()
```

```
// add server option
inboundClient.option().addServerOption(new ServerOption(host, port));
// InboundClient.getInstance().option().addServerOption(new ServerOption(host, port));

// remove server option
ServerOption serverOption = inboundClient.option().serverOptions().get(0);
inboundClient.option().removeServerOption(serverOption);
// InboundClient.getInstance().option().removeServerOption(serverOption);

```

## Star history

[![Star History Chart](https://api.star-history.com/svg?repos=zhouhailin/freeswitch-externals&type=Date)](https://star-history.com/#zhouhailin/freeswitch-externals&Date)

## Notice

    1. Send api is recommended to use asynchronous operation, especially the originate command
    2. There is a message timeout mechanism at present to deal with the false death of the connection caused by the direct shutdown of the server. Set the value of readTimeoutSeconds parameter to 0 to turn off this feature
    3. In order to improve performance, when processing EslEvent, use Netty's worker thread. If the processing logic involves IO or time-consuming operation, the processing logic must be put in a new thread to process, and the disablePublicExecutor parameter is set

## WeChat User Groups - Please note the source

![WeChat](https://gitee.com/zhouhailin/images/raw/master/a0eb627b9b6bc1c6da8f19d3292ebce.jpg)

## LICENSE

[Apache License, Version 2.0](https://www.apache.org/licenses/LICENSE-2.0) Copyright (C) Apache Software Foundation
