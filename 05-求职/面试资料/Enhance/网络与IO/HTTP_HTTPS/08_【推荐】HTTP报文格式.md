# HTTP报文格式？

HTTP报文由两部分组成：**报文头（Header）**和**报文体（Body）**，中间用一个空行分隔。

## HTTP请求报文格式

```
GET /index.html HTTP/1.1        ← 请求行
Host: www.example.com           ← 请求头部
User-Agent: Mozilla/5.0
Accept: text/html
Connection: keep-alive
                                ← 空行
username=admin&password=123     ← 请求体（可选）
```

**请求行包含三部分：**

- **请求方法**：GET、POST、PUT、DELETE等
- **请求URI**：资源路径，如 `/index.html`
- **HTTP版本**：如 `HTTP/1.1`

## HTTP响应报文格式

```
HTTP/1.1 200 OK                 ← 状态行
Content-Type: text/html         ← 响应头部
Content-Length: 1024
Server: nginx/1.18.0
                                ← 空行
<html>                          ← 响应体
<body>Hello World</body>
</html>
```

**状态行包含三部分：**

- **HTTP版本**：如 `HTTP/1.1`
- **状态码**：如 `200`（成功）、`404`（未找到）
- **状态描述**：如 `OK`、`Not Found`

## 常见头部字段

**请求头部：**

- `Host`：目标服务器域名
- `User-Agent`：客户端信息
- `Accept`：可接受的响应内容类型
- `Cookie`：会话信息
- `Authorization`：认证信息

**响应头部：**

- `Content-Type`：响应内容类型
- `Content-Length`：响应体长度
- `Set-Cookie`：设置cookie
- `Cache-Control`：缓存控制
- `Location`：重定向地址

## 相关扩展知识

**HTTP/2的改进：**

- 使用二进制格式而非文本格式
- 支持多路复用，一个连接可并发处理多个请求
- 头部压缩，减少传输开销

**在Java中的应用：**

- Servlet中通过 `HttpServletRequest` 和 `HttpServletResponse` 操作HTTP报文
- Spring Boot中的 `@RequestHeader` 和 `@ResponseHeader` 注解
- HTTP客户端库如 `HttpClient`、`OkHttp` 用于构建和解析HTTP报文

**性能优化考虑：**

- 合理设置 `Content-Length` 避免分块传输
- 使用 `Connection: keep-alive` 复用TCP连接
- 启用Gzip压缩减少传输体积