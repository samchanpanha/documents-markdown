# Principle and Application Practice of Agent MCP Protocol

**Original** **Fan Gui Hurricane** [AI Cyberspace](javascript:void(0);)

 *October 15, 2025, 10:40 AM*

# MCP v.s. Function Calling

**The mainstream before MCP was Function Calling, but the latter had two key problems:**

1. **System integration standardization requirements ** **: The ability to invoke external systems （data, tools） is a key feature of the Agent。However, when the external system does not have standards, the code maintains a lot of Tools and the complexity of system integration can be very high. Such asOpenAI Function Calling API，It is only suitable for specific LLM and platforms, and each application needs to develop the interaction logic separately, which is difficult to reuse and expand.**
2. **Context transmission optimization requirements ** **: Context is the transmission object between Agent and LLM , but Agent and LLM often need to maintain complex Context Updating code logic during the dialogue process , making it difficult to efficiently transmit complex context between LLM and external systems .**

**To this end, Anthropic launched MCP (Model Context Protocol) in November 2024 as an Agent & LLM-centric open standard communication protocol designed to simplify the integration architecture with external systems and optimize the processing logic for contextual transport.**

* **官网：https://modelcontextprotocol.io/docs/getting-started/intro**

**Difference between MCP and Function Calling:**![Please add a picture description](https://mmbiz.qpic.cn/mmbiz_jpg/t1f5IicysA2wJqbic48J5SdsNR2PhYV3za4xu1UBYrjVAwvVIdaTVZZqBibXicXCdDnxZr8pHqZuH5wC4Iys2VNxzQ/640?wx_fmt=other&from=appmsg)

**While MCP is similar in some ways to RESTful APIs and Function Calling, there are significant differences between them.**![Insert picture description here](https://mmbiz.qpic.cn/mmbiz_jpg/t1f5IicysA2wJqbic48J5SdsNR2PhYV3zao6DSaRp7lQGpzdPnlYleApfW87KVE26JCxickPtQhufEL5duMYpETdA/640?wx_fmt=jpeg&from=appmsg)

# MCP tethered architecture

**MCP is a standardized tethering architecture that has the following benefits at the tethering integration level:**

1. **High scalability and flexibility ** **: MCP supports easy integration of new tools and services to meet changing business needs without modifying existing workflows .**
2. **Automatic discovery mechanism of tools and services ** **: After registering MCP, LLM can automatically discover and integrate related tools and services。No longer need to manually set up, improve efficiency and convenience.**
3. **Reduced development complexity ** **: By abstracting the interaction logic , reducing the complexity of agent development and enabling developers to focus on enhancing core agent functionality .**
4. **Enhanced security ** **: use of mature security transmission layer protocol to ensure the security and reliability of data and interaction .**
5. **Promote Collective Intelligence ** **: Distributed Agentic AI systems can achieve results that a single architecture cannot by sharing insights and coordinating actions through standardized communication channels .**

## A layered architecture

**As shown below, the MCP sits between the LLM and the Agent.**![Insert picture description here](https://mmbiz.qpic.cn/mmbiz_jpg/t1f5IicysA2wJqbic48J5SdsNR2PhYV3zaoAiaKFFO1TFPvc4lc1nXAADCWxh3bIpnQ8xGJWboCAYwKa7j8OupicfQ/640?wx_fmt=jpeg&from=appmsg)

## Integrated architecture

![Please add a picture description](https://mmbiz.qpic.cn/mmbiz_gif/t1f5IicysA2wJqbic48J5SdsNR2PhYV3zad6NtgzCoTE5Hr73Vl7UvlaEGN7CQPY9ickEvQv3UA4U8WdQK25nZwoQ/640?wx_fmt=gif&from=appmsg)
Please add a picture description

**The integrated architecture of MCP system consists of three core components: MCP Host, MCP Client and MCP Server.**

### MCP Host

**Definition: MCP Host is the runtime environment for agents that want to access external systems via the MCP protocol.**

**Features:**

* **Provide the user interface, provide the service of the agent.**
* **The configuration file config.json is used to set up the MCP components that need to be accessed.**
* **Responsible for starting the MCP Client and using the functions of the MCP Server through the MCP Client.**
* **Provide margins of safety, user authorization, and access rights of Server to Host.**

### MCP Client

**Definition: Usually as a function module of Agent, MCP Client maintains a 1: 1 protocol connection with MCP Server, and is the bridge between Agent & LLM and MCP Server.**

**Features:**

* **Connection Management: Responsible for establishing, maintaining and closing connections with the MCP Server, handling various events in the life cycle of the connection, such as initial initiation, heartbeat detection and exception handling.**
* **Request forwarding: Forwards a request from Agent & LLM to the MCP Server and accepts the response.**
* **Error handling: handles various errors that may occur during connection and communication and provides appropriate error information to the up-level application.**
* **Feature discovery: Responsible for querying the capabilities provided by the MCP Server, including available Resources, Tools, and Prompts.**

**In the MCP architecture, the host and client work closely together. Host provides Agent running conditions and margins of safety, while Client is responsible for protocol implementation and communication management. The layered design of MCP makes it both flexible and secure, capable of adapting to various AI application scenarios.**

![Insert picture description here](https://mmbiz.qpic.cn/mmbiz_jpg/t1f5IicysA2wJqbic48J5SdsNR2PhYV3zaB0u5kT2MXI0Q9xMTciczVlsSWY1uMFfaIUStuez2XuOEslTPjNuU0jg/640?wx_fmt=jpeg&from=appmsg)
Insert picture description here

### MCP Server

**Definition: A standalone, lightweight server program that acts as a MCP Client and also as a front-end program for external systems, providing LLM with the ability to access data, perform tools, and call services. Is a bridge between the MCP Client and the external system. Typically, each MCP server will focus on a specific functional area, such as document system operation, data library access, API integration, etc.**

**Features:**

* **Abstract: Abstract the ability of external system by means of Resources, Tools, Prompts.**
* **Function Release: Responsible for establishing a connection with the MCP Client and publishing these functions to the MCP Client through the MCP protocol.**
* **Request forwarding: The left side accepts the MCP Client's request, and the right side translates and forwards the request to external systems, such as: API, DB, File, SSE, Internet, etc.**

### Local / Remote Resources

**The MCP also provides two core resources:**

1. **Local Resources ** **：Running in a MCP Host local environment, MCP Server securely accesses local resources, such as local file systems, databases, applications, etc., through a local network or protocol.Sensitive data is prioritized for local execution to protect data privacy and security.**
2. **Remote Resources ** **：MCP Server is a remote service that runs in the external environment of the Internet and is accessed securely through Web APIs, such as cloud storage, remote APIs, online services, etc.**

![Insert picture description here](https://mmbiz.qpic.cn/mmbiz_png/t1f5IicysA2wJqbic48J5SdsNR2PhYV3zal5ibcnzC1AhYBqSjMKqsEKASPs892yiaednHbQtQTDmicpXb53Tncn5Vw/640?wx_fmt=png&from=appmsg)
Insert picture description here

# MCP Communication Protocol

**MCP Communication Protocol defines the transmission protocol, communication model, message format and communication rules between client and server.**

**MCP is a stateful, context-preserving communication protocol that has the following characteristics at the communication level:**

1. **Real-time two-way communication ** **: Different from the traditional REQ-RESP RESTful API model , MCP supports real-time , two-way , dynamic sending and receiving message capabilities , and realizes instant communication and flow of information .**
2. **Excellent context-awareness ** **: MCP provides standardized Context container fields to maintain session-specific long-term memory during Agent and LLM interaction , ensuring the coherence and continuity of information , enabling LLM to generate more ideal responses .**
3. **Context transfer performance ** **: Traditional JSON format input of 10K tokens to LLM takes 500ms+, while MCP compressed input takes only 20ms。And local updates （such as new dialogue） need to retransmit the full context to LLM, while MCP supports incremental updates.**

## Transport Layer (Transport Layer)

**The Transport Layer of MCP adopts the JSON - RPC 2.0 structured communication protocol, which supports a variety of data transmission methods, and requires both stateful and stateless requests.**

1. **Stdio (standard input output)**
2. **HTTP over SSE (Server Push Event Stream)**
3. **Streamable HTTP**![Insert picture description here](https://mmbiz.qpic.cn/mmbiz_png/t1f5IicysA2wJqbic48J5SdsNR2PhYV3zaiawLG8AJ0XmPAicTQ0CniboyhwtgYZp7eHQ34deJZYf9vZwOgwf1OCNVQ/640?wx_fmt=png&from=appmsg)

### Stdio

**Stdio is suitable for local integration, especially for communication on the same machine. The client and server transfer data through the stdin, stdout, stderr pipes of the OS / Kernel.**

![Insert picture description here](https://mmbiz.qpic.cn/mmbiz_png/t1f5IicysA2wJqbic48J5SdsNR2PhYV3zaJhaUwEdDBibSw8wqWlTD06cw0GHZTAhUJqjslu1BWBG0oAcC08qXN3Q/640?wx_fmt=png&from=appmsg)
Insert picture description here

### HTTP over SSE (replaced)

**HTTP over SSE is suitable for network-based telecommunications integration for real-time updates and persistent connections.**

**The server starts two endpoints:**

1. **SSE Endpoint: S2C (Server to Guest) message push using Server-Sent Events;**
2. **HTTP Endpoint: Using HTTP POST to implement C2S (Guest to Server) messaging.**

**When the client and the server interact, the connection is first established through the SSE endpoint, then the server pushes an HTTP endpoint to the client, and finally the client sends a request to the server through the HTTP endpoint.**

![Insert picture description here](https://mmbiz.qpic.cn/mmbiz_png/t1f5IicysA2wJqbic48J5SdsNR2PhYV3za8XCEb1FxckXekIVgMq7HfdTrUCZxnVKb32yRG8yBRvNwwQV4WCp6zA/640?wx_fmt=png&from=appmsg)HTTP over SSE has obvious disadvantages:

1. **It requires long connections and consumes resources.**
2. **A strong reliance on session stickiness is not conducive to horizontal scaling.**
3. **After disconnecting, the connection must be re-established.**

**So it has been replaced by Streamable HTTP since 2025 / 03 / 26.**

### Streamable HTTP（新引入）

**Streamable HTTP's MCP Server can handle multiple client connections.**

1. **The MCP Server provides an HTTP Endpoint and supports both GET and POST requests.**
2. **The message sent by the MCP Client needs to include the Accept header, which supports the application / json and text / event-stream types.**
3. **The MCP Client opens an SSE stream with a GET request, where the Accept header is of type text / event-stream.**
4. **If you want to use stateful communication, you need to use the Mcp-Session-Id header for transmission.**

## Protocol Layer (Protocol Layer)

### Connected Lifecycle Management

**The MCP connection lifecycle management manages the interaction state and transformation between MCP Client and MCP Server, which ensures the robustness of communication and the reliability of functions in the whole interaction process. Including: Connection initialization, Session management, version and capacity negotiation, user authorization and authentication, user rights control.**

![Insert picture description here](https://mmbiz.qpic.cn/mmbiz_png/t1f5IicysA2wJqbic48J5SdsNR2PhYV3zahEwohFlYhkZKJRVw5hPttvHk8okPk63ku4l8fzJCNicmdgbzhTfuHmg/640?wx_fmt=png&from=appmsg)
Insert picture description here

**The initialization mechanism of MCP connections ensures that the MCP protocol remains compatible and extensible even between different versions or implementations.**

1. **initial request （ initial request ） ** **: sends the initial request to the server immediately after starting the client , which contains the version protocol version field and capabilities capability negotiation field .**

```
{
  "jsonrpc": "2.0",
  "id": 1,
  "method": "initialize",
  "params": {
    "protocolVersion": "0.3.0",
    "clientInfo": {
      "name": "example-client",
      "version": "1.0.0"
    },
    "capabilities": {
      "resources": {},
      "tools": {},
      "prompts": {}
    }
  }
}
```

2. **initial response （initial response） ** **: The server uses its own protocol version and capabilities to respond。Start a capability negotiation and declare the capabilities supported by the Server, including: prompts, tools, resources, etc. Only the capabilities that are successfully negotiated can be used in Session.The successful response is as follows, but if the version is incompatible or the request is invalid, the server returns an error response.**

```
{
  "jsonrpc": "2.0",
  "id": 1,
  "result": {
    "protocolVersion": "0.3.0",
    "serverInfo": {
      "name": "example-server",
      "version": "1.0.0"
    },
    "capabilities": {
      "resources": {
        "list": {},
        "read": {}
      },
      "tools": {
        "list": {},
        "call": {}
      }
    }
  }
}
```

3. **notification （ notification ） ** **: After the client accepts the response to complete the negotiation , it will send another initialized notification to confirm the successful establishment of the connection .**

```
{
  "jsonrpc": "2.0",
  "method": "notifications/initialized",
  "params": {}
}
```

4. **message exchange ** **: After the connection is ready, a Session is established between the Client and Server to maintain Connection Status, Context, and other Metadata。And normal message exchange can begin.**

**After the connection is established, the client proactively requests the server a list of tools / resources currently available and their descriptions. This information is injected into the LLM by the agent / client along with the UserQuery if necessary.**

![Insert picture description here](https://mmbiz.qpic.cn/mmbiz_jpg/t1f5IicysA2wJqbic48J5SdsNR2PhYV3zaD05HgFKfoGCkNAbO4MBV5vReCegJjl4IkibGv2fnEDKhBibAzZbdC2AQ/640?wx_fmt=jpeg&from=appmsg)
Insert picture description here

**The termination of the connection can be achieved in the following ways:**

1. **Elegant shutdown （Clean Shutdown） ** **：Client Or the Server implements this by calling the close () method.Both sides have a chance to complete the pending request and clean up resources.**

```
{
  "jsonrpc": "2.0",
  "id": 999,
  "method": "shutdown",
  "params": {
    "reason": "user_request"
  }
}
```

2. **Forced Termination （Forced Termination） ** **：Unexpected shutdown due to errors, timeouts, or other exceptional conditions.**

* **Transmission dropped ** **: This occurs when the communication channel is lost .**
* **Error condition ** **: If an error occurs , it may also cause the connection to terminate .**

![Insert picture description here](https://mmbiz.qpic.cn/mmbiz_jpg/t1f5IicysA2wJqbic48J5SdsNR2PhYV3zaeGSPxMZ9UKYsria78e21NicbdaH8GrDA6YPC9yPtYBIVAZg8V2Ncocdg/640?wx_fmt=jpeg&from=appmsg)
Insert picture description here

### Communication Modes and Message Body Encapsulation

**After the initial phase, MCP supports the following two communication modes:**

1. **The two-way request-response （ Request-Response ） pattern ** **: either the client or the server sends a request and the other party responds .**
2. **One-way notification （Notification） pattern ** **: Any party can send a one-way message without the need for a response from the other party。Used in scenarios such as status updates, event notifications, etc.For example, in a long-running operation, the sender can be implemented by a special progress notification msg.**

**There are three types of messages exchanged in the Request-Response communication pattern:**

* **Request message body ** **: Sends a request with id , method and params fields .**

```
{ "jsonrpc": "2.0", "id": 1, "method": "resources/list", "params": {} }
```

* **Result message body ** **: The successful response to the request。With id, and req-id one-to-one correspondence.**

```
{ "jsonrpc": "2.0", "id": 1, "result": {} }
```

* **Error message body ** **: The response for a failed request, with Code and Message。With id, and req-id one-to-one correspondence.**

```
{ "jsonrpc": "2.0", "id": 1, "error": { "code": -32601, "message": "Method not found" } }
```

**The MCP defines a standard set of Error Codes to effectively manage possible problems. In addition, the agent can customize the error code, starting with -32000.**

```
enum ErrorCode {
  // Standard JSON-RPC error codes
  ParseError = -32700,
  InvalidRequest = -32600,
  MethodNotFound = -32601,
  InvalidParams = -32602,
  InternalError = -32603
}
```

**There is only one type of message exchanged in the notification communication mode:**

* **The notification message body ** **: one-way notification , no response required .**

```
{ "jsonrpc": "2.0", "method": "resources/updated", "params": {} }
```

![Insert picture description here](https://mmbiz.qpic.cn/mmbiz_jpg/t1f5IicysA2wJqbic48J5SdsNR2PhYV3zacvJnIt3r5mU1icHeNchkrAnOgXuvR6OrByvUMeJfuev5TtRgOGqwibNg/640?wx_fmt=jpeg&from=appmsg)
Insert picture description here

## Feature Layer (Feature Layer)

**The Feature Layer of the MCP is responsible for the business services support of Agent & LLM, including the following capabilities.**

* **MCP Client has Sampling, Roots, and so on.**
* **MCP Server has three core capabilities: Prompts, Resources and Tools.**

**These capabilities are encapsulated and transmitted through the two communication modes and message types described above.**![Insert picture description here](https://mmbiz.qpic.cn/mmbiz_jpg/t1f5IicysA2wJqbic48J5SdsNR2PhYV3za5BoIEb57yc8vXYXdQcTebGq4Yo1jfprGtvHFagjGSCgqQovPI9typQ/640?wx_fmt=jpeg&from=appmsg)

### MCPClient Capabilities

![Insert picture description here](https://mmbiz.qpic.cn/mmbiz_jpg/t1f5IicysA2wJqbic48J5SdsNR2PhYV3zaqqkMiaXy6rRzA023KREslic2OQjshupTYiavHJb8yMoBl6YVxRyTpdSqw/640?wx_fmt=jpeg&from=appmsg)
Insert picture description here

#### Sampling (sampling)

**The server can request the client to access the LLM for sampling:**

* **The server requests the client to access the LLM. The client then gets the result from the LLM and returns it to the server.**
* **The client can check and modify the request. This human-in-the-loop design ensures that the user retains control of the interaction between the server and the LLM.**

**Related operation request for sampling:**

* **Sampling / createMessage: Request LLM to generate content.**

![Insert picture description here](https://mmbiz.qpic.cn/mmbiz_jpg/t1f5IicysA2wJqbic48J5SdsNR2PhYV3zaPMd7OMxdFhFDFkunHSSyQ2EYdWVzfsjCX4JKxHgKgjBVKCvVvicy30g/640?wx_fmt=jpeg&from=appmsg)
Insert picture description here

#### Roots (root directory)

**Defines the boundaries of the server operable host document system, with a specific secure access directory as the root directory.**

* **The Server accesses the Client to obtain Roots, which are primarily used for the literal system path, but can also be any valid URI.**
* **The server uses this to know the scope of resource access, but only as a guide, not as a mandate.**

![Insert picture description here](https://mmbiz.qpic.cn/mmbiz_png/t1f5IicysA2wJqbic48J5SdsNR2PhYV3zaS7jHL6Q4NbJIcs2rpD5Mx1rjdhJal47GsMHKZ4MxicPG00daibre7suw/640?wx_fmt=png&from=appmsg)
Insert picture description here

### The capabilities of MCP Server

**MCP abstracts all kinds of data sources, tools and services into Resources, Tools and Prompts, standardizes the way of calling, and reduces the complexity of integration. The code implementation of the MCP Server is usually based on an SDK provided by an external system.**

![Insert picture description here](https://mmbiz.qpic.cn/mmbiz_png/t1f5IicysA2wJqbic48J5SdsNR2PhYV3za5ZVu0ia3FAGTeSKpX66eDNHKXzo9ibicfG8Mnz59186iajibFicmQw3DDMiag/640?wx_fmt=png&from=appmsg)
Insert picture description here

#### Resources

**Resources are the data resources that MCP Server can provide, such as: file content, digital library records, API responses and other data resources.**

* **Each resource has attributes such as URI, name, description, and MIME type;**
* **Unique identification using a URI;**
* **Clients can discover and access resources through resources / list and resources / readEndpoints.**

**Resource-related request type:**

* **resources/list ** **：Lists the available resources,**
* **resources/read ** **：Read resource content**
* **resources/subscribe ** **：订阅资源更新**
* **resources/unsubscribe ** **：取消订阅资源更新**

**There are two basic resource types:**

1. **Text resources ** **: Contain text data encoded in UTF-8。Ensure that the file is encoded in UTF-8 to avoid Chinese garbled.**
2. **Binary resource ** **: Contains raw binary data encoded in base64 .**

![Insert picture description here](https://mmbiz.qpic.cn/mmbiz_jpg/t1f5IicysA2wJqbic48J5SdsNR2PhYV3zaxug0arbibXnWK9ubqEJnglUPYPKfqw7wC6AkTRZC2gRh0zYq8ytnzsA/640?wx_fmt=jpeg&from=appmsg)
Insert picture description here

**The URI (Uniform Resource Identifier) of a resource is usually defined by the MCP Server and follows the following format:**

```
[protocol]://[host]/[path]
```

**For example:**

```
# 本地文件
file:///home/user/documents/report.pdf
# 数据库模式
postgres://database/customers/schema
# 屏幕内容
screen://localhost/display1
# 来自 Web API 的数据
http://api.example.com/data
# Git 仓库中的源代码
git://repository/main/src
```

**A URI can also contain query parameters that further refine access to a resource, such as:**

```
weather://forecast?city=beijing&days=5
```

**The client can obtain the static resource list of the server directly from resource / list, and also allows the client to define dynamic resources from the URI template. For example:**

```
# 获取特定城市的天气预报
weather://{city}/forecast
# 访问特定用户资料
user://{id}/profile
# 获取特定数据库表记录
database://{table}/{id}
```

**The {} variable in the URI template can be replaced by a Client with a concrete value to create a valid URI.This allows the client to dynamically build resource requests without knowing all possible resource URIs in advance.**

#### Tools

**The MCP Server can provide a collection of tools that can be found and called by LLM. The essence of a tool is an executable function, including parameter validation, result processing, error handling and other logic.**

* **Unique name, description, and JSON Schema-based input parameter definitions.**
* **The client can discover and invoke the Tool through tools / list and tools / callEndpoint.**
* **The execution of the tool requires user confirmation to ensure security.**

**The execution of the tools is a tightly controlled multi-step process that provides a Human-in-loop (human intervention), reliable operation, and observability. In the tool execution process, humans play a key oversight role and are an important safety mechanism.**

* **Approve or reject proposed tool calls.**
* **Review and potentially modify the call parameters.**
* **Evaluate the results of the tool and the next steps.**
* **Provide additional context or correct misunderstandings.**

**The MCP can also provide different levels of approval requirements depending on the nature of the tool or user preference, such as:**

* **Approval is required for each call.**
* **Approval is required only for specific tools or parameters.**
* **A similar operation was approved in bulk.**
* **Provide automated approval for low-risk operations.**

![Insert picture description here](https://mmbiz.qpic.cn/mmbiz_jpg/t1f5IicysA2wJqbic48J5SdsNR2PhYV3za4vZYyQ7BmXHjC7xsNE2mSic9jbbdrW1NhPSLWuNqvx0HhfzracaicREA/640?wx_fmt=jpeg&from=appmsg)
Insert picture description here

**Tools Related request type:**

* **tools/list ** **：List the available tools.**
* **tools/call ** **：Calls the tool and performs an action.**

![Insert picture description here](https://mmbiz.qpic.cn/mmbiz_png/t1f5IicysA2wJqbic48J5SdsNR2PhYV3za1szaGSZqfcMP9EZNovNvfUzV25Cuvw81PSibrC7QkVjyMjW4pP5bYOg/640?wx_fmt=png&from=appmsg)
Insert picture description here

1. **Client gets a list of tools.**

```
{
  "jsonrpc": "2.0",
  "id": 1,
  "method": "tools/list",
  "params": {
    "cursor": "optional-cursor-value"
  }
}
```

2. **Client invocation tool.**

```
{  "jsonrpc": "2.0",  
   "id": 2,  
   "method": "tools/call",  
   "params": {
     "name": "get_weather",
       "arguments": {
         "location": "New York"
       }
    }  
}
```

3. **Server sends a change notification.**

```
{  
   "jsonrpc": "2.0",  
   "method": "notifications/tools/list_changed"  
}
```

#### Prompts

**Prompts are templates of prompts provided by MCP Server for Resources, Tools, or specific use cases. They support parameterized rendering and reuse, and help LLM generate specific types of responses, such as logical chains that can be designed in multiple steps to guide users through complex tasks. It ensures the effective and stable operation of MCP Server.**

* **Support dynamic parametric rendering**
* **Supports embedding resource contexts.**
* **Supports multiple interactive sessions.**
* **The client discovers and uses Prompt through prompts / list and prompts / getEndpoint.**

**Prompt Related request action:**

* **prompts/list ** **：Lists the available prompt templates.**
* **prompts/get ** **：Gets the content of the prompt template.**

![Insert picture description here](https://mmbiz.qpic.cn/mmbiz_jpg/t1f5IicysA2wJqbic48J5SdsNR2PhYV3zajg2tOlKzpOqOq6ll56zUnxIpQ3q2RNH0VP2dV23XvNQAeAibpJDsNicw/640?wx_fmt=jpeg&from=appmsg)
Insert picture description here

# MCP context window

**The MCP has a dynamic Context Window for storing session records (ask / answer) as well as environment data that expands with each interaction.**

**At the same time, in order to avoid data overload, MCP will retain key details while compressing non-key information into embedding form. For example, summarize the chat content of 10 messages as an intent vector.**

![Insert picture description here](https://mmbiz.qpic.cn/mmbiz_png/t1f5IicysA2wJqbic48J5SdsNR2PhYV3zavbU4ophQYnt2bQbmkcs6hChINZxnFCKruzibTMPb4cwx08HSQx41Axg/640?wx_fmt=png&from=appmsg)
Insert picture description here

# MCP Security and Trust Mechanism

**One of the core principles of MCP is "secure and controllable," and the expansion of LLM capabilities should not be at the expense of user control and digital security.**

![Insert picture description here](https://mmbiz.qpic.cn/mmbiz_jpg/t1f5IicysA2wJqbic48J5SdsNR2PhYV3zawp1XTLZMkz4zagM6iaP8CuicSlzpG3uCgoHHISW6ewzoyYotia2aiboicuQ/640?wx_fmt=jpeg&from=appmsg)
Insert picture description here

**As shown in the figure above. At the design level, MCP adopts a multi-layer security model:**

1. **Local Priority ** **: The server is run on the local host by default, reducing network risks。At the same time, the host must obtain the user's consent before exposing the data.**
2. **User acknowledges ** **: all data access and tool calls require explicit authorization from the user .**
3. **Tool Security ** **: Tool invocation must be confirmed by the user . Description information should be treated with caution .**
4. **Sampling control ** **: Sampling LLM requires user approval and Server cannot read all context directly .**
5. **Resource Boundaries ** **: Limit the scope of resource access by roots .**
6. **Command Privileges ** **: The server runs with user account privileges and access is restricted .**
7. **Transmission security ** **: support encryption transmission and authentication mechanism .**

**At the implementation level, the MCP recommends:**

* **TLS secure transport protocol is adopted.**
* **OAuth 2.0 for body verification.**
* **RBAC is used for permission control.**

![Insert picture description here](https://mmbiz.qpic.cn/mmbiz_png/t1f5IicysA2wJqbic48J5SdsNR2PhYV3zayrHBcfunZGSPiappmcfAFBAxK0ZrXoTDdr8Rose2z0BtwQHRKLYXamA/640?wx_fmt=png&from=appmsg)
Insert picture description here

# MCP workflow

![Please add a picture description](https://mmbiz.qpic.cn/mmbiz_gif/t1f5IicysA2wJqbic48J5SdsNR2PhYV3za9xUcZFVqls1AejBKYiceC0ZSGvJY24oiagIaMBYy9zFzRDlLYhQT94VA/640?wx_fmt=gif&from=appmsg)![Insert picture description here](https://mmbiz.qpic.cn/mmbiz_png/t1f5IicysA2wJqbic48J5SdsNR2PhYV3zasw8saMsfEicG6yCWiaQ47KMRWEAoYMRJKpib0OROMGVY2Pk2uU1vonib4w/640?wx_fmt=png&from=appmsg)An example of a specific MCP workflow is as follows:

1. **Use the LLM app to summarize the last 5 commits in the code base. The MCP Host with Client first calls the MCP Server and asks what Tools List is available.**![Insert picture description here](https://mmbiz.qpic.cn/mmbiz_png/t1f5IicysA2wJqbic48J5SdsNR2PhYV3za5ZpXUCG2vJlibnI6OnXE5BfE0Ru4pXwD3L8sibdJiaeJpOUvTblyIibOhA/640?wx_fmt=png&from=appmsg)
2. **The MCP Host then enters these Tools into the LLM. When the LLM receives this information, it chooses to use a tool. The MCP Host then sends a specific tool request to the MCP Server, and receives the tool execution results.**![Insert picture description here](https://mmbiz.qpic.cn/mmbiz_png/t1f5IicysA2wJqbic48J5SdsNR2PhYV3zadAxHjI6YYnpOL5VmZMCaxicQC5fro2YLSCIecKTgSx1s2oicBPzPerhg/640?wx_fmt=png&from=appmsg)
3. **Finally, the MCP Host enters the results of the tool execution into the LLM. LLM returns the final response to the user.**![Insert picture description here](https://mmbiz.qpic.cn/mmbiz_png/t1f5IicysA2wJqbic48J5SdsNR2PhYV3za2TKddEl37voDEv1KWvDUYr5I8IuTOJMLKxL9m2ro64ZAmqT14b0fAA/640?wx_fmt=png&from=appmsg)

# MCP Application Practice

## Claude Desktop MCP

**The most typical example of the MCP experience is the Anthropic Claude Desktop.**![Insert picture description here](https://mmbiz.qpic.cn/mmbiz_png/t1f5IicysA2wJqbic48J5SdsNR2PhYV3za8WfcKjYEWtXiaZPSt8PHicgVlWW7UgpzPtT0Pial5icNcric3sicbLqPDDCQ/640?wx_fmt=png&from=appmsg)

**Environmental information:**

* **macOS**
* **Claude Desktop**
* **MCP Server：FileSystem-mcp-server**
* **Authority requirements: administrator rights, local file system access right, network access right**

### Installing Claude Desktop

**Claude Desktop as MCP Host and MCP Client.**

* **Download the download: https://claude.ai/download**

**Configure your individual needs.**![Insert picture description here](https://mmbiz.qpic.cn/mmbiz_jpg/t1f5IicysA2wJqbic48J5SdsNR2PhYV3zaynhEvfLXbBlHKic64CaKLmAqIt3RBrUFGTMpmPAvpvs0KpwW3ol95jQ/640?wx_fmt=jpeg&from=appmsg)

### MCP Server Configuration File Parsing

**Claude _ desktop _ config.json is the configuration file for Claude Desktop.**

```
$ ll ~/Library/Application\ Support/Claude/claude_desktop_config.json
```

![Insert picture description here](https://mmbiz.qpic.cn/mmbiz_jpg/t1f5IicysA2wJqbic48J5SdsNR2PhYV3zaV3rgM80bre9jw3Eff2tZwUiaNYXUYJsGNdv43uoLqw0OmIicJlnqiaLFA/640?wx_fmt=jpeg&from=appmsg)
Insert picture description here

**Where the name of the MCP Server configuration is mcpServers in the following format:**

* **command ** **：Command used to start MCP Server.For MCP Server running locally, the startup command is usually npx （Node.js app） or uvx （Python app）.**
* **args ** **：The startup parameters passed to the command, depending on the MCP Server.**

```
{
  "mcpServers": {
    "server1": {
      "command": "命令名称",
      "args": ["参数1", "参数2", ...]
    },
    "server2": {
      "command": "命令名称",
      "args": ["参数1", "参数2", ...]
    },
    ...
  }
}
```

**NOTE：**

1. **After each modification of the configuration file, you need to restart the Claude Desktop app to take effect.**
2. **Make sure the JSON format is correct, otherwise the configuration will not be recognized by Claude Desktop.**

### 使用 FileSystem MCP Server

**FileSystem MCP Server is one of the most used official extensions for Claude Desktop and supports the following features:**

1. **Read _ file: Reads the contents of the specified file.**
2. **Write _ file: Create or modify the contents of a file.**
3. **File _ info: Gets the metadata of the file, such as size, modification time, etc.**
4. **List _ directory: Displays files and subdirectories in a directory.**
5. **Make _ directory: Create a new directory.**
6. **Remove: Removes a file or directory.**
7. **etc.**

**It can be installed and configured directly through the Desktop UI.**![Insert picture description here](https://mmbiz.qpic.cn/mmbiz_jpg/t1f5IicysA2wJqbic48J5SdsNR2PhYV3zaD3rWC8AicNzXjK9j4rlQibiaNMcb9OwczxZwBvLXsZjTac5OnZFY8suKA/640?wx_fmt=jpeg&from=appmsg)

**Configuration: MCP Server configuration:**

* **-y ** **：Automatically confirm all prompts and skip the confirmation step.**
* **@modelcontextprotocol/server-filesystem ** **: Specifies that the package used by the Filesystem MCP Server is the official package ( https://github.com/modelcontextprotocol/servers/tree/main/src/filesystem ) You can also specify a package from another provider.**
* **Last parameter: Specify the directory path that is allowed to access.**

```
$ vim ~/Library/Application\ Support/Claude/claude_desktop_config.json


{
  "mcpServers": {
    "filesystem": {
      "command": "npx",
      "args": [
        "-y",
        "@modelcontextprotocol/server-filesystem",
        "/Users/fanguiju/Documents"
      ]
    }
  }
}
```

**Restart the Desktop to load the MCP Server, and you can see the tag:**![Insert picture description here](https://mmbiz.qpic.cn/mmbiz_jpg/t1f5IicysA2wJqbic48J5SdsNR2PhYV3zaicygIJzqQlz68PDnRUT6ia7dgPg9ic8Qq3sGPbJpARzsGibxxpQHfhh5Og/640?wx_fmt=jpeg&from=appmsg)

**During use, the FileSystem MCP Server accesses MacOS's document system, and it always asks for human confirmation when executing the Tools. This is in line with the MCP security concept.**![Insert picture description here](https://mmbiz.qpic.cn/mmbiz_jpg/t1f5IicysA2wJqbic48J5SdsNR2PhYV3zaMmd8dKQk7T316g5fZTDNtovwHyO5iakEiaRAZoRVToDWQQJm5j8O7jsg/640?wx_fmt=jpeg&from=appmsg)

## MCP Registries

**There are several open source libraries or websites that provide hosted MCP tool resources to enhance the ability of LLMs and agents to generate responses reliably. These hosted MCP libraries are called registries. Authors of MCP tools can register their own code into these registries, along with detailed integration documentation. For example:**

* **https://github.com/modelcontextprotocol/servers**
* **https://www.mcpworld.com/**
* **https://mcp.so/**
* **https://glama.ai/mcp/servers/@KBB99/mcp-registry-server**
* **https://smithery.ai/**
* **https://www.pulsemcp.com/**
* **https://www.mcp.run/**
* **https://mcp.composio.dev/**
* **https://www.gumloop.com/mcp**

**It can be seen that MCP as a key link in the LLM ecological chain is very prosperous, and even all the traditional APIs and tools should be programmed by MCP. For programmers, MCP is a collection of efficiency tools, saying goodbye to reinventing the wheel.**
