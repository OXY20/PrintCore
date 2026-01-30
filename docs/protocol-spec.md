# 协议与接口详细设计

## 设计目标
- 清晰分离控制面与数据面。
- 支持大文件流式上传与断点续传扩展。
- 兼容多打印机、多客户端、多队列。

## 术语
- Printer：服务端物理打印机或逻辑队列。
- Job：一次打印任务。
- Client：提交打印任务的终端。

## 资源模型
### Printer
- id: string
- name: string
- location: string
- status: enum (idle, busy, offline)
- capabilities: object
  - color: bool
  - duplex: enum (none, long-edge, short-edge)
  - paper_sizes: [string]
  - resolutions: [string]
  - input_bins: [string]

### Job
- id: string
- printer_id: string
- client_id: string
- status: enum (queued, printing, completed, failed, canceled)
- created_at: string (ISO-8601)
- updated_at: string (ISO-8601)
- options: object
  - copies: int
  - duplex: enum (none, long-edge, short-edge)
  - color: enum (auto, color, mono)
  - paper_size: string
- error: optional string

## API（HTTP/JSON + 流式上传）
### 获取打印机列表
- GET /printers
- 响应：Printer[]

### 获取打印机能力
- GET /printers/{id}
- 响应：Printer

### 创建打印作业（元数据）
- POST /jobs
- 请求：
  - printer_id
  - options
- 响应：
  - job_id
  - upload_url (可选；用于分块上传或直接上传)

### 上传作业内容
- PUT /jobs/{id}/content
- Content-Type: application/pdf 或 application/octet-stream
- 支持：chunked transfer 或 Content-Length
- 响应：上传成功状态

### 查询作业状态
- GET /jobs/{id}
- 响应：Job

### 作业取消
- POST /jobs/{id}/cancel

### 作业列表（可选）
- GET /jobs?printer_id=...&status=...

## WebSocket/Server-Sent Events（可选）
- /events
- 推送作业状态变更

## 错误码建议
- 400: 参数错误
- 401: 未认证
- 403: 无权限
- 404: 打印机或作业不存在
- 409: 状态冲突
- 415: 不支持的文件类型
- 500: 服务端内部错误

## 认证与安全
- 简单模式：Bearer Token
- 加强模式：mTLS 或自签证书 + token
- 建议提供本地网络白名单
