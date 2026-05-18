---
id: "ses_2322630e7fc6833f"
title: "需求与设计同步"
createdAt: "2026-05-16T02:00:43.926Z"
updatedAt: "2026-05-18T06:36:29.051Z"
settings: {}
runs: []
---

# 需求与设计同步

<!-- message:id=xtool_history role=assistant createdAt=2026-05-18T06%3A36%3A29.051Z -->

## Assistant

# XTool研发 历史需求摘要

- 起 HTTPS 静态服务，最初展示 `helllo world`。
- 扩展为 Gemini-like Web UI，将请求转交给 agent CLI，当前内置 Codex CLI 插件。
- 支持 notebooks、sessions、上下文保存与展示、notebook notes。
- 支持 agent CLI 图形设置：agent、模型、mode、sandbox、workdir、profile、web search、timeout、安全 config overrides。
- 安装 Superpowers skills，并用子代理做架构和代码 review。
- 服务改为单用户私有，增加预置密码登录。
- 所有数据使用文件存储，改为 notebook/session 可读目录层级。
- 增加鲁棒恢复：非核心数据损坏时保留 corrupt 并恢复可用对象。
- 增加 favicon/logo。
- 增加登录失败 rate limit 和 file audit log。
- 增加回收站：删除 notebook/session 先移动到 trash，支持永久删除。
- 增加手机小屏适配。
- 增加回答下载和 agent run 产物下载。
- 增加本同步 notebook/session，并增加独立于 session 的文件/目录上传。
- 产品名改为 AIAssitant，并将 Codex CLI 访问抽象为内置 agent 插件，便于未来支持其它 agent CLI。
- 增加删除 session 中某一轮对话的能力，用于精简上下文记录。
- 支持二级子笔记本，明确问答存储在 session 的 messages.jsonl，并避免用户删除过的 Default notebook 被静默补回。

<!-- /message -->
<!-- message:id=xtool_requirements role=assistant createdAt=2026-05-18T06%3A36%3A29.051Z -->

## Assistant

# AIAssitant Requirements

Last updated: 2026-05-18

## Product Goal

Build `AIAssitant`, a private HTTPS web interface that behaves like a lightweight Gemini-style chat workspace while forwarding assistant work to a pluggable agent CLI. The first built-in agent CLI plugin is Codex CLI.

## Historical Requirements

1. Serve an HTTPS web server on this machine, initially for a static "helllo world" page.
2. Replace the static page with a web app that forwards page requests to an agent CLI.
3. Provide notebooks. A notebook can contain multiple sessions.
4. Save and display session context.
5. Provide notebook notes. Notebook notes are included as context for agent requests.
6. Provide a graphical subset of agent CLI settings such as agent selection, model, mode, sandbox, working directory, profile, web search, timeout, and safe config overrides.
7. Install Superpowers skills.
8. Use subagents for deep design/code review when requested.
9. Stop the service immediately when requested.
10. The service is private for one user, but it may be exposed on the public instance address.
11. Add preset-password login before reopening the service.
12. Use file storage only. Do not introduce SQLite or another database.
13. Improve robustness: depend only on a minimal core configuration. If non-core data is corrupt, preserve the corrupt file/directory, recover usable data, and continue.
14. Maintain this requirements document and the design document on each meaningful change.
15. If a new requested change conflicts with earlier requirements or design, stop and ask for a decision before implementing.
16. File storage should use a simple directory hierarchy that makes notebooks, sessions, and their relationships obvious for offline reading.
17. The service should have a browser-visible logo/favicon.
18. Add login failure rate limiting and file-based audit logs.
19. Allow deleting sessions and notebooks by moving them into a trash directory first.
20. Provide permanent deletion for items already in trash.
21. The web UI must remain usable on phone-size screens.
22. Allow downloading a single assistant/error answer from a session.
23. Allow downloading files produced or modified by a specific agent run, without exposing arbitrary server file browsing.
24. Maintain a notebook named `XTool研发` with a synchronized session containing the project requirement history, design document, and requirements document.
25. Provide upload of files and directories independent of any notebook/session.
26. The default Codex plugin model should be `gpt-5.5`; the default mode should be `high`.
27. The right-side global/detail information panel should be collapsible.
28. The product name shown in the UI and browser title must be `AIAssitant`, not `Codex Web`.
29. Access to Codex must be implemented as an agent CLI plugin so future agent CLIs can be added through the same interface.
30. Allow deleting one conversation turn inside a session to simplify saved context.
31. Support one child notebook level under a root notebook.
32. A user-deleted `Default notebook` must not be silently recreated on startup.

## Current Core Requirements

- The service must require login before serving the app or APIs.
- The login secret must be stored server-side only.
- All persistent state must be stored as ordinary files under `data/`.
- Core startup may depend on only:
  - TLS certificate/key files.
  - `data/auth.json`, or `WEB_PASSWORD` for first-time auth bootstrap.
  - The application source files.
- Notebook/session/message/run data corruption must not prevent startup.
- Corrupt data must be preserved under a `corrupt/` directory or with a `.corrupt.<timestamp>` suffix.
- Recovered placeholder data should be explicit and readable offline.
- The directory structure must be human-readable without special tooling.
- Dangerous agent execution settings must be constrained by server-side allowlists.
- `danger-full-access` must not be available through the web UI/API.
- The browser tab/bookmark icon should use a static asset served from `public/`.
- Login failures must be rate-limited per source IP.
- Security-relevant events must be recorded to file-based audit logs under `data/audit/`.
- Deleting a notebook or session must preserve its files under `data/trash/`.
- Permanent deletion must only apply to items already in `data/trash/`.
- Small screens must avoid horizontal overflow and keep core actions reachable.
- Answer downloads must be scoped to an existing session message.
- Generated-file downloads must be scoped to files detected as created or modified during a specific agent run.
- The `XTool研发` notebook/session must be updated when requirements/design changes are made.
- Uploads must be stored outside session data and must not automatically enter conversation context.
- Default settings must keep agent, model, and mode separate: agent `codex`, model `gpt-5.5`, mode `high`.
- The detail panel collapse state may be kept client-side.
- The Codex integration must remain an implementation detail of the built-in `codex` agent plugin. New agent CLIs should be added by creating additional plugins instead of hardcoding new CLI logic into the HTTP/API layer.
- Deleting one conversation turn from a session should remove the selected user/assistant/error message group from `messages.jsonl` so future context is shorter. Run metadata and recorded artifacts may remain under the session's `runs/` directory.
- Child notebooks are limited to one level: root notebook -> child notebook. Deeper nesting is rejected.
- Session question/answer messages are stored in `data/notebooks/<notebook>/sessions/<session>/messages.jsonl`; run metadata is stored under the same session's `runs/` directory.
- If the trash contains a deleted `Default notebook`, startup should not recreate another Default notebook merely because the root notebook list is empty.

## Conflict Policy

Before implementing future changes, compare the request against this file and `docs/design.md`.

If a request conflicts with an existing requirement or design decision, ask for clarification instead of silently changing behavior.


<!-- /message -->
<!-- message:id=xtool_design role=assistant createdAt=2026-05-18T06%3A36%3A29.051Z -->

## Assistant

# AIAssitant Design

Last updated: 2026-05-18

## Architecture

The app is a dependency-free Node.js HTTPS server plus a static single-page frontend.

- `server.js` serves HTTPS, authentication, APIs, file persistence, and generic agent run orchestration.
- `public/` contains the browser UI.
- `public/favicon.svg` is an SVG favicon used by the browser tab/bookmark UI and the in-app brand mark. It is intentionally served without authentication because it contains no user data and lets the login page show the browser icon.
- `agent-plugins/` contains built-in agent CLI adapters. The current built-in adapter is `codex`.
- `data/` contains all mutable state.
- Agent requests are routed to the selected plugin. The current `codex` plugin executes `codex exec`.
- Runs are asynchronous: message creation returns a run id, and the frontend polls for completion.
- The frontend uses responsive CSS media queries for phone-size screens.
- The right-side detail panel can be collapsed client-side; this affects only layout, not persisted server data.

## Storage Layout

Storage is intentionally file-based and human-readable.

`​``text
data/
  auth.json
  initial-password.txt
  audit/
    YYYY-MM-DD.log
  uploads/
    <timestamp>__<batch-id>/
  config/
    settings.json
  notebooks/
    <notebook-id>__<slug>/
      notebook.json
      notes.md
      notebooks/
        <child-notebook-id>__<slug>/
          notebook.json
          notes.md
          sessions/
            <session-id>__<slug>/
              session.json
              messages.jsonl
      sessions/
        <session-id>__<slug>/
          session.json
          messages.jsonl
          runs/
            <run-id>.json
  corrupt/
    ...
  trash/
    notebooks/
      <timestamp>__<notebook-id>__<slug>/
    sessions/
      <timestamp>__<session-id>__<slug>/
`​``

`data/store.json` is a legacy aggregate store. On startup the server migrates it to the directory layout if the directory layout does not exist. The legacy file is preserved.

## Robustness Model

The service distinguishes core config from recoverable business data.

Core config:

- TLS cert/key files.
- Auth config in `data/auth.json`, unless bootstrapping from `WEB_PASSWORD`.

Recoverable data:

- Global settings.
- Notebook metadata.
- Notebook notes.
- Session metadata.
- Message logs.
- Run metadata.

If recoverable data is corrupt:

1. Preserve the original corrupt file or directory.
2. Create a readable replacement with recovered metadata where possible.
3. Continue startup.
4. Expose the recovered object in the UI rather than failing the whole service.

## Authentication

Login uses a server-side password hash in `data/auth.json`.

- Password hash uses Node `crypto.scryptSync`.
- Login issues an `HttpOnly`, `Secure`, `SameSite=Strict` cookie.
- All APIs require authentication.
- The main app redirects unauthenticated users to `/login`.
- Login failures are rate-limited per source IP in memory.
- Audit logs are append-only JSONL files in `data/audit/YYYY-MM-DD.log`.

## Audit Events

The server records security and operational events that matter for a private web agent:

- `login_success`
- `login_failure`
- `login_rate_limited`
- `logout`
- `unauthenticated`
- `settings_update`
- `notebook_create`
- `notebook_update`
- `session_create`
- `session_update`
- `message_create`
- `message_group_delete`
- `run_cancel`
- `notebook_trash`
- `session_trash`
- `trash_delete`
- `tls_client_error`
- `client_error`
- `answer_download`
- `artifact_download`
- `upload_create`
- `server_error`

Audit logs intentionally avoid storing passwords and message content.

## Trash

Delete actions are soft deletes:

- Deleting a notebook moves the whole notebook directory from `data/notebooks/` to `data/trash/notebooks/`.
- Deleting a session moves only that session directory from its notebook's `sessions/` directory to `data/trash/sessions/`.
- Each trash entry includes `trash.json` with deletion metadata and original path.
- Permanent delete removes only directories already under `data/trash/`.
- There is no restore API yet.

## Agent Plugin Architecture

Agent CLI access is intentionally separated from the HTTP/API layer.

- `agent-plugins/index.js` registers available plugins and exposes their public metadata to the UI.
- Each plugin owns CLI-specific prompt construction, argument construction, executable selection, allowed config override keys, default model/mode metadata, and empty-output wording.
- `server.js` owns shared safety and persistence behavior: authentication, rate limiting, audit logs, run lifecycle, timeout, process termination, workdir allowlist, artifact detection, and file downloads.
- Current plugin: `codex`, implemented in `agent-plugins/codex.js`.

Adding another agent CLI should add a new plugin module and register it in `agent-plugins/index.js`. It should not duplicate authentication, notebook storage, audit logging, trash, upload, or download logic.

## Agent Execution Safety

The browser cannot set arbitrary agent execution policy.

Allowed sandbox values:

- `read-only`
- `workspace-write`

Working directories must be inside `WEB_WORKDIR_ALLOWLIST`, defaulting to the app root.

Allowed config override keys:

- `model_reasoning_effort`
- `model_reasoning_summary`
- `model_verbosity`

Notebook notes and prior messages are inserted into prompts as untrusted context.

Default model/mode configuration:

- agent: `codex`
- model: `gpt-5.5`
- mode: `high`
- Codex plugin mapping: model is passed to Codex CLI as `-m <model>`, while mode is passed as `-c model_reasoning_effort="<mode>"`.

## Data Model

Notebook:

- `id`
- `title`
- `createdAt`
- `updatedAt`
- `notes.md`
- sessions directory

Session:

- `id`
- `title`
- `createdAt`
- `updatedAt`
- `settings`
- `messages.jsonl`
- runs directory

Question/answer storage:

- The conversation transcript for a session is `messages.jsonl`.
- For a root notebook, the path is `data/notebooks/<notebook-id>__<slug>/sessions/<session-id>__<slug>/messages.jsonl`.
- For a child notebook, the path is `data/notebooks/<parent-id>__<slug>/notebooks/<child-id>__<slug>/sessions/<session-id>__<slug>/messages.jsonl`.
- Each line is one JSON message record.
- Run metadata and artifact references are stored separately under the same session's `runs/` directory.

Child notebooks:

- Only one child level is supported.
- Child notebook directories live under their parent notebook's `notebooks/` directory.
- The public state returns notebooks as a tree with `children`, `parentId`, and `depth`.
- A child notebook can have its own sessions, notes, and runs.
- Deleting a parent notebook moves the parent directory, including child notebooks, to trash.
- Creating a child under another child is rejected.
- If a deleted `Default notebook` is already present in trash, startup does not recreate it merely because no root notebooks remain; the service still creates/maintains the automatic `XTool研发` notebook.

Message:

- `id`
- `role`
- `content`
- `createdAt`
- optional `meta`

Message group deletion:

- Endpoint: `DELETE /api/sessions/:sessionId/message-groups/:messageId`
- The server finds the selected message, walks backward to the nearest user message when possible, then removes that user message and all following non-user messages until the next user message.
- If the selected message has no earlier user message, only the selected non-user group is removed.
- The session must not have a running agent task.
- The operation rewrites `messages.jsonl`, updates session/notebook timestamps, and records `message_group_delete` in the audit log without message content.
- Run metadata and artifact records are not deleted by this operation.

Run:

- `id`
- `notebookId`
- `sessionId`
- `userMessageId`
- `status`
- `startedAt`
- `finishedAt`
- `agent`
- `agentLabel`
- `settings`
- stdout/stderr tails
- final message
- artifacts: files created or modified under the allowed workdir during the run

## Downloads

The service supports two scoped download types:

- Session answer download: `GET /api/sessions/:sessionId/messages/:messageId/download`
- Run artifact download: `GET /api/runs/:runId/artifacts/:artifactId/download`

The service does not expose a generic server file browser. Run artifacts are captured by comparing a file snapshot of the run workdir before and after agent execution. Downloads are only allowed for artifact ids recorded in that run metadata.

## Project Sync Notebook

The server maintains a normal notebook named `XTool研发`.

- It contains a session named `需求与设计同步`.
- The session is regenerated from current project documents and implementation history when the server starts and when this feature is changed.
- It includes `docs/requirements.md` and `docs/design.md` as assistant messages so they are visible and downloadable in the web UI.

## Uploads

Uploads are independent from sessions.

- Endpoint: `POST /api/uploads`
- Storage: `data/uploads/<timestamp>__<batch-id>/`
- Directory uploads preserve client relative paths when the browser provides them.
- Uploaded files do not automatically become notebook notes, messages, agent prompt context, or run artifacts.

## Change Discipline

Every meaningful behavior/storage/API change must update:

- `docs/requirements.md` if user-visible requirements changed.
- `docs/design.md` if architecture, persistence, security, or API behavior changed.

If implementation pressure conflicts with these documents, the documents win until the user explicitly changes the requirement.


<!-- /message -->
