# 全量回归"假失败"的三个根因与修复（2026-09-29 实测）

接手体检时 `python smoke_test.py` 报 **10 通过 / 4 失败**（r2/r3/n1/n2 全灭），
但逐个直跑又全绿——问题不在实验代码，而在**编码 + 宿主残留**两层脚手架。三条实测结论：

## 1. Windows 管道下 stdout 是 GBK，`print("✔")` 当场中断脚本

PEP 528 只保证**真实控制台**用 UTF-8；Git Bash/mintty、CI、`> log.txt` 拿到的都是管道，
此时 `locale.getpreferredencoding(False)` = cp936（本机 ACP=936 实测）。证据链：

```
python -c "print('\u2714')"  → 管道下子进程 rc=1（UnicodeEncodeError: 'gbk' codec）
父进程 text=True 解同样的流 → UnicodeDecodeError → r.stdout is None
                          → TypeError: unsupported operand type(s) for +: 'NoneType' and 'str'
```

`smoke_test.run_script()` 正是踩在第三行上：r2/r3/n1/n2 被记成"失败"，
而其中 r2/r3 单跑（`PYTHONUTF8=1`）分别 exit=0。
**修法**：所有可运行入口脚本顶部加 `reconfigure(encoding="utf-8", errors="replace")`
（真实控制台下本就 UTF-8，等于无操作）；`run_script` 给子进程显式
`PYTHONIOENCODING=utf-8` + `PYTHONUTF8=1`，父进程 `encoding="utf-8", errors="replace"`
且容忍 `stdout=None`。**验证**：不加任何环境变量直接跑 → **14 通过 / 0 失败 / 0 跳过（171s）**。

## 2. 残留容器/孤儿进程让 n1/n2 假失败，且失败形态误导

宿主 8080 被遗留容器 `course-nginx-demo`（挂载的正是 `n1-good.conf`）占着，
8807 被上一轮 n1 留下的孤儿 uvicorn 占着。后果链：

- `docker run -p 8080` 静默失败（`sh()` 吞掉 docker 输出）→ 请求打到**旧容器** → n2 `KeyError: 'task_id'`
- n1 新 backend bind 失败立即退出 → `wait_ready` 命中**旧进程** → 清理阶段
  `backend.kill()` 抛 `ProcessLookupError`，把已经跑绿的检查一起带崩

**修法**：n1 补 `kill_port()`（对齐 n2 既有做法）+ 启动后校验 backend 存活 + kill 前判 `returncode`；
n1/n2 都加 `docker inspect -f {{.State.Running}}` 校验容器真的起来了，起不来直接
fail-fast 报"宿主 8080 被占"；n2 创建任务处补状态码断言。
**验证**：故意用 `portguard-test` 容器占住 8080 → n1 明确报错 exit=1，且**不留任何残留**
（端口净空、无孤儿进程）；清掉后立刻复跑 n1 exit=0。清理残留后 n1/n2 各自 exit=0
（n1：10 事件 / 跨度 6.81s / WS 透传 / 事故配置掐流；n2：6 任务 + 39 事件 + WS 审批 + 终态 done）。

## 3. 正文里的目录链接 = 必然断链

`tools/build_docs.py` 会把 `[course/](course/)` 重写成 `(course/README.html)`，
而 `course/`、`diagrams/`、`lessons/`、`reference/` 下没有 README → `check_links` 报 5 处断链。
改为指向各目录入口页后：`node tools/check_links.js` → **0 处问题**（40 页接入导航 + 9 张独立图表）。

**Implications**：三条教训已写进 NOTES.md 的硬约定（UTF-8 守卫 / 禁写目录链接 / n1-n2 fail-fast）。
今后看到"回归失败"，先分三类再动手：**编码与脚手架 → 宿主残留 → 真实代码回归**；
只有第三类才需要改实验代码。命令入口：
`python labs/smoke_test.py`、`node tools/check_links.js`、`python tools/build_docs.py`。
