# ThinkTrail

> Cross-session mindmap for AI conversations — trace your thinking across multiple chat windows.

You open multiple Cowork sessions throughout the day around a theme or project. By the end, you can't remember: where you started, which directions branched off, what went deep and what was barely mentioned.

ThinkTrail scans your Cowork session history, extracts topic hierarchies, merges related topics across sessions, and generates an **interactive, editable, draggable** HTML mindmap.

## Trigger

```
/thinktrail
```

Or `/tt`, or natural language like "help me organize my thinking".

## How It Works

1. Lists your 20 most recent sessions — you pick which to include
2. Reads transcripts — extracts topic hierarchy from each
3. Merges related topics across sessions — shows you the outline for confirmation
4. Generates an editable HTML mindmap

**Incremental mode is on by default**: the 100th invocation costs almost zero tokens — only new sessions are read.

## Flags

| Flag | Description |
|---|---|
| `--full --sessions <ids>` | Full re-read of specified sessions |
| `--name <topic>` | Custom root node name |

## Rendering

Built on [SimpleMindMap](https://github.com/wanglin2/mind-map) (MIT). Double-click to edit nodes, drag to reorganize, right-click for node menu.

## Demo

Open [demo/example-mindmap.html](demo/example-mindmap.html) in your browser.

## License

MIT
