# fastapi.sse 的三个工程事实（0.141.1 实测）

在为本课程 labs 编写 SSE 实验时实测确认（证据：v3b/v5/v6/v7 启动与请求日志）：
1. `fastapi.sse` 端点函数**必须自身是 async generator**（函数体 yield）。`return EventSourceResponse(gen())` 或返回裸 generator 都会在请求时抛 `'coroutine' object is not iterable`——框架在装饰器阶段就把端点函数本身当作事件流。
2. 返回注解 `AsyncIterable[ServerSentEvent]` 只在端点函数本身是 generator 时合法；否则路由创建阶段就报 "Invalid args for response field"。
3. 需要 404 等非 200 语义时，校验必须放进 `Depends` 依赖（在响应开始前执行）；在 generator 内 raise HTTPException 只会把已发出的 200 流中途掐断。

另外两条库差异（v3c 实测）：sse-starlette 要求 `id` 为 str（int 抛 TypeError），且 data 传 dict 时用 `str()` 而非 JSON 编码，应自行 `json.dumps`。

**Implications**：课程所有 SSE 代码统一采用"yield 风格 + Depends 校验"模式；后续 /teach 会话若涉及 fastapi.sse 升级，先重跑 labs 冒烟测试再更新代码示例。
