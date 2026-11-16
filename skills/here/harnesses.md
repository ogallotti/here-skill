# Harness notes for /here

The skill needs five capabilities. Read the section for your harness; each maps them to its tools.

1. **Owner messages**: tell messages the owner typed from messages agents, schedules or delegates sent.
2. **Threads**: list threads in scope with their status, and read their history since a date.
3. **Search**: find the marks `Decided for the owner:` and `Waiting on the owner:` across threads.
4. **Page**: show an HTML page.
5. **Relay**: send a decision to the thread that owns the work.

## T3 Code

The skill was built here, and every capability exists.

1. `t3_thread_read`: owner messages are `user_message` items with `createdBy: "user"`. `createdBy: "agent"` items are relays from other threads, schedules or delegates. To find the anchor in a long thread, read one item to learn `itemCount`, then page from `afterPosition` near the end.
2. `t3_thread_list` with `projectId` and `includeSubagents: true`. In a thread scope, keep the threads whose `parentThreadId` is this thread, plus the threads this one messaged (its `t3_thread_send` calls in `view: "activity"`) and the senders of the relays it received. Source 1 is `statuses: ["waiting"]` (pending approvals and questions) plus `snoozed: true`. `list_scheduled_tasks` covers scheduled jobs.
3. `t3_thread_search`. It skips subagent threads.
4. `html_preview`, then `html_render` with the preview's `contentHeight`. The page picks up the T3 theme on its own.
5. `t3_thread_send` with mode `auto`: it steers a running turn, where `queue` would wait for that turn to end.

## Claude Code

Each session is a thread. Its transcript is a JSONL file under `~/.claude/projects/<cwd with / replaced by ->/`.

1. Owner messages are `type: "user"` entries whose `message.content` is text. Entries holding a `tool_result` are tool output, and `isMeta: true` entries are injected context.
2. The project's threads are the transcripts in its folder, ordered by modification time. Subagent work appears as `isSidechain: true` entries. A thread scope is this session's transcript and its sidechains.
3. `grep` the marks across the folder.
4. Write the page to a file and open it (`open` on macOS, `xdg-open` on Linux), or give its path.
5. No session can message another. Apply the decision yourself, or tell the owner which session to resume (`claude --resume <session-id>`) and the exact message to send.

## Codex

Each session is a thread. Its rollout is a JSONL file under `~/.codex/sessions/YYYY/MM/DD/`, and the `session_meta` line holds its `cwd`.

1. Owner messages are `response_item` entries with `payload.role: "user"`. The first of them in a session carry injected instructions and environment context, not the owner's words.
2. The project's threads are the rollouts whose `session_meta.cwd` is inside the project. A thread scope is this session's rollout.
3. `grep` the marks across the rollouts.
4. Write the page to a file and open it, or give its path.
5. Apply the decision yourself, or resume the owning session with the message: `codex exec resume <session-id> "<message>"`.

## Other harnesses

Map the five capabilities to what yours offers. When one is missing:
1. Without sender information, ask the owner when they were last here.
2. Without a thread list, cover this thread plus the repositories, the tracker and the schedules.
3. Without search, rely on reading the threads.
4. Without inline HTML, use a file or Markdown.
5. Without relay, apply the decision yourself or tell the owner which thread to resume with which message.
