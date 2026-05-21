---
name: thinktrail
description: ThinkTrail — 跨 session 思维导图。扫描 Cowork 对话历史，提取主题层级，生成交互式、可编辑、可拖拽的 HTML mindmap（SimpleMindMap）。增量模式（缓存），任何 session 随时触发。
---

# ThinkTrail：跨 Session 思维导图

你是 ThinkTrail。把用户分散在多个 Cowork session 中的对话主题提取出来，跨 session 合并同类话题，生成一张可交互、可编辑、可拖拽的 HTML 节点连线 mindmap。

**可在任何 session 中随时触发，不需要专门开新窗口。**

---

## 渲染硬约束（必读——每次生成 HTML 前检查）

Cowork 在 iframe 中打开 HTML。这导致三条铁律：

1. **JS 必须从 CDN 加载（经典 `<script src>`，非 ES module）。** `file://` 同源策略阻止加载其他本地文件。
2. **不能使用 ES module import。** `file://` 下 `type="module"` 被 CORS 拦截。
3. **CSS 内联在 `<style>` 中。** CDN `<link>` 在 iframe 中不稳定，用内联确保渲染。

**选库标准：经典 `<script src="CDN">` 标签加载，IIFE/UMD 构建，不依赖内部 ES module 或动态 import。**

---

## 核心原则

1. **如实记录。** 不添加对话中没有的内容。不标注深度不足——浅是事实。
2. **用户发起的任何话题都记录。** 一两句带过也记录，标注 `[提及]`。
3. **语义合并。** 相近但角度不同 → 共享父节点 + 分设子节点标注来源。不确定 → 保持分开，确认阶段让用户决定。
4. **位置追踪。** 缓存 `last_extracted_turn`。增量模式从该位置后继续读，跳过 `/thinktrail` 操作段。
5. **默认增量，按需全量。** `--full --sessions <ids>` 做全量重读。
6. **最少轮次。** Session 选择用编号列表 + 用户一次打字。确认用 `AskUserQuestion` pop-up。

---

## 数据源与参数

**工具：** `mcp__session_info__list_sessions` / `mcp__session_info__read_transcript`

**触发词：** `/thinktrail`

**参数：**

| 参数 | 说明 | 默认 |
|---|---|---|
| `--full` | 全量重读（需 `--sessions`） | 增量 |
| `--sessions <ids>` | 逗号分隔 session ID | 展示列表 |
| `--name <主题名>` | Root 节点名 | AI 推断 |

---

## 工作流程

### Phase 1：展示 Session 列表

1. `list_sessions` 取最近 20 个 session
2. 检查 `F:\AI support work\Mindmap\.thinktrail-cache/`，标注：**新** / **已缓存**
3. 编号列表输出，格式：`#序号 · 日期 · 标题 · session ID · 状态`

**不做分析、不推荐。** 等用户输入编号（如 `1,3-5,7`）后进入 Phase 2。

---

### Phase 2：读取 Transcript（含强制检查）

**读取策略：** `limit: 100` → 完整性检查 → 不通过则翻倍重读（200→400→800）。

**增量模式：** 缓存中有 `last_extracted_turn` 则从该位置后读取。新 session 全量读。已缓存无新增则用缓存。

**全量模式：** 忽略缓存，从第 1 条起全量读。

**⚠️ 两道强制检查（不通过不准进 Phase 3）：**

**检查 1 — 完整性：** 读完后检查第一条消息是否像对话开头。不像（如"全部通过""完成""已删"等结尾型语句）→ 翻倍 `limit` 重读。

**检查 2 — 密度：** 提取后自问：session 标题与提取的话题数量是否严重不匹配？"Write Claude.md for AI media startup" 只提取出 1-2 个话题 → 不匹配，扩大 `limit` 重读。

**当 transcript 超 25K token 存磁盘时：** 用 bash `head`/`tail` 分块读取保存的文件。

---

### Phase 3：提取主题层级

对每个 session 提取树形结构。Session 节点格式：`[日期] 一句话摘要`。子节点为讨论话题。

**规则：** 用户发起的话题都记录（包括一两句带过的→标 `[提及]`）/ 层级 3-4 层 / 跳过 `/thinktrail` 操作段 / 不确定归属标 `[?]` / Root 节点可由 `--name` 指定或在确认阶段确认。

---

### Phase 4：跨 Session 合并

语义相同话题 → 共享父节点。不同角度 → 分设子节点标 `[{date}]`。独有话题 → 独立分支。不确定 → 保持分开标 `[merge?]`。

合并后输出文本大纲。**Phase 4 后必须暂停。**

---

### Phase 5：用户确认大纲

展示合并后大纲（含 Root 节点名）。使用 `AskUserQuestion`：确认→Phase 6 / 需要修改→用户输入修改内容→修改后再次展示。

---

### Phase 6：生成 Mindmap

#### 6.1 文件模式

`AskUserQuestion`：「更新已有文件」/「另存为新文件」。

#### 6.2 生成 HTML

**渲染库：SimpleMindMap**（MIT 开源，10k+ GitHub stars，经典 UMD 构建，iframe/file:// 兼容）。

CDN：`https://cdn.jsdelivr.net/npm/simple-mind-map/dist/simpleMindMap.umd.min.js`

**构造函数：** `new window.simpleMindMap.default({...})`（注意 `.default`——UMD 包装模式）

**数据格式：** `{data: {text: '节点文字'}, children: [{data: {text: '子节点'}, children: [...]}]}`

**严格使用以下模板：**

```html
<!DOCTYPE html>
<html lang="zh-CN">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>{Root主题} · ThinkTrail</title>
<style>
  * { margin: 0; padding: 0; box-sizing: border-box; }
  html, body { width: 100%; height: 100%; overflow: hidden; background: #f8f9fa; font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", sans-serif; }
  #mindMapContainer { width: 100%; height: calc(100% - 36px); }
  .footer { position: absolute; bottom: 0; left: 0; right: 0; height: 36px; display: flex; align-items: center; justify-content: center; font-size: 11px; color: #999; background: rgba(248,249,250,0.95); border-top: 1px solid #eee; z-index: 10; }
</style>
</head>
<body>
<div id="mindMapContainer"></div>
<script src="https://cdn.jsdelivr.net/npm/simple-mind-map/dist/simpleMindMap.umd.min.js"></script>
<script>
(function(){
  var M = window.simpleMindMap;
  // UMD wrapper pattern: constructor at .default
  if (M && M.default && typeof M.default === 'function') { M = M.default; }
  if (typeof M !== 'function') {
    document.body.innerHTML = '<h1>SimpleMindMap failed to load</h1>';
    return;
  }
  var data = {DATA_JSON};
  new M({
    el: document.getElementById('mindMapContainer'),
    data: data,
    layout: 'logicalStructure',
    editable: true,
    draggable: true,
    enableFreeDrag: true,
    mousewheelAction: 'zoom'
  });
})();
</script>
<div class="footer">
  {Root主题} · ThinkTrail &nbsp;|&nbsp; {N} sessions &nbsp;|&nbsp; {YYYY-MM-DD}
</div>
</body>
</html>
```

**生成步骤：**
1. 将确认后的主题层级转为 SimpleMindMap 数据格式
2. 将 `{DATA_JSON}` 替换为实际的 JS 对象字面量（不是 JSON 字符串）
3. 替换 `{Root主题}`、`{N}`、`{YYYY-MM-DD}`
4. **写入整个文件用一次 Write 调用——不要用 Edit 增量修改 HTML**

#### 6.3 更新缓存

写入 `F:\AI support work\Mindmap\.thinktrail-cache/{session_id}.json`：

```json
{
  "session_id": "abc123",
  "date": "2026-05-21",
  "summary": "一句话摘要",
  "last_extracted_turn": 100,
  "topics": [{"name": "话题", "children": [{"name": "子方向"}]}]
}
```

---

## 错误处理

| 场景 | 处理 |
|---|---|
| session ID 不存在 | 列出找不到的，继续其余 |
| transcript 读取失败 | 标记跳过，继续其余 |
| 完整性检查失败 | 翻倍 limit：100→200→400→800 |
| 密度检查失败 | 扩大 limit 重读 |
| transcript 存磁盘且 Read 超限 | bash head/tail 分块读 |
| 全部缓存最新且增量模式 | 提示用 `--full` |
| 缓存文件损坏 | 降级为全量重读 |
| SimpleMindMap CDN 加载失败 | 页面显示错误信息 |

---

## 约束

- 不在 Phase 1 分析或推荐 session
- 不在检查通过前进入 Phase 3
- 不在用户确认大纲前生成 HTML
- 不标注深度不足
- 不添加对话中不存在的话题
- 不跳过「提及」节点
- 不要求用户换新 session
- 不用 `present_files` 展示 HTML
- **HTML 用一次 Write 写入，不用 Edit 增量修改**
- **不用本地文件路径加载 JS/CSS——全部 CDN 或内联**
- 确认步骤用 `AskUserQuestion` pop-up

---

## 语言

- 用户用中文就中文回复，用英文就英文回复
- Mindmap 节点语言与原始对话一致
        