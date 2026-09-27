# M6/M7 扩建：Redis/Nginx 实验的实测经验（2026-09-27）

为本课程新增 r1/r2/r3（Redis 事件总线）与 n1/n2（Nginx 部署）实验过程中实测确认的工程事实：

1. **Redis 版本**：官方不支持 Windows 原生；tporadowski 移植停在 5.0.14.1（不支持 RESP3 的
   `HELLO`，而 redis-py 8 默认发 HELLO → 连接报错）。课程改用 Docker 官方 `redis:7-alpine`
   （course-redis 容器，宿主 6399），fakeredis 仅作单进程降级。
2. **r3 必须用 redis.asyncio**：同步客户端的 Pub/Sub `get_message` 是阻塞调用，
   放进 async 服务会卡死事件循环（L1.1 教训的分布式版）。
3. **Pub/Sub 不留存**：属主订阅未就绪时发布的命令会消失。客户端控制必须
   "重发直到看到状态反馈"（verify 的 send_until 模式）。
4. **Nginx 容器访问宿主**：`host.docker.internal`；后端必须绑 `0.0.0.0`
   （默认 127.0.0.1 会拒绝虚拟网卡的连接 → 502）。`proxy_pass` 用变量时需要 `resolver`。
5. **read_timeout 的精确语义**：上游静默超过阈值才掐，数据到达会重置定时器。
   复现事故的可靠方法 = 后端制造静默窗口（模拟 LLM 思考），而不是期待缓冲攒流——
   快链路+小响应下 buffering on/gzip 未必能复现延迟（gzip 还要求客户端带
   Accept-Encoding 头——"curl 正常浏览器卡死"的根源）。
6. **孤儿进程陷阱**：Windows 上验证脚本异常退出会留下占住端口的旧代码 uvicorn；
   自动化验证开始前必须按端口清残（kill_port 模式），结束用 kill 而非 terminate。

**Implications**：M6/M7 的实验代码与 README 已内嵌这些对策；后续给课程加
分布式实验时先读 learning-records/0003（archify 几何）与 0004（本条）。
