# LLM4S - Large Language Models for Scala

<h4 align="center">
    <a href="https://github.com/llm4s/llm4s/blob/main/LICENSE">
        <img src="https://img.shields.io/badge/License-Apache%202.0-blue.svg" alt ="License">
    </a>
    <a href="https://discord.gg/4uvTPn6qww">
        <img src="https://img.shields.io/static/v1?label=Chat%20on&message=Discord&color=blue&logo=Discord&style=flat-square" alt="Discord">
    </a>
</h4>
<h4 align="center">
    <img src="assets/llm4s_logo.png" width="100pt" alt="LLM4S Logo">
</h4>

## Overview

LLM4S provides a simple, robust, and scalable framework for building LLM applications in Scala. While most LLM work is done in Python, we believe that Scala offers a fundamentally better foundation for building reliable, maintainable AI-powered applications.

<br>

<p align="center">
  <img src="assets/llm4s-overview.jpeg" alt="LLM4S Overview" width="600"/>
  <br>
  <em></em>
</p>

<br>

> **Note:** This is a work in progress project and is likely to change significantly over time.

## Why Scala for LLMs?

- **Type Safety**: Catch errors at compile time rather than runtime
- **Functional Programming**: Immutable data structures and pure functions for more predictable code
- **JVM Ecosystem**: Access to a vast array of libraries and tools
- **Concurrency**: Better handling of concurrent operations with Scala's actor model
- **Performance**: JVM performance with functional programming elegance

## Features

- **Containerized Workspace**: Secure execution environment for LLM-driven operations
- **Workspace Agent Interface**: Standardized protocol for file operations and command execution
- **Multi-Provider Support**: Planned support for multiple LLM providers (OpenAI, Anthropic, etc.)
- **Agent Trace Logging**: Detailed markdown logs of agent execution for debugging and analysis

## Project Structure

- **llm4s**: Main project - contains the core LLM4S framework
- **shared**: Shared code between main project and workspace runner
- **workspaceRunner**: Code that performs the requested actions on the workspace within the docker container
- **samples**: Usage examples demonstrating various features

## Getting Started

To get started with the LLM4S project, check out this teaser talk presented by **Kannupriya Kalra** at the Bay Area Scala Conference. This recording is essential for understanding where we’re headed:

🎥 **Teaser Talk:** https://www.youtube.com/watch?v=SXybj2P3_DE&ab_channel=SalarRahmanian

### Prerequisites

- JDK 21+
- SBT
- Docker (for containerized workspace)

### Building the Project

```bash
# For the default Scala version (3.3.3)
sbt compile

# For all supported Scala versions (2.13 and 3.3)
sbt +compile

# Build and test all versions
sbt buildAll
```

### Setup your LLM Environment

You will need an API key for either OpenAI (https://platform.openai.com/) or Anthropic (https://console.anthropic.com/)
other LLMS may be supported in the future (see the backlog.

Set the environment variables:

```
LLM_MODEL=openai/gpt-4o
OPENAI_API_KEY=<your_openai_api_key>
```

or Anthropic:

```
LLM_MODEL=anthropic/claude-3-7-sonnet-latest
ANTHROPIC_API_KEY=<your_anthropic_api_key>
```

Thia will allow you to run the non-containerized examples.

### Running the Examples

```bash
# Using Scala 3.3.3
sbt "samples/runMain org.llm4s.samples.basic.BasicLLMCallingExample"
```

### Run containerised test

```bash
sbt docker:publishLocal
sbt "samples/runMain org.llm4s.samples.workspace.ContainerisedWorkspaceDemo"

# Using Scala 2.13
sbt ++2.13.14 "samples/runMain org.llm4s.samples.basic.BasicLLMCallingExample"
```

### Cross Compilation

LLM4S supports both Scala 2.13 and Scala 3.3. The codebase is set up to handle version-specific code through source directories:

- `src/main/scala` - Common code for both Scala 2.13 and 3.3
- `src/main/scala-2.13` - Scala 2.13 specific code
- `src/main/scala-3` - Scala 3 specific code

When you need to use version-specific features, place the code in the appropriate directory.

We've added convenient aliases for cross-compilation:

```bash
# Compile for all Scala versions
sbt compileAll

# Test all Scala versions
sbt testAll

# Both compile and test
sbt buildAll

# Publish for all versions
sbt publishAll
```

### Cross-Compilation Testing

We use specialized test projects to verify cross-version compatibility against the published artifacts. These tests ensure that the library works correctly across different Scala versions by testing against actual published JARs rather than local target directories.

```bash
# Run tests for both Scala 2 and 3 against published JARs
sbt testCross

# Full clean, publish, and test verification
sbt fullCrossTest
```

> **Note:** For detailed information about our cross-testing strategy and setup, see [crossTest/README.md](crossTest/README.md)

## Roadmap

Our goal is to implement Scala equivalents of popular Python LLM frameworks:

- [ ] * Single API access to multiple LLM providers (like LiteLLM) - llmconnect
- [ ] A comprehensive toolchain for building LLM apps (like LangChain/LangGraph) 
  - [ ] * RAG search
  - [ ] * tool calling
  - [ ] * logging/tracking/monitoring
- [ ] * An agentic framework (like PydanticAI, CrewAI)
  - [ ] Single agent
  - [ ] Multi-agent
- [ ] Tokenization utilities (port of tiktoken)
- [ ] Examples/ support
   - [ ] * Standard tool calling libraries
   - [ ] * examples of all use-cases
- [ ] stable platform -tests etc
- [ ] Scala Coding SWE Agent - an agent that can do SWE bench type tasks on Scala codebases.
   - [ ]  codemaps 
   - [ ]  generation 
   - [ ]  templates for library use?


## Tool Calling

Tool calling is a critical integration - we aim to make it as simple as possible:

### Tool Signature Generation

Using ScalaMeta to automatically generate tool definitions from Scala methods:

```scala
/** My tool does some funky things with a & b...
 * @param a The first thing
 * @param b The second thing
 */
def myTool(a: Int, b: String): ToolResponse = {
  // Implementation
}
```

ScalaMeta extracts method parameters, types, and documentation to generate OpenAI-compatible tool definitions.

### Tool Call Mapping

Mapping LLM tool call requests to actual method invocations through:

- Code generation
- Reflection-based approaches
- ScalaMeta-based parameter mapping

### Secure Execution

Tools run in a protected Docker container environment to prevent accidental system damage or data leakage.

## Agent Trace Logging

LLM4S provides detailed trace logging capabilities for debugging and analyzing agent executions:

### Using Trace Logs

```scala
// Create an agent with a client
val agent = new Agent(client)

// Run the agent with trace logging
agent.run(
  query = "What's the weather like in London?",
  tools = toolRegistry,
  maxSteps = None,  // optional step limit
  traceLogPath = Some("/path/to/trace-log.md")  // optional trace log path
)
```

The trace log output is formatted as a readable markdown document containing:

- Full conversation history with all messages
- Tool calls and their arguments
- Tool responses
- Agent execution logs
- Current agent status

You can also manually write trace logs at specific points during agent execution:

```scala
// Initialize agent
val state = agent.initialize(query, tools)

// Manually write the state to a trace log
agent.writeTraceLog(state, "/path/to/manual-trace.md")
```

This feature is particularly useful for:

- Debugging complex agent executions
- Understanding tool usage patterns
- Visualizing conversation flow
- Documenting agent behavior for analysis

## 📢 Talks & Presentations

See the talks being given by maintainers and open source developers globally and witness the engagement by developers around the world.

Stay updated with talks, workshops, and presentations about **LLM4S** happening globally. These sessions dive into the architecture, features, and future plans of the project.

### Upcoming & Past Talks

| Date                  | Event/Conference | Talk Title                             | Location                       | Speaker Name     | Details URL                                                                                                                                                        | Recording Link URL                                                                           |
| --------------------- | ---------------- | -------------------------------------- | ------------------------------ | ---------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------ | -------------------------------------------------------------------------------------------- |
| **25-Feb-2025** | Bay Area Scala   | Let's Teach LLMs to Write Great Scala! | Tubi office, San Francisco, CA | Kannupriya Kalra | [Event Info](https://lu.ma/5fz2y9or)                                                                                                                                  | [Watch Recording](https://www.youtube.com/watch?v=SXybj2P3_DE&t=779s&ab_channel=SalarRahmanian) |
| **20-Apr-2025** | Scala India      | Let's Teach LLMs to Write Great Scala! | India                          | Kannupriya Kalra | [Event Info](https://www.linkedin.com/posts/activity-7318299169914249216-Sec-?utm_source=share&utm_medium=member_desktop&rcm=ACoAAA8qk7UBmvcZ2O7aAJfMpsdEXBvcKSNiHWM) | [Watch Recording](https://www.youtube.com/watch?v=PiUaVKuV0dM&ab_channel=ScalaIndia)            |

> 📝 *Want to invite us for a talk or workshop? Reach out via our respective emails or connect on Discord: [https://discord.gg/4uvTPn6qww](https://discord.gg/4uvTPn6qww)*

### Why You Should Contribute to LLM4S?

- Build AI-powered applications in a statically typed, functional language designed for large systems.
- Help shape the Scala ecosystem’s future in the AI/LLM space.
- Learn modern LLM techniques like zero-shot prompting, tool calling, and agentic workflows.
- Collaborate with experienced Scala engineers and open-source contributors.
- Gain real-world experience working with Dockerized environments and multi-LLM providers.
- Contribute to a Google Summer of Code (GSoC)-eligible project.
- Join a global developer community focused on type-safe, maintainable AI systems.

## Contributing

Interested in contributing? Start here:

 **LLM4S GitHub Issues:** https://lnkd.in/eXrhwgWY

## Join the Community

Want to be part of developing this and interact with other developers? Join our Discord community!

 **LLM4S Discord:** https://lnkd.in/eb4ZFdtG

## Google Summer of Code 2025

This project is also participating in **Google Summer of Code (GSoC) 2025**! If you're interested in contributing to the project as a contributor, check out the details here:

👉 **Scala Center GSoC Ideas:** [https://lnkd.in/enXAepQ3](https://lnkd.in/enXAepQ3)

To know everything about GSoC and how it works, check out this talk:

🎥 **GSoC Process Explained:** [https://lnkd.in/e_dM57bZ](https://lnkd.in/e_dM57bZ)

To learn about the experience of GSoC contributors, check out their blogs:


## GSoC 2025: Funded Project Ideas from LLM4S

### LLM4S - Implement an agentic toolkit for Large Language Models

- **Contributor:** Elvan Konukseven
- **LinkedIn:** [https://www.linkedin.com/in/elvan-konukseven/](https://www.linkedin.com/in/elvan-konukseven/)
- **Discord:** elvan_31441
- **Mentor:** [Kannupriya Kalra](https://www.linkedin.com/in/kannupriyakalra/) (Email: [kannupriyakalra@gmail.com](mailto:kannupriyakalra@gmail.com))
- **Co-mentor:** [Rory Graves](https://www.linkedin.com/in/roryjgraves/) (Email: [rory.graves@fieldmark.co.uk](mailto:rory.graves@fieldmark.co.uk))
- **Announcement:** [Official Acceptance Post](https://www.linkedin.com/posts/elvan-konukseven_got-accepted-into-the-google-summer-of-code-activity-7326980179812139008-OoMU?utm_source=share&utm_medium=member_desktop&rcm=ACoAADmu6soBQhs2fe8_CNIw2ChlNe0Oon4E3G0)
- **Contributor Blogs:** [elvankonukseven.com/blog](https://www.elvankonukseven.com/blog)

### LLM4S - RAG in a box

- **Contributor:** Gopi Trinadh Maddikunta
- **LinkedIn:** [https://www.linkedin.com/in/gopitrinadhmaddikunta/](https://www.linkedin.com/in/gopitrinadhmaddikunta/)
- **Discord:** g3nadh_58439
- **Mentor:** [Kannupriya Kalra](https://www.linkedin.com/in/kannupriyakalra/) (Email: [kannupriyakalra@gmail.com](mailto:kannupriyakalra@gmail.com)m)
- **Co-mentor:** [Rory Graves](https://www.linkedin.com/in/roryjgraves/) (Email: [rory.graves@fieldmark.co.uk](mailto:rory.graves@fieldmark.co.uk))
- **Co-mentor:** [Dmitry Mamonov](https://www.linkedin.com/in/dmamonov/) (Email: [dmitry.s.mamonov@gmail.com](mailto:dmitry.s.mamonov@gmail.com))
- **Announcement:** [Official Acceptance Post](https://www.linkedin.com/posts/gopitrinadhmaddikunta_gsoc-googlesummerofcode-scalacenter-activity-7328890778594803714-uP8t?utm_source=share&utm_medium=member_desktop&rcm=ACoAADmu6soBQhs2fe8_CNIw2ChlNe0Oon4E3G0)

### LLM4S - Support image, voice and other LLM modalites

- **Contributor:** Anshuman Awasthi
- **LinkedIn:** [https://www.linkedin.com/in/let-me-try-to-fork-your-responsibilities/](https://www.linkedin.com/in/let-me-try-to-fork-your-responsibilities/)
- **Discord:** anshuman23026
- **Mentor:** [Kannupriya Kalra](https://www.linkedin.com/in/kannupriyakalra/) (Email: [kannupriyakalra@gmail.com](mailto:kannupriyakalra@gmail.com))
- **Co-mentor:** [Rory Graves](https://www.linkedin.com/in/roryjgraves/) (Email:[rory.graves@fieldmark.co.uk](mailto:rory.graves@fieldmark.co.uk))
- **Announcement:** [Official Acceptance Post](https://www.linkedin.com/posts/let-me-try-to-fork-your-responsibilities_big-announcement-im-thrilled-to-activity-7327724651726405635-3Y7V?utm_source=share&utm_medium=member_desktop&rcm=ACoAADmu6soBQhs2fe8_CNIw2ChlNe0Oon4E3G0)

### LLM4S - Tracing support

- **Contributor:** Shubham Vishwakarma
- **LinkedIn:** [https://www.linkedin.com/in/shubham-vish/](https://www.linkedin.com/in/shubham-vish/)
- **Discord:** oxygen4076
- **Mentor:** [Kannupriya Kalra](https://www.linkedin.com/in/kannupriyakalra/) (Email: [kannupriyakalra@gmail.com](mailto:kannupriyakalra@gmail.com))
- **Co-mentor:** [Rory Graves](https://www.linkedin.com/in/roryjgraves/) (Email: [rory.graves@fieldmark.co.uk](mailto:rory.graves@fieldmark.co.uk))
- **Co-mentor:** [Dmitry Mamonov](https://www.linkedin.com/in/dmamonov/) (Email: [dmitry.s.mamonov@gmail.com](mailto:dmitry.s.mamonov@gmail.com))
- **Announcement:** [Official Acceptance Post](https://www.linkedin.com/posts/shubham-vish_gsoc2025-scalacenter-llm4s-activity-7326533865836068864-kQVf?utm_source=share&utm_medium=member_desktop&rcm=ACoAADmu6soBQhs2fe8_CNIw2ChlNe0Oon4E3G0)

Feel free to reach out to the contributors or mentors listed for any guidance or questions related to **GSoC 2026**.

## Maintainers

Want to connect with maintainers? The LLM4S project is maintained by:

- **Rory Graves** - [https://www.linkedin.com/in/roryjgraves/](https://www.linkedin.com/in/roryjgraves/) | Email: [rory.graves@fieldmark.co.uk](mailto:rory.graves@fieldmark.co.uk) | Discord: `rorybot1`
- **Kannupriya Kalra** - [https://www.linkedin.com/in/kannupriyakalra/](https://www.linkedin.com/in/kannupriyakalra/) | Email: [kannupriyakalra@gmail.com](mailto:kannupriyakalra@gmail.com) | Discord: `kannupriyakalra_46520`

## License

This project is licensed under the Apache License 2.0 - see the [LICENSE](LICENSE) file for details.
