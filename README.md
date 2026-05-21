# ThinkTrail

> 跨 Session 思维导图 — 追溯你的 AI 对话思维轨迹

你在一天中围绕某个主题或项目，开了多个 Cowork 对话窗口。聊完你记不住：从哪出发、延伸出哪些方向、哪里想得深哪里只是提了一嘴。

ThinkTrail 扫描你的 Cowork session 历史，提取主题层级，跨 session 合并同类话题，生成一张**可交互、可编辑、可拖拽**的 HTML 节点连线 mindmap。

## 触发

```
/thinktrail
```

或 `/tt`，或自然语言「帮我整理思维导图」。

## 怎么工作

1. 列出最近 20 个 session → 你选要包含哪些
2. 读取 transcript → 提取主题层级
3. 跨 session 合并同类话题 → 展示大纲让你确认
4. 生成可编辑 HTML mindmap → 落到 `F:\AI support work\Mindmap\`

**增量模式默认开启**：第 100 次触发的 token 成本只来自新增对话。

## 参数

| 参数 | 说明 |
|---|---|
| `--full --sessions <ids>` | 对指定 session 全量重读 |
| `--name <主题>` | 自定义 Root 节点名 |

## 渲染

使用 [SimpleMindMap](https://github.com/wanglin2/mind-map)（MIT 开源）渲染。可双击编辑节点文字、拖拽重组层级、右键增删节点。

## 演示

打开 [KITE-思维轨迹.html](demo/KITE-思维轨迹.html) 查看示例 mindmap。

## 许可

MIT
