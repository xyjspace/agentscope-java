<p align="center">
  <img
    src="https://img.alicdn.com/imgextra/i1/O1CN01nTg6w21NqT5qFKH1u_!!6000000001621-55-tps-550-550.svg"
    alt="AgentScope Logo"
    width="200"
  />
</p>

<span align="center">

[**中文**](README_zh.md) | [**Documentation**](https://java.agentscope.io/) | [**Roadmap**](https://github.com/orgs/agentscope-ai/projects/2/views/1)

</span>

<p align="center">
    <a href="https://arxiv.org/abs/2402.14034">
        <img
            src="https://img.shields.io/badge/cs.MA-2402.14034-B31C1C?logo=arxiv&logoColor=B31C1C"
            alt="arxiv"
        />
    </a>
    <a href="https://central.sonatype.com/artifact/io.agentscope/agentscope">
        <img
            src="https://img.shields.io/badge/JDK-17%2B-orange?logo=openjdk"
            alt="JDK 17+"
        />
    </a>
    <a href="https://central.sonatype.com/artifact/io.agentscope/agentscope">
        <img
            src="https://img.shields.io/maven-central/v/io.agentscope/agentscope?color=green"
            alt="Maven Central"
        />
    </a>
    <a href="https://discord.gg/eYMpfnkG8h">
        <img
            src="https://img.shields.io/discord/1194846673529213039?label=Discord&logo=discord"
            alt="discord"
        />
    </a>
    <a href="https://java.agentscope.io/">
        <img
            src="https://img.shields.io/badge/Docs-English%7C%E4%B8%AD%E6%96%87-blue?logo=markdown"
            alt="docs"
        />
    </a>
    <a href="./LICENSE">
        <img
            src="https://img.shields.io/badge/license-Apache--2.0-black"
            alt="license"
        />
    </a>
</p>

## What is AgentScope Java 2.0?

AgentScope Java 2.0 is a production-ready agent framework for building and running LLM-powered applications in Java. It goes beyond a "build an agent" toolkit — providing a complete platform for **running agents in production** with harness engineering, enterprise-grade distributed deployment, and a redesigned core.

We design for increasingly agentic LLMs.
Our approach leverages the models' reasoning and tool use abilities
rather than constraining them with strict prompts and opinionated orchestrations.

## Why use AgentScope Java?

- **Simple**: start building your agents in 5 minutes with `HarnessAgent`, workspace-driven persona, multi-provider model support (DashScope / OpenAI / Anthropic / Gemini / Ollama), built-in tools, permission control, and human-in-the-loop
- **Extensible**: layered memory management, self-evolving skill repository, sub-agents, plan mode, MCP protocol integration, A2A protocol support, middleware system, and IM channel connectors (DingTalk, Feishu, WeCom, GitHub, GitLab)
- **Production-ready**: fully stateless agents for horizontal scaling, multi-tenant isolation, sandbox execution (Docker / K8s / E2B), session recovery, distributed backends (Redis / MySQL / OSS), GraalVM native image support, and built-in OpenTelemetry observability

## News
<!-- BEGIN NEWS -->
- **[2026-06] `RC2`:** AgentScope Java 2.0.0-RC2 released — subagent event forwarding, channel module, `DistributedBackend` unified interface, runtime permission switching. [Release Notes](https://java.agentscope.io/)
- **[2025-05] `RC1`:** AgentScope Java 2.0.0-RC1 released — full architectural upgrade from 1.x. [Docs](https://java.agentscope.io/)
<!-- END NEWS -->

## Community

Welcome to join our community on

| [Discord](https://discord.gg/eYMpfnkG8h)                     | DingTalk | WeChat |
|--------------------------------------------------------------|----------| ---------|
| <img src="./docs/imgs/discord.png" width="100" height="100"> | <img src="./docs/imgs/dingtalk_qr_code.jpg" width="100" height="100"> | <img src="./docs/imgs/wechat.png" width="100" height="100"> |

<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->
## Table of Contents

- [Quickstart](#quickstart)
  - [Installation](#installation)
    - [Maven](#maven)
    - [From source](#from-source)
- [Hello AgentScope!](#hello-agentscope)
- [Streaming](#streaming)
- [Multi-user Concurrency](#multi-user-concurrency)
- [Contributing](#contributing)
- [License](#license)
- [Publications](#publications)
- [Contributors](#contributors)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

## Quickstart

### Installation

> AgentScope Java requires **JDK 17** or higher.

#### Maven

`HarnessAgent` is the recommended entry point — it packages workspace, memory, session persistence, sub-agents, sandboxes, and other engineering capabilities into one builder.

```xml
<dependency>
    <groupId>io.agentscope</groupId>
    <artifactId>agentscope-harness</artifactId>
    <version>2.0.0-RC2</version>
</dependency>
```

If you only need a bare `ReActAgent` (no workspace / persistence / sub-agents / sandbox), depend on `agentscope-core` alone.

#### From source

```bash
# Pull the source code from GitHub
git clone -b main https://github.com/agentscope-ai/agentscope-java.git

# Build and install locally
cd agentscope-java
mvn clean install -DskipTests
```

## Hello AgentScope!

Start your first agent in 5 minutes with AgentScope Java 2.0:

```java
import io.agentscope.core.agent.RuntimeContext;
import io.agentscope.core.message.UserMessage;
import io.agentscope.harness.agent.HarnessAgent;
import io.agentscope.harness.agent.memory.compaction.CompactionConfig;
import java.nio.file.Paths;

public class FirstAgent {
    public static void main(String[] args) {
        HarnessAgent agent = HarnessAgent.builder()
                .name("Friday")
                .sysPrompt("You're a helpful assistant named Friday.")
                // String form resolved via ModelRegistry — picks up DASHSCOPE_API_KEY
                // from the environment. Use "openai:gpt-5.5", "anthropic:claude-sonnet-4-5",
                // "gemini:gemini-2.0-flash", or "ollama:llama3" to switch providers.
                .model("dashscope:qwen-plus")
                .workspace(Paths.get(".agentscope/workspace"))
                .compaction(CompactionConfig.builder()
                        .triggerMessages(30)
                        .keepMessages(10)
                        .build())
                .build();

        RuntimeContext ctx = RuntimeContext.builder()
                .sessionId("demo-session")
                .userId("tony")
                .build();

        // Turn 1
        agent.call(new UserMessage("Hi, Friday!"), ctx).block();

        // Turn 2: same sessionId — state from turn 1 is restored automatically
        agent.call(new UserMessage("What's my name?"), ctx).block();
    }
}
```

## Streaming

Swap `call(...)` for `streamEvents(...)` to receive incremental events — text deltas, tool calls, etc. — suitable for Web / TUI rendering:

```java
import io.agentscope.core.event.AgentEventType;
import io.agentscope.core.event.TextBlockDeltaEvent;
import io.agentscope.core.event.ToolCallStartEvent;

agent.streamEvents(new UserMessage("Summarize today in three bullets."), ctx)
        .doOnNext(event -> {
            switch (event.getType()) {
                case TEXT_BLOCK_DELTA ->
                    System.out.print(((TextBlockDeltaEvent) event).getDelta());
                case TOOL_CALL_START ->
                    System.out.println("\n[tool] " + ((ToolCallStartEvent) event).getToolName());
                default -> { }
            }
        })
        .blockLast();
```

## Multi-user Concurrency

The agent is **stateless between calls** — a single instance can handle requests from different users and sessions concurrently. Pass `userId` / `sessionId` via `RuntimeContext` and the agent automatically loads and isolates the corresponding conversation state:

```java
// Create one agent instance at startup (singleton is fine)
HarnessAgent agent = HarnessAgent.builder()
        .name("Friday")
        .sysPrompt("You're a helpful assistant named Friday.")
        .model("dashscope:qwen-plus")
        .workspace(Paths.get(".agentscope/workspace"))
        .build();

// In your HTTP handler — different requests pass different RuntimeContexts
agent.call(new UserMessage(userInput), RuntimeContext.builder()
        .sessionId(sessionId)
        .userId(userId)
        .build()).block();
```

For full production patterns (Redis session, sandbox, skill repositories), see the [documentation](https://java.agentscope.io/).

## Contributing

We welcome contributions from the community! Please refer to our [CONTRIBUTING.md](./CONTRIBUTING.md) for guidelines
on how to contribute.

## License

AgentScope is released under Apache License 2.0.

## Publications

If you find our work helpful for your research or application, please cite our papers.

- [AgentScope 1.0: A Developer-Centric Framework for Building Agentic Applications](https://arxiv.org/abs/2508.16279)

- [AgentScope: A Flexible yet Robust Multi-Agent Platform](https://arxiv.org/abs/2402.14034)

```
@article{agentscope_v1,
    author  = {Dawei Gao, Zitao Li, Yuexiang Xie, Weirui Kuang, Liuyi Yao, Bingchen Qian, Zhijian Ma, Yue Cui, Haohao Luo, Shen Li, Lu Yi, Yi Yu, Shiqi He, Zhiling Luo, Wenmeng Zhou, Zhicheng Zhang, Xuguang He, Ziqian Chen, Weikai Liao, Farruh Isakulovich Kushnazarov, Yaliang Li, Bolin Ding, Jingren Zhou}
    title   = {AgentScope 1.0: A Developer-Centric Framework for Building Agentic Applications},
    journal = {CoRR},
    volume  = {abs/2508.16279},
    year    = {2025},
}

@article{agentscope,
    author  = {Dawei Gao, Zitao Li, Xuchen Pan, Weirui Kuang, Zhijian Ma, Bingchen Qian, Fei Wei, Wenhao Zhang, Yuexiang Xie, Daoyuan Chen, Liuyi Yao, Hongyi Peng, Zeyu Zhang, Lin Zhu, Chen Cheng, Hongzhu Shi, Yaliang Li, Bolin Ding, Jingren Zhou}
    title   = {AgentScope: A Flexible yet Robust Multi-Agent Platform},
    journal = {CoRR},
    volume  = {abs/2402.14034},
    year    = {2024},
}
```

## Contributors

All thanks to our contributors:

<a href="https://github.com/agentscope-ai/agentscope-java/graphs/contributors">
  <img src="https://contrib.rocks/image?repo=agentscope-ai/agentscope-java&max=999&columns=12&anon=1" />
</a>
