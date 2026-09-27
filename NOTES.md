# NOTES

## 用户画像与偏好

- 自述已学习 agent 基础与 RAG 基础（见 learning-records/0001），网络通信与后端服务是全新领域
- 希望被当作初学者对待：概念从直觉和生活比喻入手，再进协议细节
- 明确要求：落实到**具体实操**和**代码流程讲解**，不要停留在抽象概念
- 明确要求：参考成熟开源项目，能复用就不重复造轮子；手写只为了理解原理
- 授权"全权交给"讲师视角设计，无需每步确认
- 语言：中文（代码标识符、协议字段、命令保留英文）

## 教学约定

- 每个实验的代码都在 `labs/` 下可实际运行，课程文档中引用的代码行为以实测为准
- 实验端口固定使用 8801–8807，避免与本机常用 8000 冲突
- 事件协议教学对齐 AG-UI / Dify，不自造命名
- 课程文档（course/、COURSE.md）是主教材；lessons/*.html 是每课的复习卡；reference/*.html 是速查表

## 工作方式

- 用户通过 /teach 进入学习会话：按 COURSE.md 的进度找当前课，产出 lessons/ 下下一编号的 HTML 课
- lessons/ 现有：0001 课程地图（含测验）、0002 SSE 格式实验室（交互解析器）、0003 Agent 事件时间线浏览器（单步回放）；下一课从 0004 起
- **所有 lessons/、reference/、labs 实验页面与生成的 .html 文档页必须引入共享导航组件**：`<script src="{base}assets/nav.js" data-base="{base}" data-track="lesson|readings|reference"></script>`（放在 </body> 前；base 按页面深度，根目录是空字符串）。它注入左侧目录抽屉（全课程地图+当前页高亮）和 lesson/readings 页底部的翻页条；页面内不要再手写"下一课"链接。**根目录页的 data-base="" 是合法值**，nav.js 已按 `!== undefined` 判断（勿改回 `|| "../"`，空字符串是假值会跳级）。改动后跑 `node tools/check_links.js` 全站自检
- **markdown 文档是源文件**：COURSE/GLOSSARY/README/MISSION/RESOURCES/course/*/labs README 均由 `python tools/build_docs.py` 渲染为同款样式的 .html（含侧栏与讲义篇间翻页）。改 .md 后必须重跑构建；新增文档要登记进 build_docs.py 的 BUILD 列表。不要手改生成的 .html
- 每完成一个模块，写一条 learning-record 记录证据
- 改动 labs/ 代码或升级依赖后，必须跑 `labs/smoke_test.py` 回归（9 个实验端到端）
- 图表在 diagrams/（archify 产出，共 6 张），课程文档引用它们；新增图表遵循 learning-records/0003 的几何规则
