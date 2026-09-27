# 起点评估：已有 Agent/RAG 基础，网络通信为零

用户（2026-09-26）自述已学习 agent 基础与 RAG 基础——理解 agent loop、tool calling、RAG 流程，这些属于"Agent 内部运行机制"；但 HTTP、异步 Web 服务、流式传输、SSE、WebSocket 均未系统学习过。依据：用户自述 + 提供的学习路线文档将通信层全部列为待学内容。

**Implications**：最近发展区从"HTTP 报文 + 第一个 FastAPI 服务"开始，而不是从 ASGI 源码或前端框架开始；教学顺序应严格自底向上（async → HTTP → FastAPI → Streaming → SSE → WebSocket → Agent Event），每层配最小实验。不要重复教 agent loop / RAG 本身，只教"如何把它们接出到网络上"。
