# ThinkTrail

> 追溯你的 AI 对话思维轨迹 — 把分散的 Cowork session 变成一张可交互、可编辑的思维导图。

ThinkTrail 扫描你的 Claude Cowork 对话历史，提取主题层级，跨 session 合并同类讨论，生成一张可拖拽、可编辑的 HTML mindmap。你再也不会忘记自己的思路从哪里出发、延伸到了哪里。

## 快速开始

```
/thinktrail
```

选 session → 确认大纲 → 打开导图。

## 为什么需要

多个 AI 对话窗口、多条线程、跨越多天。你忘了聊过什么、哪里想得深、哪里只是提了一嘴。ThinkTrail 从你的 Cowork transcript 中逆向重建思维轨迹。

## 功能

- **默认增量** — 首次之后只读新 session，token 成本极低
- **跨 session 合并** — 不同对话讨论同一话题？自动合并到一个节点下
- **可编辑导图** — 双击编辑文字、拖拽重组层级、右键增删节点
- **全量刷新** — `--full --sessions <ids>` 从头重读

## 演示

浏览器打开 [demo/example-mindmap.html](demo/example-mindmap.html)。

## 关键词

思维导图、AI对话历史、Cowork skill、Claude session追踪、知识图谱、思维整理、对话可视化、第二大脑、AI agent记忆、session总结、头脑风暴工具
