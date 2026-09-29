# archify 画图的实测几何规则（本工作区 6 张图的经验）

为本课程产出首批 6 张 archify 图的过程中实测确认的约束（证据：diagrams/src 各 json 的 validate 记录）；这些预算对后续新增的图同样适用——M6/M7 补齐 06–09 后，工作区现共有 9 张图，且 07/08/09 未再触发新的几何失败：

1. **可读性红线**：`scale = min(1, 930 / viewBox宽度)`；`data-node-label`（11px）与 `data-detail="context"` 子标签（7px）投影后必须 ≥6px。因此 sequence 的 viewBox 宽度上限 ≈1080（7px 子标签），architecture 的 viewBox 宽度上限 ≈1395（9px 子标签）。06 图曾在 1408 宽时以 0.06px 之差失败。
2. **浏览器视口高度**：SVG 随阅读宽度（~1410px）等比放大，总页高（头部+SVG+卡片）不得超出 900/1320 视口。sequence 需把消息控制在 viewBox[1]−83 以内、segment 在 −45 以内；内容过宽的图要"横向铺开"而不是压缩。
3. **参与者居中**：sequence 默认把参与者挤在左侧 86px 盒 + 108px 间距；viewBox 较宽时必须加 `column_fit: "spread"`，否则右侧大片空白（02 首版即此问题）。
4. **参与者子标签过长会触发 shrink-to-fit**，缩到 6px 下限以下即触发可读性失败（05 首版 "EventSource + WS" 即此问题）——子标签保持 ≤8 个字符宽度单位。
5. **图例**贴着内容底部渲染：内容过满时 `artifact/legend-clearance` 报错，隐藏图例（`meta.legend.mode: "hidden"`）即可。
6. **同库差异**：架构图的 region 节点允许较大自由度；sequence 消息与 segment 边界需保持 ≥8px 距离，否则 container-border-run。

**Implications**：后续在本工作区新增图表，直接按上述预算设计尺寸，可省掉大量试错轮次；若 archify 升级，先用 01/05 两张作金标准重跑 validate 对比行为。
