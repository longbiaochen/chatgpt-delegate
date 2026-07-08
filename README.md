# ChatGPT Delegate

ChatGPT Delegate lets Codex hand off substantive work to ChatGPT Web/App while Codex only orchestrates.

It is useful for deep research, PRD drafting, design thinking, comparison writing, community best-practice scans, and quick current-information lookups where you want ChatGPT to spend the thinking/search effort and save the final Markdown back to the local filesystem.

Project page: https://longbiaochen.github.io/chatgpt-delegate/

## Boundary

- Codex prepares the task, starts the local MCP connector, opens ChatGPT, sends the prompt, and waits for a saved file.
- ChatGPT Web/App performs the research, thinking, writing, or lookup.
- The connector only saves ChatGPT's Markdown result under `artifacts/chatgpt-delegation/`.
- The connector does not call Codex, the OpenAI API, private ChatGPT APIs, session tokens, cookies, or reverse-engineered endpoints.
- By default, Codex should return only status and a file link, not paste the full result back into chat.

## Install

```bash
git clone https://github.com/longbiaochen/chatgpt-delegate.git
cd chatgpt-delegate
python3 -m venv .venv
./.venv/bin/pip install -e ".[chatgpt,test]"
```

## CLI

Prepare a task:

```bash
chatgpt-delegate prepare "调研一下 MCP 社区最佳实践" --kind auto
```

Start the MCP server:

```bash
chatgpt-delegate serve --host 127.0.0.1 --port 8000 --public-host <public-host>
```

Check status:

```bash
chatgpt-delegate status <task-id>
```

Read metadata without loading the Markdown body:

```bash
chatgpt-delegate read <task-id> --metadata-only
```

List recent results:

```bash
chatgpt-delegate list
```

The old console entry remains available as a compatibility alias:

```bash
chatgpt-research-connector --help
```

## Profiles

`--kind auto` routes common Chinese trigger words to one of three prompt profiles:

| Intent | Kind | Profile | Behavior |
| --- | --- | --- | --- |
| 调研, 深度研究, 社区最佳实践, 研究一下 | `research` | `research_pro` | Thinking pro style, web research, long Markdown report, source list |
| 设计, 构思, 思考, 比较, 撰写, 方案, PRD | `think` | `thinking_high` | Thinking high style, structured proposal/PRD/comparison/writing |
| 查一下, 找一下, 确认, 最新 | `lookup` | `instant_web` | Instant web style, short answer with evidence and links |

## Data Flow

1. In Codex, ask: `让 ChatGPT 调研一下 xxx`.
2. Codex runs `chatgpt-delegate prepare ...` and gets `task_id` plus `prompt.md`.
3. Codex exposes the local MCP server through an HTTPS tunnel and configures ChatGPT Developer connector URL as `https://<public-host>/mcp`.
4. Codex opens ChatGPT Web/App, selects the connector, and sends `prompt.md`.
5. ChatGPT performs the work and calls:

   ```text
   save_task_result(task_id, title, markdown, summary=None, overwrite=True)
   ```

6. The connector writes:

   ```text
   artifacts/chatgpt-delegation/<task_id>/task.json
   artifacts/chatgpt-delegation/<task_id>/prompt.md
   artifacts/chatgpt-delegation/<task_id>/result.md
   artifacts/chatgpt-delegation/<task_id>/result.json
   ```

7. Codex polls `chatgpt-delegate status <task-id>` and returns the saved file path.

## MCP Tools

- `save_task_result(task_id, title, markdown, summary=None, overwrite=True)`
- `list_results(limit=20)`
- `read_result(task_id)`
- `connector_status()`
- `save_markdown_report(...)` for older prompts

`connector_status()` must report:

```json
{
  "codex_execution": false,
  "openai_api": false,
  "private_chatgpt_api": false
}
```

## Codex Skill

This repo includes a reusable skill at:

```text
skills/chatgpt-delegate/SKILL.md
```

Install or copy it into your Codex skills directory, then trigger it with requests such as:

```text
让 ChatGPT 调研一下 100M 上下文大模型的社区方法
用 ChatGPT 想一下这个 PRD
请 ChatGPT 查一下最新进展
```

## Tests

```bash
python3 -m pytest -q
git diff --check
```
