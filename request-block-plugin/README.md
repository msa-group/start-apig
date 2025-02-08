# 概述

**request-block** 是一款用于云原生 API 网关的请求屏蔽插件，通过基于 URL、请求头等特征对 HTTP 请求进行筛选和屏蔽。该插件可用于防护站点资源不被非法访问或暴露，提升系统的安全性和可靠性。

### 场景描述

在现代 Web 应用和微服务架构中，API 网关作为流量入口，需要有效过滤和管理大量的 HTTP 请求。部分资源可能不适合公开暴露，如内部管理接口、开发调试工具等。未经筛选的请求可能导致资源滥用、数据泄露或服务被攻击。因此，具备强大的请求屏蔽能力对于保护系统安全和维护服务健康至关重要。

### 应用场景

1. **保护敏感资源**：屏蔽对内部管理接口、配置接口等敏感资源的外部访问，只允许授权请求通过。
2. **防止恶意攻击**：通过识别和屏蔽来自恶意 IP、恶意 User-Agent 等特征的请求，有效防御 DDoS 攻击、爬虫攻击等。
3. **限制特定用户访问**：基于请求头中的用户标识，如 API Key、Token，限制特定用户或客户端的访问权限。
4. **隐藏后台服务信息**：防止外部用户获取后台服务的详细信息，如服务路径、版本信息等，增强应用的安全性。
5. **流量管理与优化**：屏蔽不必要的流量请求，减轻服务器负载，提高系统的响应效率和稳定性。
6. **遵循合规要求**：根据企业或行业的安全合规标准，屏蔽和管理不符合规定的请求，确保系统运行符合要求。

### 解决问题

- **提升系统安全性**：通过精准屏蔽不合规或恶意的请求，防止潜在的安全威胁和攻击，保障系统的完整性和可用性。
- **保护敏感数据**：阻止未经授权的请求访问敏感资源，避免数据泄露和滥用，维护用户和企业的利益。
- **优化资源利用**：减少不必要的请求流量，降低服务器负载，提升系统的处理效率和用户体验。
- **增强访问控制**：实现细粒度的访问控制策略，根据不同的请求特征制定相应的屏蔽规则，提高访问管理的精度和灵活性。
- **简化运维管理**：提供清晰的配置和日志记录，方便运维人员监控请求流量，调整屏蔽策略，确保系统的稳定运行。
- **支持动态配置**：允许在运行时动态调整屏蔽规则，快速响应安全威胁或业务需求变化，提升系统的适应能力。

## 架构

```mermaid
flowchart TB
A[fa:fa-users http 请求] --> B{fa:fa-route 网关路由}
	B -->|/get| P1[fa:fa-shield-alt request-block]
	P1 --> Backend[fa:fa-server FC Service]
```

本示例`request-block`插件的配置如下：

> 更多配置详情，请查阅[Github 文档](https://github.com/alibaba/higress/blob/main/plugins/wasm-cpp/extensions/request_block/README.md)

### 屏蔽请求 url 路径

```yaml
block_urls:
  - foo=bar
case_sensitive: false
blocked_message: 请求已屏蔽
```

1. 发起未配置安全防护的请求
   首先，尝试不带任何安全防护信息发起请求：

```
curl -iv 'http://env-cu9g82mm1hkui0vcv5eg-cn-hangzhou.alicloudapi.com/get'
```

预期返回结果：

```
{
  "args": {},
  "headers": {
    "Accept-Encoding": "gzip",
    "Host": "requestplugin-p-hgaiodjhwz.cn-hangzhou-vpc.fcapp.run",
    "Original-Host": "env-cu9g82mm1hkui0vcv5eg-cn-hangzhou.alicloudapi.com",
    "Req-Start-Time": "1739006141147",
    "User-Agent": "Go-http-client/1.1",
    "X-Envoy-Attempt-Count": "1",
    "X-Envoy-Internal": "true",
    "X-Envoy-Original-Host": "env-cu9g82mm1hkui0vcv5eg-cn-hangzhou.alicloudapi.com",
    "X-Envoy-Route-Identifier": "true",
    "X-Fc-Access-Key-Id": "",
    "X-Fc-Access-Key-Secret": "",
    "X-Fc-Account-Id": "1419633767709936",
    "X-Fc-Api-Server-Ip": "",
    "X-Fc-Base-Path": "/get",
    "X-Fc-Client-Ip": "",
    "X-Fc-Control-Path": "/http-invoke",
    "X-Fc-Eagleeye-Rpcid": "",
    "X-Fc-Eagleeye-Traceid": "",
    "X-Fc-Eagleeye-Userdata": "",
    "X-Fc-Function-Handler": "index.handler",
    "X-Fc-Function-Memory": "1024",
    "X-Fc-Function-Name": "request-block-plugin-668p",
    "X-Fc-Function-Timeout": "3",
    "X-Fc-Qualifier": "LATEST",
    "X-Fc-Region": "cn-hangzhou",
    "X-Fc-Request-Id": "1-67a720bd-15933561-8ab9b99753be",
    "X-Fc-Retry-Count": "0",
    "X-Fc-Security-Token": "",
    "X-Fc-Service-Logproject": "",
    "X-Fc-Service-Logstore": "",
    "X-Fc-Service-Name": "",
    "X-Fc-Tracing-Jaeger-Endpoint": "",
    "X-Fc-Tracing-Opentracing-Span-Baggages": "",
    "X-Fc-Tracing-Opentracing-Span-Context": "",
    "X-Fc-Version-Id": ""
  },
  "origin": "172.16.19.194, 100.117.33.55",
  "url": "http,http://requestplugin-p-hgaiodjhwz.cn-hangzhou-vpc.fcapp.run/get"
}
```

2. 发起带安全防护参数的请求
   为确保请求时触发安全防护，请在请求地址中添加 foo=bar 参数。以下是示例命令：

```
curl -iv 'http://env-cu9g82mm1hkui0vcv5eg-cn-hangzhou.alicloudapi.com/get?foo=bar'
```

预期返回结果：

```
<!-- 返回内容可通过  blocked_message 配置 -->
请求已屏蔽
```

### 屏蔽请求 header

```yaml
block_headers:
  - example-key
  - example-value
case_sensitive: false
blocked_message: 请求已屏蔽
```

1. 发起未配置安全防护的请求
   首先，尝试不带任何安全防护信息发起请求:

```
curl -iv 'http://env-cu9g82mm1hkui0vcv5eg-cn-hangzhou.alicloudapi.com/get' -H 'example-test: test'
```

预期返回结果：

```
{
  "args": {},
  "headers": {
    "Accept-Encoding": "gzip",
    "Example-Test": "test",
    "Host": "requestplugin-p-hgaiodjhwz.cn-hangzhou-vpc.fcapp.run",
    "Original-Host": "env-cu9g82mm1hkui0vcv5eg-cn-hangzhou.alicloudapi.com",
    "Req-Start-Time": "1739007119680",
    "User-Agent": "Go-http-client/1.1",
    "X-Envoy-Attempt-Count": "1",
    "X-Envoy-Internal": "true",
    "X-Envoy-Original-Host": "env-cu9g82mm1hkui0vcv5eg-cn-hangzhou.alicloudapi.com",
    "X-Envoy-Route-Identifier": "true",
    "X-Fc-Access-Key-Id": "",
    "X-Fc-Access-Key-Secret": "",
    "X-Fc-Account-Id": "1419633767709936",
    "X-Fc-Api-Server-Ip": "",
    "X-Fc-Base-Path": "/get",
    "X-Fc-Client-Ip": "",
    "X-Fc-Control-Path": "/http-invoke",
    "X-Fc-Eagleeye-Rpcid": "",
    "X-Fc-Eagleeye-Traceid": "",
    "X-Fc-Eagleeye-Userdata": "",
    "X-Fc-Function-Handler": "index.handler",
    "X-Fc-Function-Memory": "1024",
    "X-Fc-Function-Name": "request-block-plugin-668p",
    "X-Fc-Function-Timeout": "3",
    "X-Fc-Qualifier": "LATEST",
    "X-Fc-Region": "cn-hangzhou",
    "X-Fc-Request-Id": "1-67a7248f-157f9e7f-ba1d831519dc",
    "X-Fc-Retry-Count": "0",
    "X-Fc-Security-Token": "",
    "X-Fc-Service-Logproject": "",
    "X-Fc-Service-Logstore": "",
    "X-Fc-Service-Name": "",
    "X-Fc-Tracing-Jaeger-Endpoint": "",
    "X-Fc-Tracing-Opentracing-Span-Baggages": "",
    "X-Fc-Tracing-Opentracing-Span-Context": "",
    "X-Fc-Version-Id": ""
  },
  "origin": "172.16.37.65, 100.117.33.211",
  "url": "http,http://requestplugin-p-hgaiodjhwz.cn-hangzhou-vpc.fcapp.run/get"
}
```

2. 发起带安全防护 Header 的请求
   为确保请求时触发安全防护，请在请求头中添加 example-key。以下是示例命令：

```
curl -iv 'http://env-cu9g82mm1hkui0vcv5eg-cn-hangzhou.alicloudapi.com/get' -H 'example-key: 102234'
curl -iv 'http://env-cu9g82mm1hkui0vcv5eg-cn-hangzhou.alicloudapi.com/get' -H 'my-header: example-value'
```

预期返回结果：

```
<!-- 返回内容可通过  blocked_message 配置 -->
请求已屏蔽
```

### 屏蔽请求 body

```yaml
block_bodies:
  - "hello world"
case_sensitive: false
blocked_message: 请求已屏蔽
```

1. 发起未配置安全防护的请求
   首先，尝试不带任何安全防护信息发起请求:

```
curl -iv 'http://env-cu9g82mm1hkui0vcv5eg-cn-hangzhou.alicloudapi.com/get' -d 'example hello world'
```

预期返回结果：

```
{
  "args": {},
  "headers": {
    "Accept-Encoding": "gzip",
    "Content-Length": "21",
    "Content-Type": "application/json",
    "Host": "requestplugin-p-hgaiodjhwz.cn-hangzhou-vpc.fcapp.run",
    "Original-Host": "env-cu9g82mm1hkui0vcv5eg-cn-hangzhou.alicloudapi.com",
    "Req-Start-Time": "1739007475849",
    "User-Agent": "Go-http-client/1.1",
    "X-Envoy-Attempt-Count": "1",
    "X-Envoy-Internal": "true",
    "X-Envoy-Original-Host": "env-cu9g82mm1hkui0vcv5eg-cn-hangzhou.alicloudapi.com",
    "X-Envoy-Route-Identifier": "true",
    "X-Fc-Access-Key-Id": "",
    "X-Fc-Access-Key-Secret": "",
    "X-Fc-Account-Id": "1419633767709936",
    "X-Fc-Api-Server-Ip": "",
    "X-Fc-Base-Path": "/get",
    "X-Fc-Client-Ip": "",
    "X-Fc-Control-Path": "/http-invoke",
    "X-Fc-Eagleeye-Rpcid": "",
    "X-Fc-Eagleeye-Traceid": "",
    "X-Fc-Eagleeye-Userdata": "",
    "X-Fc-Function-Handler": "index.handler",
    "X-Fc-Function-Memory": "1024",
    "X-Fc-Function-Name": "request-block-plugin-668p",
    "X-Fc-Function-Timeout": "3",
    "X-Fc-Qualifier": "LATEST",
    "X-Fc-Region": "cn-hangzhou",
    "X-Fc-Request-Id": "1-67a725f3-1541092d-64135fc2aa6d",
    "X-Fc-Retry-Count": "0",
    "X-Fc-Security-Token": "",
    "X-Fc-Service-Logproject": "",
    "X-Fc-Service-Logstore": "",
    "X-Fc-Service-Name": "",
    "X-Fc-Tracing-Jaeger-Endpoint": "",
    "X-Fc-Tracing-Opentracing-Span-Baggages": "",
    "X-Fc-Tracing-Opentracing-Span-Context": "",
    "X-Fc-Version-Id": ""
  },
  "origin": "172.16.37.65, 100.117.33.237",
  "url": "http,http://requestplugin-p-hgaiodjhwz.cn-hangzhou-vpc.fcapp.run/get"
}
```

2. 发起带安全防护 Header 的请求
   为确保请求时触发安全防护，请在请求头中添加 example-key。以下是示例命令：

```
curl -iv 'http://env-cu9g82mm1hkui0vcv5eg-cn-hangzhou.alicloudapi.com/get' -d 'hello world'
```

预期返回结果：

```
<!-- 返回内容可通过  blocked_message 配置 -->
请求已屏蔽
```

#### 注意事项

#### 请求 Body 大小限制

当配置了 block_bodies 时，仅支持小于 32 MB 的请求 Body 进行匹配。若请求 Body 大于此限制，并且不存在匹配到的 block_urls 和 block_headers 项时，不会对该请求执行屏蔽操作 当配置了 block_bodies 时，若请求 Body 超过全局配置 DownstreamConnectionBufferLimits，将返回 413 Payload Too Large
