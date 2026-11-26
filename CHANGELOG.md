# Changelog

## 1.37.0

- **Background Sessions.** `/background <task>` (or **Coda: Run In Background**) runs an agent in its own git worktree, in this window, while you keep working; several run at once. The strip above the input shows each run's steps and current action, and when one finishes you can **Review** its report and diff, **Apply** its changes to your checkout as uncommitted changes (three-way merged), **Open** the worktree in a window, or **Discard** it. Before, "run in background" only created the worktree and opened it in a new window for you to drive by hand.
- **Isolated Runs Are Auto-Approved.** Inside a worktree, edits and commands run without cards; deletes, pushes and pull requests still ask. The pending-changes bar lists only files in your open folders, so a background run's edits never mix with yours.

## 1.36.0

- **A Browser With No Setup.** `browserAction` now drives a real Google Chrome through Chrome DevTools MCP, started on demand with `npx` in an isolated profile after you confirm, instead of asking you to install Playwright in your project. The agent lists the browser's tools, opens pages, reads accessibility snapshots, clicks and fills by element id, reads console and network output, and saves screenshots under `.coda/attachments/`. `coda.browser.mcpCommand` changes the server or adds `--headless`. The server's own usage statistics are turned off.
- **MCP Prompts and Resources in the Panel.** Prompt templates from connected servers are slash commands, `/mcp.<server>.<prompt> [args]`, and **Coda: Attach MCP Resource** attaches a server resource to your next message. Both update as servers connect.
- **OpenTelemetry Export, Opt In.** `coda.telemetry.otlpEndpoint` sends one trace per run (run, step and tool spans with provider, model, tokens, durations, outcomes; no prompts or file contents) over OTLP/HTTP JSON to a collector you name. Empty by default; Coda still has no telemetry of its own.
- **Edit Tools, Measured.** `npm run report:edit-tools` tallies edit-tool calls and errors per model from recorded live-eval runs. From 310 recorded tasks: `editFile` is 532 of 545 edit calls, `applyPatch` and `applyDiff` were never called, so the four edit paths are not being confused; `createFile` fails 13–15% of the time on two models and is the thing to look at next. See `docs/EDIT_TOOL_REPORT.md`.

## 1.35.0

- **Fill-in-the-Middle Inline Completions.** On Mistral Codestral, DeepSeek and Ollama, inline completions now use the provider's FIM endpoint (`/v1/fim/completions`, `/beta/completions`, `/api/generate` with `suffix`) instead of a chat prompt: the prefix and suffix are sent as fields and only the gap comes back. A model that refuses FIM falls back to the chat prompt for the session. `coda.inlineCompletions.nativeFim` turns it off.
- **The Model Provider's Own Web Search.** `coda.webSearch.provider: "model"` gives agent runs on Anthropic the `web_search` and `web_fetch` tools, and runs on OpenAI (Responses) the `web_search` tool, run on the provider's servers with citations and billed by them; other providers fall back to DuckDuckGo. Gemini's search cannot yet be combined with Coda's tools outside a preview mode, so it is not included.
- **Claude's Context Editing and Memory Tool (opt in).** `coda.anthropic.contextEditing` asks Anthropic to clear the oldest tool results server-side once a request passes 100k input tokens (newest five kept, the memory tool never cleared), and gives Claude its memory tool over `.coda/memories/`. Coda's own token estimate accounts for what the server will clear, so its compaction stays a backstop instead of firing first. Off by default until a before/after on the eval suite with an Anthropic key; the logs show what each step cleared.
- **Live Capability Probes Feed the Matrix.** `npm run probe:capabilities -- --provider <id> --model <id> --api-key-file <file> --write` runs four real requests (tool calling, parallel calls, forced choice, streaming calls) and records the result in the provider capability matrix, where a verified yes upgrades the hand-written row and a no never downgrades it. First row: NVIDIA, verified with `poolside/laguna-xs-2.1`.

## 1.34.0

- **Remote MCP Servers.** A server entry with a `url` connects over MCP's Streamable HTTP transport, so hosted servers (GitHub, Sentry, Linear, Atlassian and others) work alongside local `command` servers. When a server requires sign-in, Coda runs the MCP authorization flow (OAuth 2.1 with PKCE, metadata discovery, dynamic client registration) through your browser and keeps the token in VS Code's SecretStorage. Dropped sessions reconnect on their own. **Coda: MCP Marketplace** gains **Connect remote server**.
- **One MCP Config for Every Agent.** Besides `.coda/mcp.json`, Coda now reads `.mcp.json` (Claude Code) and `.vscode/mcp.json` (VS Code) from the workspace, and plugins may bring remote servers too. Header values accept `${env:VAR}`.
- **Servers Can Ask You Questions.** MCP elicitation requests appear as VS Code input boxes; `ping` and `roots/list` are answered. Prompt templates and resources are read from servers that offer them (the panel does not list them yet).
- **Hooks for the Whole Run.** Beyond `pre-tool` and `post-tool`, hooks can run on `post-tool-failure`, `session-start`, `user-prompt-submitted`, `pre-compact`, `subagent-start`, `subagent-stop`, `agent-stop` and `session-end`. Claude Code and VS Code event names are accepted, each hook gets the event's JSON on stdin and in `CODA_HOOK_PAYLOAD`, a prompt hook can stop a run, and what a session-start or prompt hook prints becomes context for the model. Hook files in `.coda/hooks/`, `.github/hooks/` and `.claude/settings*.json` are read in trusted workspaces.
- **Quick Fixes from the Language Server.** New tools `getCodeActions` and `applyCodeAction` let the agent apply the fix VS Code offers for a diagnostic (a missing import, a missing member), and `organizeImports` sorts and prunes imports the way the language server does. The edits are tracked, so Undo and the pending-changes list cover them.

## 1.33.19

- **A Long File List Folds.** A run that changed many files added one full-width row per file to the chat, pushing everything else out of view. While Coda works, only the three newest files now show, under "N more files". When the run ends, a list of more than three files folds into one "N files changed +x −y" line that opens with a click. The pending-changes bar above the input still lists every file with Keep / Undo.

## 1.33.18

- **See What Coda Is Doing.** While Coda works, its newest six steps stay open, with older ones folded behind "N earlier steps". Steps still in progress (shown on the live status line) were taking places in that window while staying invisible, so often only two steps showed under a large "earlier steps" count. Now you see six real steps, and the count matches what is folded.
- **One Set of Numbers for Changed Files.** A run's file rows and summary line added up each edit (an edit that rewrote 35 lines to change three counted +35 −35), while the pending-changes list showed the real difference, so the same change read "+73 −64" in one place and "+19 −10" in another. Both now show the real difference, and a file edited several times is one row ("×3"). A new one-line file reads "+1", not "+1 −1".
- **Short Paths in Pending Changes.** In a project opened through a linked folder (including macOS temporary folders), the pending-changes list showed each file's full path instead of its place in the project.
- **No "Waiting for Approval" on a Finished Run.** A run whose approvals were answered without a click (a session approval, or the run ending) kept that warning on its summary.
- **"Destructive" Means Destructive.** Inline scripts (`node -e "…"`, `python -c "…"`) still ask before running, but no longer carry the red "Destructive" badge unless they delete something; a run's test scripts were all flagged "can delete data".
- **Test Results Without Readable Counts.** When a test run's output has no counts Coda can read, the results box says so and points to the raw output, instead of showing "FAILED · 0 passed · 0 failed".
- **Smaller Download.** An unused 1 MB image is no longer included in the extension.

## 1.33.17

- **A Clearer Extension Page.** The Marketplace page says what Coda does and how, instead of adjectives: providers are listed by name (all 37, grouped as in Coda's provider picker), the modes and defaults are described as they are today (Ask by default, Assisted, Auto-accept edits), and it points to these release notes.
- **A Rating Request, Rarely.** After a run in which the build or tests Coda ran passed, Coda may ask once whether you'd rate it: first on the third such run, and again only ten verified runs after "Later". "Rate Coda" or "Don't ask again" ends it for good. The link opens the store your editor installs from (the Marketplace in VS Code, Open VSX in Cursor, Windsurf or VSCodium).

## 1.33.16

**Review changes in one place.**

- **Pending Changes Above the Input.** Every file Coda changed that you have not reviewed yet is listed above the chat box ("3 files changed +61 −11"), across turns, until you keep or undo it. Keep or Undo one file or all of them; open a file or its diff. Run cards now offer Review only, so there is one place to decide. Changes made by shell commands are not listed (use Source Control for those), and the bar says so.
- **Review in the Editor.** Open a changed file to see each change highlighted with "Keep | Undo | 2 of 5 | ↑ | ↓" above it, and Previous, Next, Keep all and Undo all in the editor title bar. Undo refuses a file you edited after Coda did, so it never overwrites your work.
- **Staging Keeps Changes.** Staging a file in Source Control keeps its pending changes (only the staged ones, if you staged part of it).
- **Survives a Reload.** Pending changes are still listed after you reload the window, as long as the file still holds what Coda wrote.

**Approvals that ask less, and say more.**

- **Assisted Permissions.** A new level between Ask and Auto-accept edits: your model rates each action that would show a card, and low-risk ones run without asking, noted as "Auto-approved: low risk — reason". High-risk, unclear or slow ratings still show the card. Destructive commands, deletes, push/PR and MCP tools are never auto-approved. It costs one short model request per rated action.
- **Remember for This Session, Workspace or Always.** The card's checkbox, which silently meant "always, in this workspace", is replaced by a choice. Command cards also offer "Allow all commands in this session" (destructive commands still ask). `/autoApprove` and `/disableAutoApprove` turn approvals off and on for the current chat, with a warning. Session approvals come back when you resume that chat after a reload.
- **Edit Cards Stay Small.** Each changed file is one row (path and +/− counts); click it to see the diff, or open the change side by side in VS Code's diff editor. Only destructive actions are tinted.
- **Security Fixes.** A remembered command matched as a wildcard: remembering `ls *.ts` also allowed `ls; rm -rf src .ts`. It now matches exactly. `git reset --hard` and `git clean` are treated as destructive.
- **Per-Folder Permission Settings.** In a multi-root workspace, a permission mode set on one folder applies to runs in that folder, and the panel switch saves to the level where your setting lives.

**New ways to work.**

- **Quick Chat** (`Coda: Quick Chat`, ⇧⌥⌘L / Ctrl+Shift+Alt+L when Copilot chat is off): ask a quick read-only question about the code at your cursor; the answer appears beside the editor, and "Open in Chat" continues it in the panel.
- **Compare Models** (`/compare`, or "Compare models…" in the model menu): one question on 2–4 of your configured models, side by side, with time, tokens and cost for each. Keep the answer you prefer and the conversation continues on that model. Comparisons are read-only.
- **Customizations Screen** (`Coda: Open Customizations`, or the gear menu): instructions, skills, agents, slash commands, hooks, MCP servers and plugins in one place, each with where it comes from, and open, create, enable/disable and remove.
- **Plugins.** Install bundles of skills, agents, slash commands, hooks and MCP servers from a folder or a git URL (Copilot's agent-plugin format). Nothing a plugin adds runs until you trust it, and it switches off again if its hooks or servers change. Workspace `.coda/commands/<name>.md` files now define new slash commands.

**Polish.**

- **Readable in Light Themes.** Secondary text, counts, status chips, error and warning text and code-block icons now meet WCAG contrast in light and dark themes (Coda's UI check measured 3,503 elements with no failures).
- **Run Details Button.** The header's unlabeled yellow number is gone; the button that opens run details has a real icon and says what it does.
- **Screen Readers Hear Stops.** Stopping a run is announced once ("Run stopped"), including when an approval card is withdrawn because the run ended.
- **Approval Notes Wrap.** "Auto-approved: low risk — reason" shows the whole reason instead of cutting it off, and no longer repeats as the live status line while a card waits.

## 1.33.15

- **The Agent Is Told When It Goes Round in Circles.** When an edit put a file back the way it was a few edits earlier, or the agent kept alternating between two files, Coda showed an "Edit thrashing detected" warning in the panel but never told the agent, which carried on undoing and redoing. The edit's own result now tells the agent to re-read the original error and try a different approach before editing again; if the same file keeps flipping back, it is told to stop and explain what it tried.
- **The Thrashing Warning Shows Once.** After one detection, every later edit in the session showed "Edit thrashing detected" again, even healthy ones, and Coda's automatic repair stayed switched off for the rest of the session. The warning now appears only on the edit that thrashed, and repair pauses only while the thrashing is recent.

## 1.33.14

**Your mode decides what a run may do; Coda no longer guesses first.**

- **No Extra Model Call Before Each Run.** Coda used to ask the model to classify every request (question, fix, review…) before starting, which added a model call and up to a minute before the first action, and sometimes guessed wrong. That step is gone: the run starts straight away, and what it may do comes from the mode and permissions you chose. The `coda.agent.intentClassifier` and `coda.agent.toolSurface` settings are removed.
- **Ask Is the Default Permission Mode.** Reads and your project's own checks (tests, build, lint, type-check) run freely. Every file edit, and every shell command that can change files (`sed -i`, `npm install`, `prettier --write`, `find -delete`…), shows an approval card. **Auto-accept edits** applies edits without a card; risky commands still ask. **Full auto** needs a sandbox, which Coda does not have yet, so it runs as Auto-accept edits and says so. Plan, Explore and `/review` cannot change your code at all; Plan may still save its plan under `.coda/plans/`.
- **A Declined Edit Becomes a Patch.** When you decline an edit, the agent finishes by giving the change as a patch, and the answer offers **Apply patch** and **Open as diff**. A run with nobody to approve (a headless or scheduled run) treats approvals as declined and carries on instead of stopping.
- **Changes Are Checked Because They Happened.** When a run changes files, Coda checks the editor's errors in those files and runs the project's build and tests before finishing, and sends failures back to the agent to fix. A run that changed nothing is not checked; documentation-only changes skip the build and tests; changes made from the shell or by sub-agents are checked like edits. A failed check is no longer dropped if the run ends some other way, and changes left unchecked by an early stop are reported.
- **About 15 Tools to Choose From.** Each run sees one fixed core of tools; the rest are a search away (`searchTools`), and the search offers only tools the run can actually use.

**Approvals you can trust.**

- **Enter Does What the Focused Button Says.** Pressing Enter with Deny focused used to approve the action. Shortcuts now act only inside the approval card, focus moves to the card when it appears (not while you are typing), and screen readers announce it.
- **One Card at a Time.** When several approvals are waiting (for example from parallel sub-agents), they queue as "1 of 3" and each says who is asking. Sub-agents' cards now reach your panel, and a sub-agent no longer times out while you decide.
- **See What Will Change.** Edit cards show a real diff of the file with its path and line counts, for every edit tool. Commands are shown in full with their working directory; destructive ones carry a red "Destructive" badge. Deny is one button; the reason and "Deny and stop run" appear after it.

**A panel that tells the truth.**

- **Answers Can No Longer Plant Buttons.** Text in an answer could create a working "Run in terminal" button. Answers are now sanitized, links are limited to web, mail and relative addresses, and "Paste into terminal" no longer presses Enter for you.
- **Status Matches What Is Happening.** A green run no longer shows "1 failed" from an earlier test run; a "Tests passed" chip shows when it verified. An unrelated error (such as a file preview failing) no longer marks the run as failed. Opening a past conversation no longer shows "Working", and a finished run's spinner can no longer come back. Tool results land on their own row, and every row ends as done, failed or stopped.
- **Reload Without Losing the Run.** Reloading the panel mid-run brings back the run's progress and any open approval or question, so you can still answer it.
- **Undo Means This Run.** Undo on a run's card reverts only that run's files, and Keep/Undo show what they did ("Reverted 3 files").
- **Readable Failures.** A failed check names the command and its first error, with the full output a click away; retries read "Fixing, attempt 1/2".
- **The Answer Appears Once,** and tool rows read the same way throughout ("Ran `npm test` · failed", "Edited `src/a.ts` +8 −8"), with aligned line counts.

**Reading and writing.**

- **Streaming Holds Still.** Code blocks grow in place without jumping or losing your scroll, one-line commands keep their Copy and Run buttons, and the toolbar no longer covers code. Scrolling up while an answer streams stops the auto-scroll and shows "Jump to latest"; finishing no longer yanks you down. The finished answer always matches what a reload would show.
- **Thinking Streams Live,** in a collapsed row showing its latest line. File paths in answers (`src/a.ts:12`) open the file.
- **One Place for Permissions.** The panel's mode menu has Mode (Chat / Agent / Plan) and Permissions (Ask / Auto-accept edits), saved where your setting lives; the status bar shows the same. The old "Tools" picker is gone.
- **Composer Fixes.** Escape closes a menu without stopping the run. Enter while composing Japanese, Chinese or Korean text no longer sends. @-mentions stay where you typed them. Stop puts queued messages back in the box; New chat during a run asks for a second press. Deleting a conversation can be undone.
- **Keyboard and Screen Readers.** The mode menu, history list and approval cards work from the keyboard; Tab no longer switches modes; new answers and run results are announced; motion respects the reduced-motion setting.

## 1.33.13

Checked against each provider's official documentation.

- **Claude's Newest Models No Longer Refuse Agent Runs.** Claude Fable 5.x, Mythos 5.x, Opus 5.5, Sonnet 5.5 and Haiku 5.5 check that nothing earlier in a conversation changed after they reasoned about it; Fable 5.2, Mythos 5.2 and Mythos 5.1 refuse the request otherwise. Coda changed its tool list and instructions between the first and second step of most agent runs, so those runs could fail on step two. The first step now keeps the run's tools and instructions. When Coda must shorten earlier context to stay within the window, it asks Claude to set aside the reasoning that no longer applies instead of failing, and says so in the run.
- **DeepSeek Works Again.** The default model, `deepseek-chat`, was retired by DeepSeek on 2026-07-24; the default is now `deepseek-flash`. A second agent turn on DeepSeek was refused because DeepSeek requires earlier answers' reasoning to be sent back when tools are in use; Coda now keeps it. The thinking level now controls DeepSeek's thinking (on/off and low, high or max).
- **Ollama Agent Runs Work on Every Ollama Version.** Coda talked to Ollama through its newer Responses endpoint, which needs Ollama 0.13.3 or later and cannot follow up on earlier steps the way Coda asked it to. Coda now uses Ollama's long-standing Chat endpoint.
- **The Thinking Setting Reaches Each Provider in the Form It Accepts.** Groq, xAI and Mistral now receive the level in their own setting (before, it was silently ignored), and only for the models each documents; xAI's reasoning cannot be turned off, so "None" sends nothing there. NVIDIA (Qwen 3.5, Nemotron 3 Super, GPT-OSS, DeepSeek V4 Pro), Cerebras, Together, Fireworks, DeepInfra, Alibaba and Ollama models get their documented control; OpenRouter follows each model's own catalog entry. Gemini 2.5 Pro, which cannot turn thinking off, gets its minimum budget instead of an invalid zero. A provider that still refuses the setting gets the request again without it, once, and the refusal is remembered for a week. Other models are sent nothing and use their own default.
- **Thinking Shows in Chat, and Tagged Thinking Stays Out of the Answer.** Chat answers show "Thinking… Ns" and "Thought for Ns" like agent runs. Thinking that a host writes into the answer between `<think>` tags is shown as thinking instead of raw text, and is no longer kept in the history. Thinking rows can be opened to read the reasoning (collapsed by default; `coda.chatUI.showThinking` turns this off). Rows that cannot be opened no longer show a chevron.
- **Questions and Sub-Agents Get the Time They Need.** A question Coda asks you waits as long as you take, and pressing Stop now ends it. A sub-agent may run up to `coda.subAgents.maxMinutes` (60 by default); a 30-minute limit used to stop it first.
- **Models That Only Stream Can Be Used in Agent Mode.** The tool-calling check now streams, like every real run; some Alibaba models refuse non-streamed calls and were wrongly kept out of Agent mode.
- **Context Size Is No Longer Mistaken on Long Runs.** On a host that counts tokens differently from Coda, two similar token reports late in a long run could teach Coda a far too small context window. A window is now learned only when the reported count stops growing while the prompt grows, which is what a server that cuts prompts does.
- **"Working" Clears When a Chat Answer Ends.** The status strip kept showing "Working" after a chat answer was complete.

## 1.33.12

- **No More "Context Updated" Forever.** On a model Coda had no context size for (many NVIDIA models), one reply that reported fewer tokens than Coda had estimated was taken as proof that the model could only hold about 4,600 tokens. Every later step then looked too big, so Coda compacted before each one, dropped what the agent had just read, and the run never got anywhere. Coda now lowers a model's window this way only when two replies in a row agree on the same cap, and drops that limit as soon as the model accepts more.
- **A "Context Updated" Row Means Context Was Freed.** Past half the window, every step showed a compaction row even when nothing was removed, so a long run showed one after every tool call. The row now appears only when the context actually got smaller; on a long test run, 20 rows became 4.
- **Long Runs Keep Their Memory When Compacting.** When a run reached the limit, Coda deleted every tool call except the last few in one go (about 30,000 tokens down to 4,000 on a test run), so the agent forgot what it had done and repeated work. Now the file contents the agent wrote in older calls are shortened to a note pointing at the file, with the call itself kept. If that is not enough, the oldest steps go one at a time, each leaving a line such as `runCommand npm test: failed` in an "earlier in this run" record. Compaction aims for about half the window, so the run has room to work before the next one.

## 1.33.11

- **Approvals Wait for You.** A permission card left unanswered for about three minutes made Coda report "Connection to the model lost" and restart the step; after two of those the run failed. Time spent waiting for your answer no longer counts against the run, however long you take.
- **Long Commands No Longer Look Like a Dropped Connection.** A command that ran for more than three minutes, such as a long test suite, was treated the same way: the run reported a lost connection while the command was still working. Only real silence from the model now counts.
- **No 30-Minute Limit on a Run.** A run still working after 30 minutes was stopped with "The run reached its overall time limit", and never resumed. A run now ends when the work is done, when you stop it, or when its context cannot be recovered; a model that stops responding is still caught and the run resumed. Chat answers lose their 5-minute limit in the same way.
- **Long Runs Recover More Than Twice.** A run could resume after a dropped connection or a full context only twice in its whole life, so a long run on a flaky provider failed on its third drop, hours later. The allowance now refills after five completed tool calls; a run that keeps failing without progress still stops quickly.

## 1.33.10

- **Answers Look the Same While They Stream.** While Coda was writing an answer, lists had no bullets or indentation, inline code had no background and headings blended into the text; the formatting appeared only when the run ended and the answer was drawn again. The streaming text now uses the answer's own styles, so nothing changes when it lands. The finished answer is also shown in the full text colour instead of the dimmer secondary colour, matching how it streamed.
- **Lists Stream Correctly.** A list with blank lines between its items was cut apart while it streamed: nested bullets indented four spaces showed as a code block, nested bullets indented three spaces lost their nesting, and a numbered list became one list per item. The agent's notes between actions kept the broken version for good. Lists now stay whole until they end.
- **No Raw Markdown While Streaming.** Markers such as `**`, `~~`, `](` and `&amp;` showed as text mid-answer until the next line break, and a half-written bold phrase or link URL showed raw until it was finished. While text streams, an unfinished bold, strike or code span is shown closed and a link shows its text while the URL arrives; the finished answer is always rendered exactly as written.

## 1.33.9

- **Sub-Agent Cards Match the Rest of the Panel.** The cards were outlined boxes with a yellow outline while running. They now use the panel's borderless, softly lifted card style, with the state shown by the icon and a faint tint. The Stop button stays quiet until you point at it. In high-contrast themes every card (including the activity, approval and permission cards, which had no edge there before) gets the theme's contrast border.

## 1.33.8

- **The Agent Can Actually Use Sub-Agents.** The tool that starts a sub-agent was not offered to the agent in any mode: the model would have had to look it up by name first, and in recorded runs it never did. It is now available in every mode, and the agent's instructions say when delegating pays: research across many files, independent parts of a task, and long side work.
- **Sub-Agents Run at the Same Time, Edits Included.** Sub-agents that edit files used to take turns. They now run together. A file-changing tool call holds a lock on the files it touches, so two sub-agents editing the same file take turns on that file and neither change is lost; edits to different files, and commands, run in parallel.
- **Background Sub-Agents.** The agent can start a sub-agent in the background and keep working. The report arrives as a message when it is ready. If the agent finishes first, the run waits for the report and continues with it, so delegated work is never dropped.
- **Your Own Sub-Agents.** Markdown agent files in `.coda/agents/` or `.claude/agents/`, in the workspace or your home folder, define new sub-agent types with their own instructions, tools and model. The format is Claude Code's, so existing agent files work unchanged.
- **No More Five-Minute Cutoff.** A sub-agent was stopped after five minutes even while it was working. It now stops only after 10 minutes with no activity, or 60 minutes in total; a command that is still running counts as activity. Both limits are settings.
- **A Live Card for Each Sub-Agent.** Every sub-agent gets its own card in the panel, from the moment it is started. While it works, the card shows its name, its task, "Running · 1m 12s · 8 steps" and the step in progress. When it ends, the card says Done, Failed or Stopped (with the reason), and opening it shows the full task, its steps, and the report it gave the agent. Background sub-agents are marked as such. Cards stay visible while other steps scroll by, and after the run, so who did what is always one click away. Before, a sub-agent showed only a start row and an end row, mixed in with every other step.
- **Plan Mode Holds for Sub-Agents.** Under the plan or explore preset, a sub-agent could still be given editing tools. Sub-agents now inherit the parent's restriction and are read-only.
- **Recovery No Longer Replays Stale Results.** After Coda resumed from a dropped connection or a full context, repeating an earlier action returned its old result instead of running it: a file read after an edit showed the old content, and re-running the tests after a fix showed the old failure. Now only actions that change things are protected from running twice; reads and checks always run, and a command runs again once a file has changed.
- **Rate-Limit Errors Reach the Retry Logic Intact.** On a 429 or 403 response, Coda read the response body before the AI SDK could, so the provider's error message was lost and a rate-limited request could fail instead of being retried.
- **Sturdier MCP Servers.** A server that dies mid-run no longer raises an unhandled error, a server that fails to start is stopped instead of left running, images and resources from MCP tools are described instead of arriving as empty text, and tool schemas that omit `"type": "object"` keep their parameters.
- **Parallel Sub-Agents Cannot Edit the Wrong Lines.** A line-based edit refuses to apply if another agent changed the file since this agent read it, and asks it to re-read first. Before, the edit could land a few lines off after a parallel change.
- **A Step Budget for Sub-Agents.** After 40 tool calls (`coda.subAgents.maxSteps`), a sub-agent is told to stop and report what it has; if it keeps going, it is stopped and its partial report is kept.
- **Every Sub-Agent Report Is Capped.** Long reports from any sub-agent, not only explorers, are trimmed for the agent's context, with the full text saved to a file the agent can read.
- **Stop One Sub-Agent.** Each running sub-agent's card has a Stop button that ends that sub-agent only; the rest of the run carries on.
- **Credentials Kept Out of Logs.** Found by Coda's own agent auditing this codebase, then completed in review: log messages and error stacks are now redacted like the rest of a log entry; GitHub fine-grained, Slack and AWS temporary tokens, PGP private keys and credentials in URL query strings are recognized; and an MCP server's command line is logged with values after flags such as `--api-key` or `--token` hidden. Everything written to an MCP server's output channel, including its own error output, is redacted.
- **"Always Allow" Works for MCP Tools.** In the default permission mode, ticking "always allow" on an MCP tool's approval was saved but never used, so Coda asked again every time; allow and deny rules for MCP tools in `coda.permissions.rules` were ignored the same way. They now apply. MCP tools without a rule still ask, and tools that always ask, such as deleting a file or pushing, still ask whatever a rule says. Found by Coda's own agent reviewing this codebase.
- **Sub-Agents Start Faster.** Each sub-agent made a model call to classify its brief before starting, and a failed call ended it before it did any work. The brief is now classified locally.

- **Paste Files and Screenshots Into the Chat.** Pasting a file, or dropping one from outside VS Code, attached nothing: the chat panel is sandboxed, so it only received the file's name, and Coda looked for a file of that name in your workspace. The panel now reads the file and Coda saves it in `.coda/attachments/`, which has its own `.gitignore` so attachments are never committed. The chip shows the file's own name. Attachments older than 30 days are removed. The limit is 20 MB per file.
- **Images Reach the Model as Images.** A pasted screenshot or a mentioned `.png`, `.jpg`, `.gif` or `.webp` is sent to the model as an image when the model accepts images. Large screenshots are scaled down first. When a model is known to be text-only, Coda says the image was not sent and tells the model it exists. Before, a mentioned image was decoded into the prompt as text.
- **Files Outside the Workspace Can Be Attached.** A file picked with the attach button or dropped from outside the workspace became a `../..` path that never resolved. It is now copied into `.coda/attachments/` and attached.
- **Binary Files Are Named, Not Dumped.** Mentioning a binary file, such as a zip or a PDF, used to paste its bytes into the prompt as text. Coda now names the file and its size instead.
- **File Names With Spaces.** Attaching a workspace file whose path contains a space attached only the part before the space. Such paths are now quoted.

## 1.33.7

- **A Slow Language Server No Longer Stalls the Agent.** Go to definition, find references, symbols, hover, type hierarchy, available refactorings and rename planning waited on the language server with no time limit, so a server that was starting, indexing or hung froze the agent's step. Each lookup now gives up after 5 seconds and says so. When definition or references time out or find nothing, Coda falls back to the workspace symbol index and then a whole-word text search, labelled as a fallback that may include comments or other things with the same name. Applying a rename or refactoring is never cut off partway.

## 1.33.6

- **Long Test Suites No Longer Fail by Timeout.** Coda's own end-of-run test, build and lint checks were cut off after two minutes, even for suites that take longer. A timeout counted as a failure, so Coda then asked the model to fix it and ran the same check again, twice, which could add about six minutes and still end in a false failure. Checks now get ten minutes, and a check that runs out of time is reported as unverified with the reason, not retried. Test, build and lint commands the agent runs also get ten minutes by default.
- **"Tests Pass" Means the Whole Suite.** When related tests could be found, Coda ran only those, up to ten files, to confirm the tests pass, so a change could break the rest of the suite and still be reported as verified. That check now always runs the project's full test command. The quicker related-tests run counts only for the separate "relevant tests ran" check, and only Coda's own run can mark either one verified.
- **Faster Startup.** MCP servers now start at the same time and in the background, so one slow server no longer delays the others or holds up Coda's panel and commands. Workspace indexing no longer delays startup either.
- **Safer MCP and VS Code Tools.** Tools from MCP servers now follow what each server declares about them. A tool declared read-only can be used in read-only modes and plan mode; any other tool, including one that declares nothing, is blocked there. A tool that acts outside your machine, such as a deploy or opening a pull request, is gated like Coda's own push and pull-request tools. VS Code tools that Coda judges read-only by name stay available in read-only modes and plan mode. Reading through an external tool no longer counts as inspecting your workspace.
- **The Agent Is No Longer Told to Call Tools It Cannot Use.** The prompt listed every tool as available, including ones that were not loaded yet, and in the same breath listed those as "not loaded". It now lists exactly the tools the model can call, and separately the ones it must load first.
- **Large Tool Results Are Trimmed.** A tool result over 50KB now reaches the model as its beginning and end, with the full text saved to a file the model can read in parts. The project tree is capped at about 12,000 characters. Before, a large result was re-sent with every later step of the run.
- **More Reliable Recovery From a Full Context.** When a run recovers from exceeding the model's context limit, it keeps the tools the model was told it has, instead of dropping some of them.

## 1.33.5

- **The Coda Logo in the Sidebar.** The activity-bar icon is now drawn from the Coda logo: the wrapped head, the visor with its smiling eyes, and the ear pods. It fills its space like the built-in icons, so it no longer looks small beside them. The same mark appears in the panel header and beside each Coda reply.
- **No Raw Tags Under Answers.** Some models, Gemini among them, ended every reply with a `<finalReport><result>…</result>…` block copied from Coda's instructions. Final reports are now plain, short Markdown, and replies that already contain the block show without it.
- **Each Reply Reads as One Turn.** The list of files attached as context now sits under your message as "Added N files as context". The plan card belongs to the run that made it and folds away with the run's steps when it ends. Before, both appeared outside the reply, above the Coda label, and a later run could overwrite an earlier run's plan.
- **Review, Keep and Undo Are Visible.** After a run that changed files, these actions sit in one quiet row under the changed files. Before, they were inside the collapsed "Completed" section, so they only showed if you opened it. Changed-file rows no longer show how long each write took.
- **A Header That Says What Is Happening.** The header used to say "Ready" even while Coda was working or waiting for your approval. It now shows Working, Needs approval, Not configured, or indexing progress, and nothing when idle.
- **Clearer Errors.** An error now starts with a plain title, such as "This model is not available", "The API key was rejected" or "Could not reach the model provider", with the provider's message underneath. Where there is a fix, a Choose model or Change API key button sits beside Retry.
- **A Calmer Approval Prompt.** The reason box appears only when you ask for it. Allow once and Deny stay together on the right, and "Deny and stop run" is set apart on the left so it is hard to hit by accident. Button labels no longer wrap on narrow panels.
- **Less Clutter.** Each message is labelled once: the timeline dots and line are gone, and the model name is plain text. There is one Stop button, in the message box. Checkpoints are a thin divider. A running reply is labelled "Coda" from its first step. The start screen offers four starting points instead of ten cards, and the mode and model pickers sit together in the message box.

## 1.33.4

- **Runs Survive a Dropped Connection.** When the connection to the model dropped or stalled partway through a run ("terminated", a socket error, or a model that stopped streaming), the run failed and the work so far was lost. Coda now waits briefly and resumes from where it was, up to twice per run. Tools that already ran are never run a second time, and the panel shows "Connection to the model lost, resuming".
- **No More Silent Stops.** A provider error on a later step, such as a "404 Not Found" from NVIDIA in the middle of a run, ended the run with no explanation. It is now resumed the same way. An error that persists is reported with the provider's message.
- **Long Thinking Steps Are No Longer Cut Off.** Each model step had a fixed time limit, which stopped slow reasoning models in the middle of a good answer. The per-step limit is gone. Runs still stop if the model sends nothing for 90 seconds at the start of a reply, or goes silent for 3 minutes while streaming, and the overall run limit still applies.
- **Fewer "Run Stopped Before Completion" Endings.** Some models end a turn with no reply and no tool call partway through a task. Coda now asks the model to continue from its tool results, up to twice, before giving up.
- **Cleaner To-Do Lists.** The agent now sends its whole to-do list in one call, instead of one call per item, which could happen several times in parallel. Lists no longer gain duplicate items or more than one item in progress.

## 1.33.3

- **Long Runs Stay Readable.** During a run, the panel shows the six most recent steps, and older ones fold behind an "N earlier steps" line that opens them again. The agent's latest message stays visible above the steps it led to. Since 1.33.0, every step of a run had stayed on screen, so long runs became a growing log.
- **No More "No File Changes" After an Edit.** Every agent run added a second summary card saying "No file changes", even when files had changed. Each run now has one summary with the correct file count.
- **A Finished Turn Reads as One Message.** The finished-run row now sits under the "Coda" label, followed by the answer and then the files that changed, each with Undo and Open diff on hover. Previously the row sat under your message and the changed files sat between it and the answer.
- **File Names Fit.** Changed-file rows no longer cut paths short ("src/paginate.…") or hide them entirely on narrow panels.
- **No "Working" Chip After a Run Ends.** The status chip above the message box now clears when a run completes.
- **Quieter, Consistent Steps.** Edit and command rows are the same compact height as read rows, without divider lines. Steps that took under a second no longer show "0ms". A running command's output appears as a plain output area, with no second header or RUNNING badge. Failed commands show one terminal icon instead of two.
- **No Extra Card on Plain Answers.** Answers and failed runs no longer end with a "Completed · No file changes" card under the suggested follow-ups. A summary card still appears when there are changes to review, keep, or undo.
- **Tighter Messages.** Your messages no longer have a blank line under them, and messages with several paragraphs no longer show double gaps.
- **A Run With No Tool Calls Keeps Its Messages.** When a run ended without using any tools, the agent's messages and thinking rows from that run disappeared. They now stay.
- **Readable Message Icons in Light Themes.** The small icons beside "You" and "Coda" now have enough contrast in light themes.

## 1.33.2

- **Change Your API Key Any Time.** A key that still worked could not be replaced: `Coda: Configure` keeps a working key, and keys are kept in VS Code's secret storage, which Settings does not show. The new `Coda: Change API Key` command, also available from the gear menu as **Change API key**, replaces or removes the key for your provider. A new key is checked with the provider before it is saved, so a mistyped key never replaces a working one.

## 1.33.1

- **Successful Writes No Longer Show as Failed Edits.** Every file Coda created or edited successfully was also reported as "Edit to <file> failed (attempt 1)", with the success message as the reason. The files were written correctly; the label was wrong. Genuine edit failures are still reported with the tool's own error.
- **Coda's Server Token Is Kept Out of Git.** `gitCommit` with `all: true` could commit `.coda/server.token`, the local server's access token, along with other machine-local files. Coda now writes a `.coda/.gitignore` for its runtime files, once and without overwriting yours, and `gitCommit` never stages the token or port. Project configuration such as `.coda/mcp.json` can still be committed. If you committed these files with an earlier version, remove them from the repository; a new token is generated each time Coda starts.
- **Web Fetching Works Again.** `fetchUrl` and `docsLookup` failed on every website in current VS Code with `ERR_INVALID_IP_ADDRESS`. Requests now connect, and an address that resolves to a private network is still blocked, now checked across every address a name resolves to.
- **Documentation Lookup Reads the Docs.** `docsLookup` found a library but then requested it at a malformed address and got a 404. It now returns the documentation.
- **Folder-Wide Codemods Apply.** `codemodRun` with a folder as its `path` matched no files and reported "No matches". A folder now covers the source files inside it.
- **Failed Pushes Say Why.** A `gitPush` that failed (no remote, rejected, authentication) reported "Everything up-to-date." It now reports git's error, and an up-to-date push no longer counts as pushed.
- **One Row per Stretch of Thinking.** Back-to-back "Thought for Ns" rows merge into one row with the combined time.
- **No Sandbox Tool Until There Is a Sandbox.** The `sandboxRun` tool never ran anything, so it is removed, and the agent is told plainly that commands run directly on your machine without isolation.
- **Tool Conformance Check.** `npm run test:tools` runs each Coda tool for real in a throwaway workspace and checks its result and its effect on disk. This is a development tool and is not shipped in the extension.

## 1.33.0

- **Agent Runs Read in Order.** While the agent works, the panel shows what it says, what it thinks, and what it does in the order it happened, so each message sits directly above the actions it led to. Messages dim once the agent acts on them. One status line at the bottom shows the current step, how long it has run, and a Stop button, replacing the stacked activity box. When the run ends, the work folds into a single row and only the answer stays open below it. Previously, messages the agent wrote mid-run disappeared when the final answer replaced them.
- **Thinking Rows.** Models that report their reasoning show "Thinking… 3s" while they reason and "Thought for Ns" afterward. The reasoning text itself does not reach the chat panel unless you turn on the new `coda.chatUI.showThinking` setting (off by default), which lets those rows expand.
- **A Clearer Finished-Run Row.** The collapsed row is one line: time, tool calls, and a warning only when something is still failing, such as "⚠ 2 failed". Failures the agent later redid successfully count as recovered and no longer turn the row amber. Opening the row lists unresolved failures first, then all activity. The conflicting "N operations" count is gone.
- **One Line at Every Width.** On narrow panels the finished-run row drops whole items instead of wrapping, and "Read N files" rows shorten file names with an ellipsis while the count and "+N" stay visible. Hovering either shows the full text.
- **A Failed Last Step Stays Marked as Failed.** The last step of a run that ended on an error is no longer repainted as a success when the run finishes.
- **A Run Without an Answer Keeps Its Last Message.** If a run fails or is stopped before it answers, the agent's last message stays visible below the folded row instead of being hidden inside it.

## 1.32.0

Includes every fix listed under 1.31.1, which was not published separately, among them the conversation history fix for 1.31.0.

- **Context Limits Are Discovered, Not Assumed.** Unknown models now start in full mode instead of a reduced 16K guess, and the "assuming 16K" notification is gone. When a provider refuses a request as too large, Coda takes the limit it states, or steps down from what was refused if it states none, and retries in the same run. Limits a provider states are remembered for 30 days. A request refused on its answer size retries once with the real ceiling.
- **Overflow Rejections Are Recognized.** A plain `400` saying the context is too long now counts as an overflow. Previously only structured error codes did, so those rejections failed the run. Rate-limit and auth errors never count.
- **Recovery on the First Request.** A first request has no history to summarize, so a rejection there used to end the run. It now retries with a request shaped to the window it just learned.
- **Silent Truncation Detection.** If a server reports far fewer input tokens than were sent, that number is treated as the real window. This applies only to models nothing else lists, and lasts for the session.
- **Lean Requests on Every Model.** On each step, a repeated read replaces its earlier copy, a read goes stale once that file is edited, and large results older than the newest 12 shrink to one-line records. Tool calls stay on record, so the agent still knows what it did. On very large windows this may cause some extra re-reads.

## 1.31.1

- **Conversation History Fix.** In 1.31.0, trimming earlier turns to fit a small window could also overwrite them in the saved conversation. Turns left out of a request now stay in stored history. History already lost in 1.31.0 cannot be recovered.
- **The System Prompt Now Reaches the Model on Every Step.** The AI SDK treats per-step instructions as a replacement that carries forward, so agent runs were losing project rules (`AGENTS.md`), the execution contract, and the task brief, either from the first step or from step 1 on. Every step now sends the full prompt. This changes agent behavior on every provider.
- **Published Context Windows.** Models with no listed window now resolve through the models.dev catalog (a once-a-day download with no user data), per provider, before Coda falls back to a guess. A notification appears only if nothing lists the model, and points to `coda.contextWindowTokens`.
- **Runs Keep Their Tool History.** Auto-compaction now shrinks old tool results to one-line records and removes tool calls only when the provider would otherwise refuse the request. Budgets leave a small window real room, and count tool definitions accurately under Zod 4.
- **Compaction Summaries and Better Truncation.** Earlier turns are summarized once per run during compaction, and a slow or failed summary falls back to trimming. Truncated tool output now keeps its end, where failures and stack traces usually are.
- **Loop Protection.** A non-mutating tool call that keeps returning the same result gets a warning at the third repeat, and the run stops at the eighth. The same failed call repeated five times, such as calling an unavailable tool, also stops the run.
- **Honest Outcomes and Retries.** A final answer that says it could not proceed after tools succeeded gets one nudge, then reports `incomplete` with the text kept. The inspection retry now also covers unforced steps, and the first inspection tool fits the request and window (`gitStatus` for change questions) and stays callable afterward.
- **Duplicate Tool Calls on Retries.** Transcripts carried into retries listed earlier tool calls twice. Each call now appears once.
- **Token Usage on OpenAI-Compatible Providers.** Streaming runs now request usage, so budgets and the context meter use real counts. An endpoint that rejects the option gets the request again without it.
- **Lower Prompt-Cache Cost.** On Anthropic and Bedrock Claude, the cache breakpoint now moves with the run, and the system prompt is byte-identical from step 1 on.
- **Lighter Workspace State.** Conversations and checkpoints move from VS Code's state database to files under the extension's storage folder, migrating automatically. After this migration, an older Coda version will not see them. Capability probe results also persist across restarts (30 days for a pass, 1 day for a fail).

## 1.31.0

- **VS Code Tool Bridge.** Coda's agent can now call tools that other extensions and VS Code itself register through the Language Model API (`vscode.lm.tools`), executed with `lm.invokeTool`. The default is read-only: only tools that look like searches, reads, diagnostics, or usages are exposed, capped at 24, and Coda's own tools are excluded to prevent loops. `coda.vscodeTools.mode` (`off` / `readOnly` / `all`), `include`, `exclude`, and `maxTools` control it, and `Coda: Show VS Code Tools` lists what is bridged. Browser tools are not read-only, so enabling them takes an explicit `include`.
- **Coda Tools for VS Code's Agent.** `coda_repo_map`, `coda_search`, `coda_get_diagnostics`, and `coda_verify_workspace` are registered as language model tools so Copilot and agent mode can call them. All four are read-only; `coda_verify_workspace` summarizes live diagnostics and does not run tests or a build.
- **Native Editor Surfaces.** Checkpoints appear in the Timeline view with a compare-to-current diff and a confirmed restore (on VS Code builds that provide the Timeline provider API). Review findings can be shown as inline comment threads with Explain, Apply fix, and Dismiss. New editor, explorer, SCM, terminal, comment, and Timeline context-menu entries; Refactor and Generate-tests code actions; and Add Selection / File / Terminal Output to Chat, which fills the composer without sending.
- **`@coda` Participant Uses the Host's Model.** The chat participant now honors the model chosen in VS Code Chat, names attached tools to the agent, offers deterministic follow-ups, and renders changed files as references and a file tree with review buttons.
- **Small-Context Models.** Windows below 48K tokens run a compact request: 10 visible tools instead of 24 and a condensed contract, cutting fixed overhead from about 10.5K to about 4K tokens (everything else stays reachable through `searchTools`). Output reservation can no longer equal the whole window, the safe budget and compaction trigger now count the prompt and tool definitions, and a user-declared `coda.contextWindowTokens` is honored when sizing output.
- **Context Trimming Keeps Tool Memory.** When a step exceeds its budget, Coda now drops earlier conversation turns first, then shrinks old tool results to one-line records, and only last of all removes tool calls. Previously every tool call was deleted, so small-window runs forgot what they had just inspected and repeated themselves. Turns omitted from one request are never written back to the saved conversation.
- **Provider Error Clarity and Recovery.** A 404 from a provider is now reported as an unserved model (and suggests checking the base URL) instead of "could not verify native function calling"; 401/403, 429, and 5xx name their causes; an explicit "tools not supported" rejection is recorded so the model is not re-probed. When a provider ignores a forced inspection tool and answers in prose, the step is retried once without forcing.
- **Multi-Root Workspaces.** All 49 uses of the first workspace folder now resolve the folder the user is working in. Remote windows route Simple Browser previews through `asExternalUri`.
- **Chat UI Fixes.** A failed run no longer leaves a spinning "Working" chip above the composer. The "Answer the open question" suggestion no longer appears after Coda has answered your question. Code blocks no longer indent code twice, and the model badge, code-language label, syntax tokens, and red diff count meet text contrast in light and dark themes. Runs that changed nothing say "No file changes" instead of "0 files +0 -0".
- **Opt-In Integrations (off by default).** `coda.modelProvider.enabled` lists your configured model in VS Code's model picker (any extension using VS Code language models could then use your key), `coda.mcpProvider.enabled` offers `.coda/mcp.json` servers to VS Code's agent, and `coda.github.useVscodeAuth` creates pull requests with your VS Code GitHub sign-in when the `gh` CLI is missing. Scoped `.github/instructions/*.instructions.md` files (matched by `applyTo`) and `.github/skills` are also read.
- **Live Agent Evals and UI Preview.** `npm run eval:live` runs the real agent inside VS Code on fixture repositories and grades it with hidden tests (`eval:live:validate` checks the tasks without a model), and `npm run ui:preview` renders the chat UI in light and dark themes with overflow and contrast audits. Both are development tools and are not shipped in the extension.

## 1.30.12

- **VS Code Models Agent-Mode Reliability.** `vscode-lm` now advertises the documented `LanguageModelChatToolMode.Required` transport capability once a model passes the tool probe, so the mandatory first inspection is actually forced instead of being sent as an unforced suggestion. Forced inspection sends exactly one tool, matching the VS Code API constraint that some models accept only a single tool in required mode.
- **Recovery When a Provider Ignores Forcing.** A forced response that returns text with no tool call is no longer emitted as a normal turn. The adapter rejects it before it reaches the UI and AgentLoop retries the inspection once without forcing. Provider tool-mode rejections are labeled so the existing bounded recovery can run, and intent classification gets the same one-shot unforced retry.
- **Exact Model Identity for `auto`.** The `auto` sentinel is resolved to the concrete VS Code model id before probing, context-window selection, logging, and model construction, so capability evidence cannot be attached to the wrong model after the host's model order changes.

## 1.30.11

- **Context-Aware Follow-Up Suggestions.** Replaced the generic three-question fallback with ranked suggestions driven by the terminal outcome, failed commands, changed files, task category, delivery requirements, and the last request. Suggestions now refresh after Continue/revision runs, disappear when a new run starts, and insert into the composer without auto-sending.
- **Honest Incomplete-Run Recovery.** Runs that end without a provider final response are now reported as `incomplete` rather than a misleading success. The summary explains that the run stopped early, and Continue is offered from the saved progress.
- **Narrow-Panel and Approval Reliability.** Header, status strip, approval cards, action rows, and follow-up chips now fit panels down to 240–480 px without clipping or overlap. Approval cards keep their buttons visible when details expand or the composer grows, and the status strip no longer duplicates a decision card already on screen.
- **Touch and Accessibility Polish.** Primary touch targets are enlarged for coarse pointers, compact counters expose full values through tooltips and ARIA labels, follow-up chips wrap long file names, deny reasons are collapsible, and focus states are clearer.

## 1.30.10

- **Non-Terminal User Denial Feedback.** Denying an approval request no longer immediately aborts the run. The refusal and optional free-text reason are fed back to the model as a failed tool result so it can choose an alternative or ask how to proceed.
- **Repeat Denial Guard & Run Bounds.** Re-requesting an already-declined action is auto-denied without showing a duplicate card; runs enforce a strict bound (max 2 distinct user denials) before stopping as blocked.
- **Granular Approval Cards.** Added explicit "Deny" (with optional reason) and "Deny and stop run" actions to approval prompt cards.

## 1.30.9

- **Harness-Run Verification Contracts.** Verification requirements (`builds`, `lint`, `tests_run`) are now strictly determined and executed by the host harness against discovered project commands, rather than relying on model-declared shell execution. Tests require `executed > 0`, `failed === 0`, and `exitCode === 0`.
- **Harness Verification Failure Feedback Loop.** If a harness-run verification fails on a project test or build command, the output is fed back directly to the model as synthetic tool-result feedback with bounded retries (max 2), allowing the model to repair failures before finalization.
- **Command Policy Hardening against Injection.** Enforced newline and carriage return (`\r`, `\n`) command segmentation to prevent multiline shell smuggling; added Tier 2 classification for inline interpreter code execution (`python -c`, `node -e`, `bash -c`, `sh -c`, `perl -e`, `ruby -e`) and pipe-to-interpreter commands.
- **Enhanced Test Runner Summary Parsing.** Added and verified test count parsers across Go test, Pytest, Cargo test, Mocha, and Jest/Vitest.

## 1.30.8

- **VS Code Language Model (`vscode-lm`) Native Tool Calling Fix.** Fixed streaming tool execution bridge where tool-call chunks were not being dispatched to the AI SDK agent loop, resolving the "Required workspace inspection produced no successful tool evidence" failure mode when using VS Code models (Copilot).
- **Tool Schema & Mode Compatibility for `vscode-lm`.** Added fallback support for parameter schemas and configured `LanguageModelChatToolMode.Auto` / `Required` on chat requests.

## 1.30.7

- **Streamlined, Event-Accurate Activity Stream.** Retained Coda's true event-driven activity rail with per-operation elapsed timers, exact file/command targets, and real-time execution feedback, eliminating synthetic static milestone assumptions.
- **1-Click "Run in Terminal" on Code Blocks.** Interactive `[▶ Run in Terminal]` action on shell and terminal code blocks directly executes commands in the user's active VS Code terminal without manual copying.
- **Empty-Input Tab Mode Toggle & Composer Ergonomics.** Pressing `Tab` inside an empty composer toggles instantly between Agent and Chat mode, with Backspace at cursor position 0 removing trailing attachments and dynamic focus ring highlights indicating active mode.
- **Tactile Multi-File Resolution Action Bar.** Elevated the run summary file resolution controls into a prominent, modern action bar featuring primary `Keep changes`, `Review changes`, and `Undo changes` buttons.
- **Clean Collapsible Reasoning Accordion.** Automatically collapses settled model reasoning blocks into quiet `<details>` accordions (`Thought for Xs`), preserving narrative readability while keeping full thought audits expandable on demand.
- **Type-Aware Context Attachment Badging & Dropzone Overlay.** Upgraded composer context chips with specialized icons for media, symbols, terminals, and code files, paired with an illuminated backdrop overlay on file drag-and-drop.

## 1.30.6

- **1-Click "Run in Terminal" on Code Blocks.** Rendered interactive `[▶ Run in Terminal]` actions on shell, bash, zsh, and terminal code blocks, routing commands directly to the active VS Code integrated terminal.
- **4-Phase Autonomous Lifecycle Stepper.** Integrated a high-level milestone stepper into the live agent activity row (`Inspect` → `Edit` → `Verify` → `Done`) providing clear visual progress during multi-step runs.
- **Empty-Input Tab Mode Toggle & Composer Ergonomics.** Pressing `Tab` inside an empty composer toggles instantly between Agent and Chat mode, with Backspace at cursor position 0 removing trailing attachments and dynamic focus ring highlights indicating active mode.
- **Tactile Multi-File Resolution Action Bar.** Elevated the run summary file resolution controls into a prominent, modern action bar featuring primary `Keep changes`, `Review changes`, and `Undo changes` buttons.
- **Clean Collapsible Reasoning Accordion.** Automatically collapses settled model reasoning blocks into quiet `<details>` accordions (`Thought for Xs`), preserving narrative readability while keeping full thought audits expandable on demand.
- **Type-Aware Context Attachment Badging & Dropzone Overlay.** Upgraded composer context chips with specialized icons for media, symbols, terminals, and code files, paired with a backdrop overlay on file drag-and-drop.

## 1.30.5

- **`editFile` Multi-Hunk Batch Edits & Fuzzy Indentation Tolerance.** Support for atomic multi-hunk replacements via an `edits` array in a single WorkspaceEdit transaction, with whitespace-normalized fuzzy fallback matching to prevent edit rejection from minor indentation discrepancies.
- **`readFile` Structural AST Outline Mode.** Added `mode: "outline"` / `outline: true` to return high-level class, function, interface, and method signatures with line numbers, allowing models to survey 2,000+ line files in ~200 tokens.
- **`getDiagnostics` LSP QuickFix Harvesting.** Proactively queries language server code action providers (`QuickFix`) on compiler error ranges, attaching suggested fixes (such as missing imports or type assertions) directly to diagnostic outputs.
- **`runCommand` Error-Saliency Truncation Filter.** Smart failure prioritization preserves compiler errors, stack traces, and failing assertions when command outputs exceed budgets, filtering out verbose passing noise.
- **`manageTodos` Prerequisite Task Dependencies (`dependsOn`).** Enforces execution discipline by verifying all prerequisite tasks are completed before dependent tasks can be started.
- **Hardened Invariant Analysis.** Refined bracket and delimiter balance verification by stripping comments, string literals, and regex patterns to prevent false-positive AST rejections.

## 1.30.4

- **Dynamic Prefix Prompt-Cache Anchoring.** Structured 3-tier prompt caching partitions message histories into static cold roots, persistent warm session anchors (`Summary of earlier conversation (auto-compacted):`), and uncached live tails—reducing repetitive token inference costs by up to 90% and cutting latency on long-running multi-turn sessions.
- **AST Structural Invariant Verification.** Pre-edit syntactic guards prevent collateral file corruption by validating bracket/parentheses symmetry and rejecting accidental blanking of source files prior to disk writes.
- **Visual Blast-Radius Tree on Diff Cards.** File change rows compute and display downstream transitive dependent files (`X dependents`) using the dependency graph, providing instant risk visibility before approving edits.
- **Background Worktree Compare & Merge.** Extended background task management with 1-click diff inspection against HEAD and clean automated merging with conflict detection.
- **Transient In-Editor Writing Highlights.** Added active editor writing decorations providing immediate visual cues during file modifications.

## 1.30.3

- **Topological PageRank Symbol Repo Map (`repoMap`).** High-speed architectural orientation on Turn 1 using a power-iteration PageRank over the workspace import graph, extracting core classes, interfaces, and function declarations within an explicit token budget.
- **In-Loop Self-Healing Diagnostics Micro-Feedback.** Immediate post-edit diagnostic probing that detects syntax errors and type mismatches on mutated files, injecting actionable compiler notices directly into the model's tool step.
- **Test Impact Analysis (`findRelatedTests`).** Fast blast-radius analysis mapping edited files to relevant test suites and formulating exact test runner invocations to satisfy verification gates without full test-suite latency.
- **Evidence-Backed Commit & PR Packaging.** Automatic synthesis of audit-grade verification proof from Coda's Delivery Ledger into Git commit messages and pull request descriptions.
- **VS Code Language Model (Copilot) Hardening.** Modernized `vscode-lm` engine target to VS Code 1.95+, enabled case-insensitive model selection, added explicit user consent justification, and excluded `vscode-lm` from typing completions to protect user quota.

## 1.30.2

- **Trustworthy, accessible agent activity.** The activity stream now reports actual tool calls, file changes, checks, and failures without conflating them; edit badges use unified-diff line changes rather than before/after file totals. Each operation retains its own elapsed time, earlier work remains inspectable through a bounded disclosure, and the composer status strip no longer repeats an operation already visible in the conversation.
- **Safer, more usable operation details.** Read groups can reveal retained snippets and full captured content on demand. Tool arguments now lead with a redacted summary and place raw values behind an explicit sensitive-data warning. Truncated multiline output reports line ranges, with a direct route to the complete output where available. File navigation labels accurately distinguish opening a file from opening its diff.
- **Stable assistive-technology behavior.** Activity rows are reconciled in place with semantic list roles, a dedicated concise live announcer, and a stable Stop control outside changing operation content—avoiding focus loss and repeated screen-reader announcements during a run.

## 1.30.1

- **Steering message accuracy & UX safety.** Clarified rejection messaging when steering cannot be delivered due to no active run or a stale run target. Hardened "Insert at Cursor" against read-only and virtual editor documents with native undo stack integration. Documented architectural ledger boundaries for manual editor insertions.

## 1.30.0

- **Lifecycle event bus hardening & recursive secret scrubbing.** Tool execution is wrapped in a dedicated middleware layer that handles event-driven tool interception. Handlers returning `block: true` gracefully report structured policy feedback to models without crashing turns or throwing unhandled exceptions. Handlers returning `modify` reliably rewrite tool arguments before execution. `session_before_compact` is emitted prior to message compaction. Model-facing tool outputs (Anthropic, OpenAI, Gemini, Bedrock) are recursively scrubbed of API keys, Bearer tokens, and private keys while unredacted machine data is strictly preserved in internal ledger reconciliation state.
- **Sub-agent delegation & trusted command scoping.** Explorer sub-agents receive an isolated, lightweight, read-only system prompt that strips out parent verification contracts and git workflows, reducing token overhead. Explorer outputs are constrained by a structured 4-part distillation envelope with an enforced character ceiling to protect the parent model's token envelope. Delegated commands passed to executor sub-agents are strictly validated against authoritative workspace project commands; sub-agents can only narrow the parent's command trust list, never expand it or smuggle chained shell commands.
- **Native VS Code notebook tooling (NB-01).** Replaced raw `.ipynb` JSON file manipulation with capability-detected VS Code notebook APIs (`vscode.NotebookEdit.replaceCells`, `vscode.WorkspaceEdit`). Supports reading cell structure, outputs, and metadata, as well as atomic cell insertions, replacements, and deletions with pre-edit file snapshots, external conflict checks, safe save error handling, and approval gating. Cell and notebook executions are probed against active notebook controllers and explicitly labeled as asynchronous triggered actions.
- **Limited, adapter-aware DAP bridge (DB-01).** Added capability-detected debugger inspection using VS Code's native Debug Adapter Protocol. Supports active session detection, JSONC parsing for `.vscode/launch.json` (handling comments and trailing commas), dynamic active thread-ID resolution with adapter fallback, source breakpoint management, stack frame retrieval, variable queries, and scoped expression evaluation when sessions are paused.
- **Safe loopback browser preview.** Added `openBrowserPreview` with strict loopback validation (`isSafeLoopbackUrl`). Only local loopback interfaces (`localhost`, `127.0.0.1`, `::1`) are permitted, preventing SSRF or remote credential exfiltration, while clarifying that panel docking remains host-managed and directing self-contained component previews to `previewUI`.
- **Chat UX polish: Insert at Cursor, persisted composer history, & mid-run steering.** Added a safe "Insert at cursor" code block action that directly replaces the active editor selection using an undo transaction with visual feedback. Persisted prompt history across webview reloads and tab switches in VS Code webview state with `ArrowUp`/`ArrowDown` recall. Enabled mid-run steering allowing developers to steer active agent runs immediately or queue messages for subsequent turns.

## 1.29.6

- **Subagents now use their intended model lanes.** Explorer work can use the configured Scout model, ordinary writable work uses the Agent lane, and explicitly classified repair or architecture work uses the Architect lane.
- **Provider-safe routed models.** Coda validates provider compatibility and tool-calling support before switching a subagent model, then falls back to the parent model with a structured reason when the lane model cannot run the task.
- **Accurate role telemetry.** Run statistics now identify the actual model and purpose used by each run, including fallback reasons, token usage, and estimated cost when pricing is available.
- **Gemini model selection remains stable.** Selecting a Gemini model in the UI continues to control the main Agent unless an explicit compatible routing override is configured.

## 1.29.5

- **Provider-aware prompt-cache control and telemetry.** Cache annotations now respect provider capabilities and bounded breakpoint limits, while run summaries record cache reads, cache writes, and cache-read ratios where providers expose them.
- **Semantic tool-result retention.** Large historical tool outputs now collapse into deterministic, tool-specific tombstones that preserve key facts and point to the full disk spill without breaking the AI SDK message protocol.
- **Bounded verification repair.** Verification failures can enter a scoped repair loop with a three-attempt ceiling, pre-existing/infrastructure/unrelated-failure guardrails, and honest Delivery Ledger reconciliation when repair is exhausted.
- **Edit thrashing protection.** Rolling file fingerprints detect exact reversions and multi-file oscillation before cyclic edits consume the run budget.
- **Role-based model routing.** Scout and architect lanes route reconnaissance and complex repair work separately, with per-role token, latency, cost, and verification telemetry.

## 1.29.4

- **The Activity Bar icon is drawn for the Activity Bar.** VS Code renders an extension's sidebar icon as a **24px CSS mask filled with the theme's icon colour** — the file's own colour is discarded and the 1254px line art supplied for it collapsed to about 0.15px per stroke, which reads as a faint smudge. `assets/coda-side.svg` is a purpose-drawn 24px mark instead: a solid head, the ear nubs, and the visor cut out of it with the two eye arcs inside, so it stays legible at sidebar size and follows light and dark themes. The 1254px source image is kept out of the package.
- The Marketplace icon and the command icon are unchanged.

## 1.29.3

- **The listing points at the product, not at itself.** `homepage` and `repository` are now the Coda AI page and its GitHub home, `bugs` is its issue tracker, and the README footer links the publisher page — it previously linked a GitHub organization that does not exist. All three sidebar links now resolve.
- **Security reports have a private channel.** The README sends reports to GitHub's private vulnerability reporting form and links the published policy, instead of directing people to the Marketplace listing. Please do not open a public issue for a vulnerability.
- **One claim removed.** The README advertised MIT "with source available for inspection". The source is not published, so it now says only what is true: MIT licensed. The Support section also links the issue tracker.
- **New icon.** The extension icon is the current Coda mark, sized to the Marketplace's 1024×1024 ceiling, and the command icon is its vector counterpart.
- **Publisher renamed to Coda AI.** This is the publisher-page name, set on the Marketplace rather than in the package; the publisher ID (`express-genix`) and extension ID are unchanged, so existing installs and links keep working.

## 1.29.2

- **A model id Coda shipped can no longer break setting up a provider.** The setup wizard's free-tier cards verified your key against a model id written into the webview — `gemini-3.7-flash`, `openai/gpt-oss-120b`, `llama3.1` — and a connection check takes a model it is handed as your explicit choice, so it was used even when your account's own list no longer contained it. A vendor retiring one of those ids turned that card's **Connect** button into a failed check that blamed your API key. The cards now send only what you typed, and the host resolves the model: the provider's catalog default **when the account actually lists it**, otherwise a model the account does have. That is the rule the VS Code Models path already used, so it is not a new behavior — it is the one the rest of the file had. An explicitly chosen model (an Azure deployment name, a Bedrock inference profile) is still used exactly as given. Quick Start from the command palette passed the catalog default the same way and went through the same correction.
- **Keeping it that way.** A test now fails if any webview source ships a model id, the same guard the wizard's provider ids already had. A model id belongs to the provider's own catalog or to the catalog's per-provider default — read from there, never restated in a second place that can go stale on its own.

## 1.29.1

- **An incomplete Claude catalog entry no longer hides your reasoning setting.** 1.29.0 read which thinking form to send from any accepted type in the model's catalog entry, so a model that published only `disabled` — whether asking for *no* thinking is accepted, a different question from what to send when thinking is on — was taken to accept no thinking form at all, and the reasoning level you had chosen was dropped from the request without saying so. The form now comes from `adaptive` and `enabled` alone, and an entry that publishes neither keeps the version and name rules, exactly as a model the catalog has not described at all.

## 1.29.0

- **The reasoning level you pick is now the one the model accepts.** Coda decided both parts of a Claude thinking request from the model id: a version rule for which form to send, a name rule for the Haiku restriction, and a hand-written list of which two models stop at `max` effort. Anthropic's own model list reports all of it per model — the accepted thinking forms, the accepted effort levels, the context window, the output ceiling and the lifecycle — so the catalog now decides, and the version and name rules remain only as the fallback for a model the catalog has not described (no key yet, the picker never opened, a proxy that lists no capabilities). A level the model does not list is clamped to the nearest one it does, with ties going higher, so Extra High on a model that tops out at `max` still means that model's ceiling — the same choice the old two-model list was hard-coding, now derived from the provider's answer instead of enumerated by hand. Context window and output ceiling are read from the same catalog entry, which retires the bundled table for every model Anthropic reports.
- **The model picker says what the provider says about a model's life.** Anthropic's model list labels each entry `active`, `deprecated` or `retired`. A retired model fails every request, so it is no longer offered; a deprecated one still works for accounts that already have access, so it keeps its place in the list, marked with a **Deprecated** badge and its retirement date, after the active models. Removing a model you can still call would be worse than warning about it, and marking one the provider has not labelled would be inventing it — a model with no published lifecycle is neither hidden nor badged.
- **Opening the model picker is no longer a provider request, and no longer a batch of them.** Each open fetched the catalog live, and on a provider whose models must be verified before Agent mode admits them it also ran one inference call per unverified model. Coda now keeps the last catalog it read — per provider, endpoint and whether a credential was used — for five minutes and paints from it, so a second open costs nothing. The snapshot holds only the list; every window, request fact and tool-eligibility decision is still resolved from the store that owns it, so a cached list can never make an unverified model look verified, and a failed refresh drops the snapshot rather than serving it. The verification sweep also stops repeating itself: a model whose last probe ended with no evidence — a timeout, a rejected request, a rate limit — is not probed again on the next open, though it is retried once the backoff passes, and a run on it still probes immediately. The model you are configured on is verified first, so the one a run needs is settled in the first batch.

## 1.28.3

- **Choosing "off" for reasoning no longer breaks the newest Claude models.** The reasoning-effort setting has a *none* level that asks the provider not to think, and Coda sent `thinking: {type: "disabled"}` for it on every model. Claude Opus 5.5, Sonnet 5.5, Fable 5.x and Mythos 5.x reject that form with a 400, so a run with reasoning off failed at its first model call. Coda now sends the field only when the model's own catalog entry says the form is supported — Anthropic reports this in its model list — and otherwise leaves it out, so "none" never asks for something the model refuses.
- **A provider that rejects a forced tool call no longer ends the run.** An agent run's first step forces the model to call an inspection tool, and some models reject a forced call outright. The list of models that do was curated from vendor documentation and could only ever name the models already known; Coda now also recovers from the rejection itself — the step is re-sent unforced, once, and the run continues in prose. The curated list is still there, to avoid the failure rather than to recover from it.
- **A Claude codename without a version number is treated as current.** Coda picks between the two thinking forms by model version, and an id with no digits in it (such as `claude-mythos-preview`) fell through to the budget form that the newest models reject. Digit-less Claude ids now take the current form, matching what the resolver already documented for ids it does not recognise.
- **Provider options now describe the model that actually runs.** Reasoning effort, the thinking form and the model's thinking-disabled evidence are all model-specific, but they were derived from `coda.model` even when model routing sent the run somewhere else — and the agent runner can also fall back to the base model when a routed one cannot call tools. The agent runner now passes the model it resolved, and the chat and inline paths follow the same resolution they already use. With routing off, which is the default, nothing changes.

## 1.28.2

- **The Marketplace listing can no longer contradict the release it ships.** `vsce` publishes `package.json`'s version, `README.md` (the item page — version badge included) and `CHANGELOG.md` verbatim, but nothing linked the three: the README badge had fallen three releases behind, and 1.28.0's notes had been filed beneath 1.27.8's. The badge now reads the shipped version and the notes are ordered newest-first. A release step (`npm run release:bump`) updates the version and the badge together and refuses to run until the changelog entry exists, and the test suite now fails if any of the three drift apart.

## 1.28.1

- **Claude 4.6 and newer run again.** 1.28.0 fixed *when* Coda may force a tool call, but every Claude model was still sent the older `thinking: {type: "enabled"}` parameter with a token budget, and Claude 4.7 and later (Opus 5.5, Sonnet 5/5.5, Fable 5.1, Haiku 5.5, Mythos 5.1) reject that form with a 400 — so an agent run failed at its very first model call ("Coda could not complete this run") before any tool ran. Coda now sends adaptive thinking with the documented effort level on Claude 4.6 and later and keeps the budget form on 4.5 and earlier, chosen by model version rather than a fixed list of names.
- **The first inspection can be guaranteed again on the models that allow it.** Adaptive thinking does not conflict with a forced tool call — a forced call simply skips thinking — so the forced inspection step now applies to Claude 4.6 and later, where 1.28.0 had stepped aside for every thinking model. The models that reject a forced call outright (Opus 5.5, Sonnet 5.5, Fable 5.1, Mythos 5.1) still ask in prose, unchanged.

## 1.28.0

- **Agent runs work on current Claude models again.** Coda enables extended thinking by default, and Anthropic rejects a request that both enables thinking and forces a tool call ("Thinking may not be enabled when tool_choice forces tool use") — which is exactly what Coda did on an agent run's first inspection step. Every non-Haiku Claude model therefore failed at run start with "Coda could not complete this run", and the Haiku family worked only because it is the one model that gets no thinking budget. Forcing now yields to thinking, which is the setting you chose; the step still asks for exactly one inspection tool in prose, it just cannot be guaranteed. Claude served through Bedrock and Vertex inherits the same fix.
- **But that fix was not enough for the newest Claude models, and now they run.** Anthropic and Bedrock separately document that Claude Opus 5.5, Sonnet 5.5, Fable 5.1 and Mythos 5.1 reject a forced tool call outright — `any` and `tool` return a 400 whatever the thinking setting — so the guard above only covered thinking, and the provider row re-asserted forcing for exactly these models, failing every agent run at the first step. Coda now recognises them on every transport it speaks (Anthropic, Bedrock, Vertex, OpenRouter) and asks for the inspection step in prose instead.
- **DeepSeek runs again.** DeepSeek turns thinking on by default and its API rejects `required` and named tool choices while thinking is on, but the exception watched for a model id (`deepseek-reasoner`) that no longer exists, so the current models (`deepseek-flash`, `deepseek-v4-pro`) were still sent a choice the API refuses. No DeepSeek model is forced any more; tool calling is untouched.
- **Cohere is never sent a `tool_choice` it may not accept.** Cohere accepts only `REQUIRED`/`NONE`, and only on Command-r7b and newer — a boundary the model catalog cannot confirm per model, so Coda used to force `REQUIRED` on any tool-capable Cohere model and could fail an older one at run start. It now asks in prose.
- **NVIDIA NIM is never sent a `tool_choice` its runtime does not document.** The row carried OpenAI's full `auto`/`none`/`required`/`named` set, but NIM for LLMs documents only `none`, `auto`, and a named tool — `required` is documented for NIM for VLMs alone — so the forced first step asked for a mode the LLM runtime does not list. Coda now forces by naming the tool, which both runtimes document.
- **Bedrock no longer pins a tool by name.** Converse accepts a named `toolChoice` only for Anthropic Claude 3 and Amazon Nova; Coda uses the broader "must call a tool" mode instead.
- **Gemini's capability row no longer understates parallel calls**, which Google documents (multiple `functionCall`s in one response).
- **The default model is current, and no longer stored twice.** Several catalog defaults were years out of date — Bedrock pointed at Claude 3.5 from 2024, OpenRouter at an id that no longer exists — and four adapters kept a private copy of their own default, which would not have followed a catalog change. Defaults are now the ids the registries Coda reads at runtime actually list, and each adapter reads the catalog instead of keeping a copy.
- Gemini's thinking API is selected by major version rather than an enumerated list of releases, so a Gemini version Coda has never seen is judged as current instead of falling through to the legacy budget path.
- Internal: every provider capability claim was re-verified against the vendors' current official documentation; where a restriction is model-level rather than provider-level the capability matrix now states it as such, with the source for each claim. The OpenAI and Azure adapters now name the Responses API explicitly rather than inheriting it from the SDK's default — OpenAI documents that function calling on its current models requires it — so an SDK upgrade cannot silently move them back to Chat Completions.

## 1.27.8

- **A wrong key can no longer be saved silently.** Choosing a provider from the command palette wrote it and *then* asked for the key, so pressing Escape switched provider with no credential and left chat disabled, and nothing was verified, so a bad key surfaced only as a failing run. All three paths — palette, setup wizard, Quick Start — now gather, verify, and only then write, through one implementation. Escape at any prompt changes nothing, and switching provider repoints the per-purpose model routes so routing cannot pick up the previous provider's model id.
- **A custom Ollama URL is now actually used.** It was verified and then written to `providerBaseUrls.ollama`, which nothing reads — the runtime reads `coda.ollamaUrl`.
- **Providers that publish no model catalog can be connected again.** Azure OpenAI, AWS Bedrock, Google Vertex AI, Perplexity and Snowflake Cortex answer no model list by design, and verification read that as a rejected credential. The wizard refused to save them at all ("Connection verification failed. Configuration was not saved."), and once configured another way the panel called them unverified and disabled the composer — so chat was unusable. A catalog that does not exist is no longer treated as a failed check: the wizard saves, the panel enables chat, and both say the connection could not be checked rather than claiming it was verified. Nothing is asserted that was not confirmed.
- **Bedrock asks for its region up front, and a model that cannot be listed is offered as an editable default** — an Azure deployment name, a Bedrock inference profile — instead of being committed unseen.
- **The provider is switchable where it is named.** The operator panel's Provider value is now a real control built from the delivered catalog, the model selector's accessible name says "Change model and provider", and a provider that already holds a key is marked "Key saved". The Context Profile control, which had no styling at all and rendered as a stock browser button beside styled ones, now matches them, and both operator-panel menus close on an outside click.
- **Key portals for the providers that were missing one:** Azure OpenAI, Google Vertex AI, Cloudflare Workers AI and AI Gateway, GitLab Duo, and Ollama Cloud. Portals Coda cannot name — SAP AI Core, Snowflake Cortex, GitHub Copilot, self-hosted gateways — still show no link rather than a wrong one.

## 1.27.7

- **The setup wizard offers the whole provider catalog, not six providers.** The "Bring Key" tab hardcoded six pills and six key portals while the catalog already held thirty-seven providers, so a provider added to the catalog could not be configured from the wizard at all, and a renamed portal rotted silently. Pills and key portals now render from the same catalog the rest of the extension reads, the long tail sits behind a listbox trigger, and a provider whose portal Coda does not know shows no "Get key" link rather than a wrong one.
- **Providers that need an endpoint can finally be set up from the wizard.** The tab had no base-URL field, which made every endpoint-requiring provider (self-hosted, gateways, Azure) impossible to configure without editing settings. The field now appears for exactly the providers the catalog marks as needing one, and the endpoint is verified and saved together with the key.
- **Setup leads with the option that asks nothing of you.** When GitHub Copilot (or any other `vscode.lm` model) is available, the wizard opens on the VS Code tab and marks it, instead of leaving a new user on a key-prompt tab with no sign that a zero-setup path exists.
- **The walkthrough's ticks mean something again.** The five getting-started steps shared only two completion signals, so a step could be checked off by a button click unrelated to it. Every step now maps to its own milestone, fired by a real action — first message sent, agent run started, a change actually reviewed — so the checklist reflects what you have done.
- **First run says what it wants, and what it needs.** The activation toast now says what Coda is instead of only "Connect a model", and its "Don't show again" button is gone because dismissing already did the same thing. The "start free or bring your own key" explanation in Quick Start was never displayed — VS Code renders that detail line only for modal messages, and it was a toast — so the picker is now modal. The welcome copy no longer promises setup "in under a minute", and the Local tab states its real prerequisite (Ollama running with a model pulled) instead of only "No API key needed".
- **"Coda: Check Connection" is reachable from the chat.** It is offered in the Command Center so an unverified user can diagnose without leaving the panel, and it was retitled from "Check Configuration" to describe what it does: it runs a health check against the configured provider.
- Internal: the dead `viewsWelcome` block was removed — welcome content renders only for empty *tree* views, and the Coda panel is a webview — and the provider payload sent to the webview is now asserted by tests against the catalog, so no provider id can be hardcoded in the setup wizard again.

## 1.27.6

- **Truncated tool output is no longer a dead end.** A command result between 3,000 and 8,000 characters showed "…N hidden" with no way to reach the rest: the webview capped the body at 3,000, but the host only spills to a file above 8,000, so the remainder was in hand yet unreachable. The truncation note now offers **Show all**, which reveals the rest inline, and can be collapsed again. Results above 8,000 characters keep **Open full output**.
- **Changed-file rows in the run card open the file.** They looked clickable — pointer cursor, hover highlight — but were inert. They are now buttons that open the file (the host still refuses paths outside the workspace).
- **The Retry on a failed run is visible immediately.** It was hidden until hover like an ordinary response's actions, which is exactly the moment a recovery action should be obvious. An error now keeps its actions in view.
- **Keyboard users can reach every action.** Hover-only controls — per-message actions, edit, delete-a-saved-chat, copy-code, and per-file undo/open — now also appear on keyboard focus, and open dialogs (Command Center, settings, operator inspector) keep Tab inside them instead of letting it walk out into the page behind.
- **Rollback asks in-app.** The operator rollback used a blocking native dialog; it now uses the same inline confirmation as every other destructive action, themed and keyboard-consistent.
- **A late event from an older run can no longer be applied to the current one.** The "is this event stale?" rule and the terminal tombstone were each duplicated in three places that could disagree; both now have a single owner, so a late or out-of-order event from a previous run is dropped consistently.
- **A run that reported only its completion is no longer discarded.** The live run card was read from a state slot that was never written, so a run whose terminal signal arrived without a preceding status update could be silently dropped instead of clearing the transcript.
- Internal: the webview coordinator (`main.js`) is now type-checked alongside the rest of the webview, and the model-list cache moved from ad-hoc `window` globals into the state store.

## 1.27.5

- **Approved permissions stop stacking into a wall.** Every decided tool approval used to stay inline as its own row, so a long run — fifteen `git` permissions in a row, say — produced fifteen "Allowed" lines that buried the one decision the user actually had to make. Decisions now fold into a single **permission ledger**: the same subtle, softly-shadowed card surface as the live activity card, keeping the newest four decisions in view, older rows receding, and the rest counted behind a "N earlier permissions" disclosure. A `cd <dir> && <command>` wrapper is reduced to the command with the directory kept as quiet context, and the full command stays in the row's tooltip. Every decision remains in the DOM, so the ledger is a window, not a deletion.
- **Each run keeps its own ledger.** A new run starts a fresh card, so a previous run's decisions stay where they happened instead of being rewritten into an unrelated part of the transcript.
- **High-contrast users keep the card edge.** The tint-and-shadow card family — the pending approval card, the live activity card, and the new permission ledger — now re-asserts its boundary under `forced-colors`, where the palette swap otherwise erases both the tint and the shadow.

## 1.27.4

- Dead code cleanup: removed the command dock (chips/overflow) and permanently hidden input hint. Command Center now lives only in the header; no UI surface was lost.

## 1.27.3

- **Command Center has a visible way in.** ⌘K was the only route to the Command Center, and the token `/run` `/parallel` `/pr` `/ship` commands had no visible teaching. Rather than restore the chip row — which is redundant with the `/` menu and eats the vertical space reclaimed for chat — a compact Command Center button now sits in the header beside the surface icons. The dock markup, its narrow-panel overflow, and the always-hidden `/`-hint line are gone rather than left dead in the tree.
- **Escape stops a run again.** With nothing else open, Escape is the universal stop gesture, but the "is an overlay open?" check tested an attribute the command center never uses, so it was always considered open and the stop never fired. The check now reads the overlay's actual open state.
- **Reverting changes asks first.** Undoing an entire run from the summary card, and undoing a single file from its change row, both reverted immediately with no redo. Both now confirm, matching every other destructive action.
- **Keeping or reverting changes no longer passes silently.** "Keep changes", a kept file, and a successful revert now show a brief, screen-reader-announced confirmation instead of nothing; a failed revert still reports the failures.
- **Token usage is shown.** Each run's input/output token totals were captured but never displayed; the completed run's execution details now include a **Tokens** metric (with the input/output split on hover) beside cost.
- Removed the context readout that could never render: its `<span>` was `hidden` inside a 2px, overflow-clipped row. The exact occupancy now travels with the meter as an accessible value and in its tooltip, so nothing is silently dropped.

## 1.27.2

- **A stopped run and an unfinished run now read differently.** The run card used to label both "Stopped", which made a run that simply ran out of steps look like one you deliberately cancelled. A cancelled run reads **Stopped**; a run that finished incomplete reads **Needs review**.
- **You are told when a run fails, even a short one.** The finish notification's duration gate now applies only to successful runs, so a fast failure, timeout, block, or incomplete run that finishes while you are away still notifies you. Incomplete and step-limited runs also offer **Review Changes**, and the notification title carries the task category.
- **The context meter warns you before it fills.** A small nudge appears as the window approaches capacity — first a heads-up, then a notice that compaction is imminent, then a note that older turns were summarized — instead of only recolouring the 2px meter.
- **Screen readers hear the answer land.** The streamed answer is not a live region, so completion and failure were silent for screen-reader users; both are now announced through a shared, visually hidden live region.
- **Follow-up prompts match what just happened.** An unfinished run offers how to continue it; a run that changed a file offers to write tests for that file — instead of the same three generic chips every time.
- Fixed the run-status strip re-announcing itself: it no longer rewrites the DOM when the chip set is unchanged, so a spinning "Working" state stops spamming the live region.
- Cost values explain an inexact number (`~` / unknown) through a tooltip rather than a bare glyph.
- Markdown rendering now degrades to escaped plain text if the optional `marked.js` bundle is missing, instead of throwing and blanking the surface.

## 1.27.1

- Removed the contradictory "VERIFIED EDIT SUMMARY" footer that could claim an edit failed after it had succeeded. Ground truth is now the diff cards and per-tool warnings alone, matching how other agent surfaces present changes.

## 1.27.0

- **The agent now works where your code lives.** Coda declares `"extensionKind": ["workspace"]`, so it runs in the workspace extension host (local, SSH, WSL, dev container, Codespaces) and its file and shell tools operate on the real workspace instead of a UI-host view of it.
- **Commands can no longer hang on a prompt.** Every headless command runs with non-interactive defaults (`CI=1`, `PAGER`/`GIT_PAGER`/`GH_PAGER=cat`, `GIT_EDITOR=true`, `GIT_TERMINAL_PROMPT=0`), and commands that inherently need a terminal — editors and REPLs (`vim`, `less`), `git commit` without a message, `git rebase -i`, `-it` container execs, `gh auth login` — are rejected with a suggested non-interactive form instead of timing out.
- **A smaller, faster tool surface.** Each request now sends only the 15–25 tools that fit the active mode, instead of every registered tool at once. When the model needs a capability that is not visible, a new `searchTools` tool finds it and activates it on the next step, so nothing is lost. Per-request tool-definition tokens (visible and deferred) are measured and logged per mode.
- **MCP servers no longer pay a token tax until used.** Tools from connected MCP servers stay registered but are not sent with a request until the model discovers them, so a large server adds zero tool-definition tokens to your prompt until it is actually needed.
- **Long-running processes, without blocking the run.** Coda can now start a dev server, watcher, or test runner and keep working while it runs: `startProcess` launches it, `readProcessOutput` streams incremental output, `sendProcessInput` answers a prompt, and `stopProcess` stops it and its whole process tree, with `listProcesses` for the full set. `waitFor` blocks until a port is listening or a pattern appears in the output, and `runTask`/`listTasks` drive your VS Code tasks. `readManyFiles` reads a batch of files under one shared budget, and `fileStat` reports size, type, modification time, and whether a file is binary.
- **Current documentation and approval-gated API calls.** `docsLookup` queries a Context7-compatible documentation index and pins results to a library version, so the agent cites current API docs instead of guessing. `httpRequest` makes bounded, SSRF-guarded API calls that always ask for approval first.
- **A real network egress policy.** New `coda.network.allowedDomains` and `coda.network.blockedDomains` classify outbound requests, and `coda.network.proxy` / `coda.network.noProxy` route them through your proxy. The policy is enforced on every redirect hop of a fetch, and private-network hosts stay blocked.
- **Truncation you can see through.** Every tool result is now bounded by a single ~20K-character budget, and a truncated result states exactly how many characters were shown and omitted, an estimated omitted-token count, and how to read the rest — instead of silently dropping the tail. The full-file read default dropped from 50KB to 20KB to match.
- **Structured file-edit results.** File mutations (create, edit, delete, rename, undo) now return typed results carrying the operation, path, and line deltas, so the run summary, reconciliation, and audit ledger read facts instead of re-parsing prose.
- **Named permission modes with a one-click toggle.** A new three-way `coda.permissionMode` — **Ask**, **Auto-edit**, **Auto-all** — sits on top of the engine's approval settings, is shown in the status bar, and is switchable from the status bar or via `Coda: Switch Permission Mode`. Turning on the broadest mode warns first. Existing `permissions` / `autoApproval` configurations keep working unchanged.
- **Protected paths.** Key files, credentials, `.env` files, and your configured globs are now refused or gated behind approval, and per-tool idempotency metadata tells the model which actions are safe to repeat.
- **Explain and fix diagnostics where they appear.** Every error and warning now offers a **Coda: Explain this problem** code action, with **Coda: Fix this with agent** on errors, so you can act on a squiggle without leaving the editor.
- **A context meter that stays live during agent runs.** The context-usage meter now updates from the real per-step token count as an agent run proceeds, not only in chat, so you can see the window filling before it is full.
- **A notification when a long run finishes while you are away.** New `coda.notifyOnFinish` (`unfocused` | `always` | `never`) raises a notification with **Show Chat** / **Review Changes** when a long run completes, fails, or is waiting on your approval and the window is not focused.
- **Merge-conflict, coverage, and secret tools.** `resolveConflicts` parses conflict markers (including diff3 base blocks) and proposes ours/theirs/manual per hunk; `getCoverage` reads LCOV or Istanbul `coverage-summary.json` and names the worst files; `scanSecrets` scans for hard-coded credentials (AWS, GitHub, Slack, Stripe, Google, private keys) with redacted output. All three are read-only inspections.
- **Structural (AST) search and codemods.** `astSearch` answers questions text search cannot — every call to a function, every class named X, every module importing Y — by parsing your code with tree-sitter, so it never matches code-like text inside a string or comment. `codemodRun` applies syntax-aware rewrites (rename a symbol, retarget calls, remove imports) atomically across the workspace, and aborts if any file changed outside Coda.
- **Tool hooks.** New `coda.agent.hooks` runs your own pre-/post-tool scripts matched by tool name. A failing pre-hook blocks the tool with a clear message (the run continues rather than aborting), and post-hook output is appended to the result. Hooks are disabled in untrusted workspaces.
- **Scheduled tasks.** New `coda.agent.scheduledTasks` runs a prompt as an agent turn on an interval, with an optional run cap, no self-overlap, and run state persisted across reloads.
- **Nested project rules.** Coda now loads nested `AGENTS.md`, `CLAUDE.md`, and `.coda/rules.md` files (closest first) within a token budget and with caching, so rules can live next to the code they govern.
- **Multi-root workspaces.** Path resolution is now folder-qualified, so a workspace with several roots resolves each file against the correct root.
- **Hardened by construction.** A red-team suite gates the release: permission decisions ignore untrusted content, path traversal is rejected in every encoding, chained or encoded destructive commands cannot reach auto-approval, and credential files and private-network hosts stay gated even in Auto-all mode.
- **Every provider and tool is conformance-tested before release.** One gate runs a fixed core task set against every provider in the catalog and fails the release on any failure; another validates every registered tool against each provider's structured-output schema family. A per-tool metrics ledger records calls, error rate, retries, and cancellation latency for the evaluator.

## 1.26.34

- **No more hardcoded output limits.** Coda now resolves each model's real per-response ceiling dynamically — from the LiteLLM model registry (refreshed every 24 hours), the provider's own `/models` catalog, and limits learned from provider rejections — instead of clamping every response to a fixed 16,384 tokens. A new or updated model (GPT, Claude, Gemini, or a private gateway) picks up its true limit with no extension update, and `coda.maxOutputTokens` only applies when you set it explicitly.
- Thinking budgets are now mathematically coupled to the output ceiling: Anthropic's `budget_tokens` always leaves at least min(8,192, 25%) tokens of completion headroom — eliminating the fatal `budget_tokens must be less than max_tokens` API crash — and Gemini's legacy thinking budgets are capped against the same envelope, so internal reasoning can no longer consume the entire response and leave nothing for the answer.
- If a model still spends its whole budget on reasoning and returns nothing, Coda now recovers automatically instead of stopping: it expands the output ceiling to the model's real limit, lowers reasoning effort to Low, and steers the model to answer directly — one attempt, with the existing Continue card kept as the fallback.
- When a provider rejects a request by stating the model's real output cap (e.g. `max_tokens is too large ... maximum value is 16384`), Coda learns and caches that ceiling, so the next call succeeds without any manual setting change.

## 1.26.33

- Fixed the output-limit continuation looping: a segment that hits the token limit without producing any text or running any tools (which is what a reasoning model does when it spends its whole output budget on thinking) is no longer re-asked. Coda now stops immediately, shows a **Model produced no output** status, and hands the run back to you via the Continue card instead of burning the whole budget on empty retries.
- The auto-continue decision is now mirrored to the Extension Host console, so a continuation cycle can be diagnosed from the logs rather than looking like an unexplained runaway.

## 1.26.32

- If an automatic output-limit continuation fails partway, Coda now keeps the answer it already streamed and offers Continue to retry, instead of losing the partial response to a hard error.
- The streamed continuation and the committed answer are now joined byte-for-byte by a single separator definition, so the text you watch render and the text saved to history can no longer drift apart.
- Cleaned up the shared constants module: removed sixteen values that nothing used, and gave the HTTP timeout and conversation-size limits a single source of truth instead of duplicated literals.

## 1.26.31

- A response cut off by the model's output-token limit no longer stops the run behind a manual "Run paused" card. Coda now continues the answer automatically in the same run — feeding the transcript back with a "resume exactly where you stopped" instruction, up to 5 times — and streams the continuation onto the same answer surface so the text you already see is never erased. A long analysis now reads as one continuous reply instead of pausing to be clicked forward.
- Refreshed the welcome screen and in-webview setup onto the softer, borderless design language: a quiet hero glyph, ghost suggestion rows, a soft accent for the first-win action, and codicons instead of emoji throughout the provider setup.
- Removed the redundant "Setup Required" banner — it was never displayed and is fully replaced by the in-webview setup wizard.

## 1.26.30

- Each response in a conversation now keeps its own activity window. Previously a run whose finalize was missed left its narrative (activity window, read group) open, so the next message's activity slid in under the first response instead of the current one — and repeated reads accumulated into a stale "Read N files" group.

## 1.26.29

- Permission requests now pause the run until you answer — no 5-minute deadline that silently dropped an action while you were away. A pending tool approval (and the agent's own "ask the user" question) waits indefinitely and is only cleared when you respond or when the run is stopped / the panel is closed — the same block-until-you-answer behaviour as Codex and Copilot.

## 1.26.28

- Tool-approval cards no longer leak raw JSON: arguments render as a quiet key/value list (nested objects and arrays included), a tool with no arguments shows nothing instead of `{}`, and the collapsed chip after you decide summarises the action in plain words — e.g. `src/a.ts` — rather than the first 60 characters of stringified arguments.

## 1.26.27

- Finished the shared type scale: the whole transcript now reads from the same size and weight tokens instead of a scattering of near-identical hard-coded values.
- Unified the timeline hairline behind one width token, and collapsed three stacked, overridden composer button size rules into a single source of truth.
- Menus and the inline confirmation toast traded their hard 1px borders for a soft ring shadow so they read as quiet regions rather than boxed cards, and hover-only reveals are now gated to true hover pointers so a touch device can never get stuck in a hover state.

## 1.26.26

- Menus now ease out instead of vanishing the instant you dismiss them, and the composer's Stop control enters and exits smoothly rather than blinking in.
- Every file-change row gained an inline **Undo** that reverts just that file and shows the result on the row, and the live activity row now ticks an elapsed duration so a long operation reads as running rather than frozen.
- The run-status strip is announced to screen readers, hover-only controls stay reachable on touch devices, keyboard focus is visible wherever the default outline was stripped, and a Windows High Contrast (forced-colors) palette keeps every surface legible.
- Quiet polish throughout: one deceleration curve across entrances, no `transition: all`, decorative spinners paused while the panel is backgrounded, and a shared type scale for the transcript rows and chips.

## 1.26.25

- A file edited several times in one run now collapses into a single summary row ("Updated path ×6 +86 −63") with cumulative totals instead of stacking one near-identical row per edit; the row lights back up while the next edit is in flight.
- Unified the transcript's change rows and approval cards onto the same soft, borderless surface as the activity stream, and threaded a timeline (chain-of-thought) rail through the live activity steps so the run reads as one continuous line instead of a stack of stray cards.
- Tool approvals no longer dump raw JSON: a non-edit approval shows its arguments as a quiet key/value list.

## 1.26.24

- The run-status strip now shows what Coda is doing during read-only exploration instead of a bare "Working": a live-operation chip ("Inspecting files · src/a.ts") plus an accumulating "Explored N" count of the files and directories it has looked at, so a long look-around reads as progress rather than a static spinner.

## 1.26.23

- Added a persistent run-status strip above the composer: live counters for files changed, terminals running, and artifacts produced, plus a pinned **Waiting on you · <action>** chip whenever a tool approval, continue prompt, or question is blocking the run.
- The live activity stream now keeps only the most recent five operations in view and fades older ones, so the current action stays readable without the list growing without bound.
- Softened the activity/run and inline-diff styling — borderless rounded regions with subtle shadows — for a quieter transcript.

## 1.26.22

- Automatic tool denials no longer abort the run. A plan-mode, execution-profile, command-policy, or delegated-subagent denial is now returned to the model as a normal tool result so it can adapt and still deliver a final answer; only a genuine refusal, a Stop, or repeated identical denials end the run. (User refusals, timeouts, and lost transport responses remain terminal as before.)
- A run that does stop without a final message now reports the real cause and lists the changes already applied to your working tree, instead of the bare "run stopped before completion" notice.
- Removed the plan-mode instruction that promised read-only shell commands, which the policy did not actually allow — the prompt now matches what is permitted.

## 1.26.21

- Agent runs are no longer capped by a step counter. The loop continues while the model keeps working and ends when it delivers a final answer, a permission decision stops it, or you press Stop; length is bounded by the context window and compaction, never a task step budget.
- Edits now land on disk as soon as a tool applies them; follow-up commands, git, and tests no longer read a stale unsaved buffer.
- Stopping a run now kills the whole process tree, including detached children and grandchildren (POSIX process-group signal; Windows `taskkill /T /F`), and `runCommand` honors cancellation with exit code 130 instead of hanging.
- The local review server now requires a per-session token (`X-Coda-Token`, written to `.coda/server.token` with owner-only permissions) and rejects browser-originated requests; the git hook sends the token and adds a request timeout.
- Writable subagents are serialized so they cannot race on shared workspace state; read-only subagents stay fully parallel.
- The conversation picker (clear/load/delete) no longer queues behind a running turn — it refuses while a run is active instead of blocking.
- Diff application tolerates blank context lines and trailing whitespace, preserves the file's exact context bytes, and reports the precise mismatching line when a patch fails.
- A completed agent turn now presents a single collapsed summary line (outcome + files + operations + time) before the answer; the run-summary card mounts inside it and reasoning lines fold into the trace.

## 1.26.20

- Stopped the composer from stealing focus from the editor: run-state changes, view reloads, and tab switches now only focus the input when the Coda panel already owns focus.

## 1.26.19

- Fixed the "Open in editor" button on structured diff panels silently doing nothing — it now uses the delegated `data-open-file` handler that actually opens the file.
- Removed the duplicated `@@` prefix on diff hunk header rows (previously rendered as `@@@@ …`).
- Escaped the collapsible raw-output block in structured findings so tool text renders literally instead of being parsed as HTML.

## 1.26.18

- Audited the agent panel against the target behavior spec and closed every gap (42/42 items passing; report in `AGENT_UI_AUDIT.md`).
- Approval cards now render an inline diff preview with a warning tint instead of raw JSON, and `⏎`/`Esc` decide the active card.
- `Esc` stops the active run when no overlay is open, and in-flight tool rows finalize with a distinct `cancelled` state on stop.
- The composer queues messages typed while a run is active (shown as chips) and sends them when the run completes; Up/Down recall previously sent messages.
- Streaming code is highlighted after a short debounce instead of on every delta, and a selection inside the streaming tail is no longer dropped mid-render.
- Expanded tool rows show a collapsible **Arguments** section; live terminal output is trimmed to a rolling tail in the DOM.
- Added a quiet "Still working…" status after ~5s of silence, and off-screen timeline blocks use `content-visibility` so long sessions stay responsive.
- Exposed bindable commands and keybindings: Focus Message Input, New Session, Stop Active Run, Accept/Deny Pending Edit, Toggle Chat/Agent Mode, Jump to Latest Message.

## 1.26.17

- Slash autocomplete now matches commands with fuzzy subsequence search over the command name and description, ranked so prefix matches come first — `/cm` finds `/compact`, `/rev` finds `/review`.
- Added global frontend error reporting: uncaught webview errors and unhandled promise rejections are sent to the extension's **Coda** output channel, so silent UI failures become diagnosable.
- Made `npm run publish` rebuild from source first, preventing stale bundles from shipping in future releases, and stopped tracking packaged release binaries in git.

## 1.26.16

- Fixed the `/resume` structured form: its primary Resume button silently did nothing because the submit handler expected a free-text field the form does not have. It now dispatches the selected checkpoint (or the latest one).
- Made `/run`, `/parallel`, and `/resume` open their structured forms immediately, without waiting on the async command catalog, so command-dock chips and the Command Center never degrade to a bare text insertion.
- Fixed the slash/mention autocomplete and structured forms being clipped to invisible: the input container's `overflow: hidden` was cutting off the popovers that anchor just above the input box.

## 1.26.15

- Streamed chat and agent output as coalesced `uiDelta` text deltas instead of full accumulated messages, so long answers arrive slice-by-slice at frame rate.
- Rendered streaming output with block-scoped rendering: committed paragraphs and closed code fences are parsed exactly once and frozen, and only the incomplete tail re-renders — per-token cost no longer grows with answer length.
- Skipped re-rendering a finalized answer when the streamed raw text already matches the final content, eliminating end-of-run flicker.

## 1.26.14

- Rendered `applyDiff` results as a structured diff panel with color-coded add/del/hunk lines, change stats, and an Open-in-editor button.
- Rendered search tool results as grouped, clickable file:line match rows instead of raw text.
- Improved live terminal steps: the output panel auto-opens with a live "Running" badge while a command executes, streaming output auto-scrolls, and failed commands open their output automatically.

## 1.26.13

- Added the verified onboarding wizard that validates provider, API key, and model access during setup.

## 1.26.12

- Removed persistent management panels from the Chat container and returned Checkpoints, MCP, and Background Tasks to contextual command/run surfaces.
- Kept the full chat composer controls visible while idle for a stable, predictable input experience.

## 1.26.11

- Moved Checkpoints, MCP Servers, and Background Tasks into a separate Coda Management activity-bar container so they no longer consume Chat space.

## 1.26.10

- Completed the remaining official VS Code integration wiring and removed unsupported API paths.
- Added native model, task/testing, diagnostics, language-status, workspace-trust, lifecycle, diff, and TreeView coverage.
- Added regression coverage for VS Code integration behavior and large-workspace safety.

## 1.26.9

- Widened the chat composer in narrow VS Code panels while preserving the compact layout.

## 1.26.8

- Added native VS Code model support through `vscode.lm`, including streaming and tool-call bridging.
- Added VS Code Git, Testing, task-discovery, terminal-fix, chat-reference, multi-diff, and native TreeView integrations.
- Added regression coverage for the VS Code integration bridges.

## 1.26.7

- Improved agent orchestration with AI SDK-native step compaction and preserved tool-result context.
- Removed redundant automatic planning and unused post-run judge paths.
- Added bounded semantic call-hierarchy navigation using VS Code language services.
- Added semantic type-hierarchy navigation for supertypes and subtypes.
- Expanded language-service integration for symbols, definitions, references, hover information, refactorings, and rename.
- Fixed release documentation so retrieval and optional embedding behavior are described accurately.

## 1.26.6

- Preserved mixed review-and-implementation requests through the agent workflow and fixed approval-card event sequencing.
- Refined the chat composer with clearer controls and accessible selector states, then tightened the input and menu spacing for a compact sidebar layout.

## 1.26.5

- Aligned post-edit validation policy with baseline-reconciled results so pre-existing failures warn without blocking subsequent writes.
- Kept newly introduced failures and missing baselines fail-closed, with regression coverage for the full validation decision matrix.

## 1.26.4

- Made the backend run verdict authoritative across `runStatus` and `runComplete`, including reordered, duplicated, or missing terminal events.
- Added structured terminal-state reconciliation and explicit protocol-error handling when a terminal verdict is missing.
- Preserved authoritative zero step counts and expanded backend-to-webview terminal contract coverage.

## 1.26.3

- Prevented generic file reads from bypassing managed skill loading and added run-scoped skill context budgets.
- Replaced retired Marketplace metadata badges with reliable version, Marketplace, and license badges.
- Unified approval routing around the canonical run identity and removed global approval sinks, session fallbacks, and unscoped response APIs.
- Added structured approval outcomes so policy blocks, user denials, transport failures, surface disposal, and successful approvals remain distinct in the audit trail and UI.
- Added approval-flow regression coverage, including real VS Code extension-host tests for fail-closed routing, duplicate requests, cancellation, and non-retryable denials.

## 1.26.2

- Added safe import of instruction-only skills from local packages and public GitHub repositories, with commit pinning, archive limits, path validation, previews, and import hashes.
- Refined the chat composer selectors and mode menus so labels remain readable, long descriptions wrap, and nested menus are not clipped in narrow panels.
- Removed competitor-specific comparisons from user-facing settings and help copy; settings now describe Coda's behavior directly.
- Updated Marketplace README images to use public HTTPS assets and improved Marketplace publishing metadata.

## 1.26.1

- Made structured verification records and the canonical run verdict the sole runtime authority for completion, rollback, and UI verification state.
- Removed legacy verification adapters, counter-based verdict reconstruction, and text-based validation status markers.
- Added bounded DNS lookup timeouts for fetch operations so stalled resolution cannot hang an agent run.

## 1.26.0

- Consolidated the responsive conversation rail so user and assistant markers stay inside the panel and never overlap their labels.
- Removed stale competing marker offsets from the stylesheet to keep narrow-layout behavior single-source and maintainable.
- Added automated coverage for the webview contract, responsive rules, accessibility controls, and VS Code E2E smoke surface.

## 1.25.100

- Fixed conversation marker labels overlapping the first characters of user and assistant names.
- Reserved marker space in the responsive timeline rail so narrow layouts remain legible.

## 1.25.99

- Fixed narrow-sidebar conversation markers being clipped by the messages viewport.
- Kept user and assistant turn icons inside the timeline rail gutter at responsive widths.

## 1.25.98

- Replaced the remaining model, preset, and tool-permission native selects with theme-aware listbox popovers.
- Unified model/provider/context/command menu keyboard navigation, focus restoration, selection state, and outside-close behavior.
- Added regression coverage for real DOM focus semantics and removed the last native-select path from the webview chrome.

## 1.25.97

- Hardened smart command approval so only explicitly safe command contracts auto-approve; unknown commands remain confirmation-gated.
- Added workspace-owned slash-command templates under `.coda/commands/` with bounded `{{args}}` and `{{command}}` rendering.
- Added explicit `--worktree` routing for `/parallel` and `/swarm`, reusing isolated background worktrees without changing ordinary run behavior.
- Made chat rewind and regeneration explicit about keeping or reverting Coda-tracked workspace changes, with transcript restoration when a replacement run cannot start.
- Added behavioral coverage across command policy, slash templates, worktree routing, and rewind safety.

## 1.25.94

- Added a structured public activity stream for agent lifecycle and tool progress without exposing raw model reasoning.
- Added an explicit queued state when a submission waits behind an active run.
- Kept activity targets limited to bounded resource identifiers and excluded model-authored queries, commands, URLs, and package text.
- Added activity-stream behavior coverage and verified the release with the full test suite and VS Code E2E smoke tests.

## 1.25.93

- Fixed sequential-run streaming admission when a previous run's terminal UI event is delayed or lost.
- Hardened streaming rendering against overlapping chunks, duplicate finalization, surfaceless terminal events, and stale render state.
- Added runtime regression coverage for run succession, incremental streaming, and single-flight terminal finalization.

## 1.25.92

- Hardened the agent/webview contract with structured verification outcomes, canonical preset metadata, and explicit project task discovery.
- Removed inline webview style mutations for strict CSP compatibility and simplified UI state rendering.
- Added request-hunk transport coverage and strengthened context, command-discovery, project-registry, and benchmark regression tests.

## 1.25.36

- Stabilized the running status bar with a single activity owner, one-second updates, compact elapsed-time text, and token-rate details in the tooltip.
- Replaced competing status timers and repeated visibility calls to eliminate flicker during agent runs.

## 1.25.35

- Fixed approval-mediated tool turns dropping the real tool result after the approval response, which could leave OpenAI-compatible providers with incomplete `tool_calls` histories.

## 1.25.34

- Prevented AI SDK missing-tool-result failures by removing unresolved assistant tool calls before resumed or continued provider turns.
- Preserved valid mixed tool turns and approval responses while dropping only incomplete protocol entries.

## 1.25.33

- Fixed AI SDK tool-message protocol violations by preserving structured `tool-call`, `tool-result`, and approval parts across agent turns and persisted resumes.
- Added canonical sanitization that removes orphan or duplicate tool results before provider calls, with regression coverage for real AI SDK message shapes.

## 1.25.32

- Fixed false-positive semantic loop detection by preserving meaningful tool inputs in bounded, deterministic action fingerprints.
- Improved loop-detection coverage for searches, symbols, edit ranges, replacements, commands, and unknown/MCP tool arguments.

## 1.25.31

- Fixed agent tool cards that could remain stuck on an active spinner after the run completed.
- Unified webview tool-card markup and finalization state handling for reliable completion rendering.

## 1.25.30

- Improved agent activity lifecycle rendering, including reliable cleanup between runs and explicit completion/error states.
- Added automatic recovery for stale activity indicators and duplicate thinking elements in the chat webview.

## 1.25.29

- Added persisted code-graph and optional embedding indexes for faster, more relevant workspace retrieval.
- Added terminal session history, conversation forking, background-task management, and automatic post-edit validation controls.
- Hardened session recovery, model/token handling, workspace tracking, and background worktree management.
- Registered the new commands and configuration settings in the extension manifest.

## 1.25.28

- Hardened agent execution boundaries: workspace-safe paths, command and verification policy enforcement, Git mutation permissions, and model-aware tool ordering.
- Added run-scoped approval routing and serialized write/validation sequencing to prevent cross-run approvals and stale validation results.
- Strengthened MCP and checkpoint safety with trust gating, transport limits, conflict detection, content hashes, and recoverable restore failures.
- Added context provenance and execution telemetry, plus regression coverage for the security and orchestration contracts.

## 1.25.27

- **Webview TypeScript migration, slice 4 — messages**: `components/messages.js` (the largest, most UI-critical component) ported to strict TypeScript (`webview/src/ui/messages.ts`, ~620 lines) with behavioral parity: message virtualization (spacers + deferred timers), user/assistant/error rendering, inline edit-and-resend, non-blocking inline confirm, regenerate-with-model menu, thinking indicator, token usage, history restore, agent text-block actions, and context-sources chips. Typed interfaces for `VirtualEntry`/`TokenUsage`/`HistoryEntry`/`ModelOption`; `state.ts`-keyed generic types (`currentStreamEl`, `thinkingTimer`) enforced by the store's `keyof` typing. Build now emits **five** per-module bundles served in dependency order (`state → markdown → scroll → messages → components → main`); `window.CodaMessages` declared in `globals.d.ts`. Old `messages.js` removed.
- **Webview TypeScript migration, slice 3 — markdown & scroll**: `components/markdown.js` and `components/scroll.js` ported to strict TypeScript (`webview/src/ui/markdown.ts`, `webview/src/ui/scroll.ts`) with minimal ambient types for the `marked`/`hljs` script globals. Build now emits four per-module bundles served in dependency order (`state → markdown → scroll → components → main`); `window.CodaUI`/`window.CodaScroll` are declared in `globals.d.ts`. Script load order is unchanged, so classic-script components need no edits. Old JS files removed.
- **Webview TypeScript migration, slice 2 — state store**: `webview/state.js` ported to strict TypeScript (`webview/src/state.ts`) with a fully typed `CodaState` shape and generic pub/sub API (`get/set/on/off` keyed on `keyof CodaState`). The webview build now produces **per-module bundles in load order** (`dist/webview/state.js` → component scripts → `dist/webview/main.js`), since component scripts consume `window.CodaStore` before the main bundle runs. Old `state.js` removed.
- **Typed webview globals**: new `webview/src/globals.d.ts` declares `window.CodaStore`, `window.CodaVscode` (bridge, state, `OutgoingMessage`-typed `postMessage`) and the `_codaLastTurn*` accessor globals — future TS modules get full type safety against the extension-host boundary.
- **Quality gate**: `quality:ci` now runs `typecheck:webview` (strict TS check of `webview/src/**`) alongside the extension typecheck, so webview regressions fail CI. Version bumped to 1.25.27.

## 1.25.26

- **Webview CSP hardening**: Removed the `cdn.jsdelivr.net` allowance from the chat webview Content-Security-Policy — `highlight.js` and `marked` were already served from the bundled `lib/` copy, so the CDN entry was a pure supply-chain/offline risk with no benefit.
- **Code Map panel hardening**: Mermaid is now bundled to `dist/mermaid.min.js` at build time and served via `asWebviewUri` instead of loading from `cdn.jsdelivr.net`. The panel also gained a strict CSP (`default-src 'none'` + script nonce) and uses mermaid `securityLevel: 'strict'` (was `loose`).
- **Secrets hygiene**: Removed the stray `.env.local` from the working tree (already gitignored via `.env*`).
- **`commandPolicy` layering fix**: Replaced the hidden `require('vscode')` inside `configuredPolicy()` with a normal static import — the module boundary is now honest and test stubbing works through the standard path.
- **`isSensitiveFile` precision**: Sensitive-path matching is now segment-aware (exact `.ssh`/`.aws`/`.gnupg` segments; absolute-path prefixes for `/etc/*`, `/proc`, `/sys`, `C:\Windows\System32`). Fixes substring false positives such as `src/etc/passwd-manager.ts`, `docs/ssh/keys.md`, `fixtures/.awsx/notes.txt`, and relative workspace folders named `proc`/`etc`.
- **Package size −45% (6.7 MB → 3.69 MB VSIX)**: bundle analysis (new `npm run analyze:bundle`, `scripts/analyze-bundle.js`) revealed the production `extension.js` is only 2.59 MB minified (the 11.3 MB figure was the dev build). The real packaging bloat was: a stale, unreferenced `dist/harnessRuntime.mjs` (8.8 MB uncompressed) shipped in every release; tree-sitter grammars packaged twice (`assets/grammars` + `dist/grammars` — the runtime already falls back to `assets/`); and the dev-only `dist/eval/benchmarkRunner.js`. All three are now excluded via `.vscodeignore`.
- **E2E smoke tests** (`npm run test:e2e`): new `@vscode/test-electron` harness (`tests/e2e/`) that launches a real VS Code instance with Coda loaded and verifies the packaging-critical contract: activation, core command registration, chat webview open, and workspace-root visibility — no API key required. Reuses the installed VS Code stable when available (falls back to a cached download), so it runs offline-friendly in CI. Not part of the fast `quality:ci` gate; run via `npm run quality:full`.
- **Refactor**: Extracted command groups from `extension.ts` (1,731 → 897 lines) into a new `src/commands/` directory: `scm.ts` (commit message, agent commit, PR package, review artifact), `scmReview.ts` (multi-file review picker), `scmHook.ts` (pre-commit hook installer), `mcp.ts` (marketplace, status, restart, add), `checkpoints.ts` (restore checkpoint, per-step revert), and `memory.ts` (conversation export, audit-log export, memory manager — with chat-provider/audit-log dependencies injected to keep the module decoupled). MCP helpers were hoisted from function-local to module scope, `coda.mcpAdd` now reuses the shared `readWorkspaceMcpConfig`/`writeWorkspaceMcpConfig` helpers instead of duplicating config I/O inline, and the checkpoint quick picks share a timestamp formatter. Command-contract and regression tests updated to scan the new modules via a shared `readExtSrc()` helper.
- **Webview TypeScript migration, slice 1**: `webview/main.js` is now bundled by esbuild (entry `webview/main.js` → `dist/webview/main.js`), enabling typed ES modules under `webview/src/`. First converted module: the security-sensitive outgoing-message allow-list is now a typed bridge (`webview/src/messages.ts` — `OutgoingMessageTypes` const array + `createVscodeBridge`), typechecked via a dedicated strict `webview/tsconfig.json` and linted as TypeScript. Source-scanning tests read the combined bundle source via a shared `readWebViewSrc()` helper, so the allow-list contract follows the code as more modules migrate.
- **Tests**: Added 3 regression tests for `isSensitiveFile` (positive, lookalike-negative, relative-negative). Suite is now 254 tests, all passing; lint, typecheck, and build verified green.

## 1.25.21

- **Tool protocol integrity**: Detects provider tool calls that finish without a tool result or approval response and stops before sending invalid history to the next provider turn, preventing `AI_MissingToolResultsError`.

## 1.25.20

- **Unified chat surfaces**: Routes the native `@coda` participant through the same agent and chat backend as the Coda panel, including shared tools, approvals, sessions, change tracking, and run status.
- **Agent reliability**: Persists and restores interrupted runs, tool lifecycle state, terminal failures, and compacted conversation history without losing tool results.
- **Context handling**: Adds proactive model-aware compaction before provider turns while preserving structured tool-call/result history.
- **UI polish**: Keeps setup guidance visible when configuration is missing and disables recovery controls when no run is active.
- **Publishing**: Refreshes onboarding, provider setup guidance, native chat descriptions, and production packaging metadata.

## 1.25.17

- **Native Chat entry point**: Changes the Coda header icon to open VS Code's built-in Chat pane with `@coda` selected, matching the Codex/Copilot interaction model instead of launching a separate webview panel.

## 1.25.16

- **Deterministic right-side launch**: Removes the contributed Coda side-bar view that VS Code could persist inside Explorer. The header icon now opens Coda as a dedicated right editor-column webview panel, avoiding the bottom-left/Explorer placement issue.

## 1.25.15

- **Fresh right-side view identity**: Replaces the old persisted `coda.chatView` contribution with `coda.rightChatView` under a new Secondary Side Bar container so upgraded installs no longer reopen Coda inside Explorer.

## 1.25.14

- **Header button reliability**: Fixes the Coda header icon click action so it focuses the Secondary Side Bar and the `coda.chatView` view directly, with fallbacks for VS Code command differences.

## 1.25.13

- **Header access button**: Adds a Coda icon button to the editor title/header area so the right-side Coda chat can be opened directly without hunting through the Command Palette.

## 1.25.12

- **Publish-ready icon update**: Uses the provided Coda PNG for the Marketplace icon and keeps the right-side Coda UI/secondary side bar placement from 1.25.11.

## 1.25.11

- **Right-side Coda UI and unified branding**: Moved the Coda chat view from the left Activity Bar to VS Code's secondary side bar so Explorer and Source Control stay visible. Standardized VS Code UI branding on the provided `assets/coda.svg`, generated the required marketplace PNG from that SVG, and removed the old duplicate icon/logo assets.

## 1.25.10

- **Agent reliability and publishing readiness**: Completed the P0/P1/P2 fix-plan alignment across AI SDK streaming/tool approvals, retry/backoff, compaction retention, edit safety, search invalidation, durable resume, auto-permission safeguards, provider IDs, memory display/deletion, test generation paths, and dirty-buffer reads. Verified with typecheck, full regression tests, benchmarks, and production build.

## 1.25.9

- **Pollinations connectivity correction (regression in 1.25.8)**: My earlier "verification" was flawed — it used a raw `curl` without an `Authorization` header, which (transiently) returned a completion. Re-tested through the actual AI SDK client path: Pollinations now **requires an API key** (returns `401 A valid API key is required` for keyless, empty, and bogus bearer tokens; the AI SDK also rejects empty keys client-side). Changed Pollinations to `needsKey: true` and updated the Quick Start option accordingly, so users are prompted for a (free) Pollinations key instead of silently failing. Live connectivity for keyless is no longer possible.

## 1.25.8

- **Provider connectivity fix — Pollinations (default free provider)**: It was pointed at the deprecated `text.pollinations.ai/v1` endpoint with model `gpt-4o-mini`, which now returns `404 Model not found` — so the free default was completely non-functional. Switched to the current `gen.pollinations.ai/v1` endpoint (verified working keylessly via a live request) and updated the default model to `openai` with a refreshed, current model list (`openai`, `openai-fast`, `gpt-oss`, `gemini`, `mistral`, `command-a-plus`, `qwen-coder`). Fixed the matching Quick Start free-option model.
- **Connectivity verification (this release cycle)**: Confirmed no deprecated AI SDK factory usage anywhere in the codebase (all providers use `createOpenAI`/`createAnthropic`/`createGroq`/`createAzure` etc.). Verified the changed model IDs/endpoints against current provider docs: DeepSeek (`deepseek-chat`/`deepseek-reasoner`, base URL without `/v1`), GitHub Models (correctly retired), Anthropic (`claude-sonnet-4-6`/`claude-opus-4-6` are real 2026 models), and Groq (`openai/gpt-oss-120b` is the correct ID).

## 1.25.7

- **Model catalog refresh (correct IDs)**: Anthropic default is now `claude-sonnet-4-6` (+ 4.6/4.5/legacy entries); Cohere, DeepSeek, Groq, and GitHub Models IDs normalized to current model IDs via per-provider `normalize*ModelId` helpers. Fixed DeepSeek base URL (dropped the stray `/v1`).
- **GitHub Models retired**: Marked retired (new endpoint `https://models.github.ai/inference`, free tier off), removed from the Quick Start free options and Settings free wording; `resolveBaseUrl` auto-migrates the old Azure endpoint to the new one. Relevant docs/README updated.
- **Stream/empty-run robustness**: AI SDK stream errors now propagate as real errors (previously swallowed as warnings), and agent runs that finish with no text and no tool output throw a clear error instead of silently succeeding. `streamEnd` is emitted only on success, and the webview `error` handler fully resets UI state (no hung "streaming" state).
- **Quieter logs**: Missing optional workspace files (`package.json`, `memory.json`, `.codaignore`) no longer emit scary `ENOENT` warnings.

## 1.25.6

- **Model selection keeps smart routing in sync**: Selecting a model via `Coda: Configure` or the in-webview picker now updates `model` and, when smart routing is enabled, all routing overrides (`chatModel`/`agentModel`/`inlineModel`/`reviewModel`) through a new `updateSelectedModel` helper.
- **Explicit mode only**: Removed the automatic chat→agent mode switching (imperative-verb detection + toast). The chat input now strictly honors the mode you've selected.
- **Destructive-action confirmations**: Rejecting/reverting a file and rolling back all tracked changes now show a `window.confirm` prompt to prevent accidental loss.
- **Build & Debug presets in UI**: The in-webview preset dropdown now offers `Build` and `Debug` (server-side support already existed via `/mode`).
- **Surfaced swallowed setup errors**: MCP init failures and recent-files collection errors now log (`logWarn`) and surface a visible warning/error instead of being silently swallowed.

## 1.25.5

- **Model-fetch errors surfaced in the UI**: `Coda: Configure` / provider model list now catches failed model fetches and posts a clear error to the webview (e.g. "Could not fetch models for X" or "No API key configured for X") instead of silently leaving an empty list. The in-webview model dropdown shows the error instead of a bare "No models found".
- **Agent error logging**: `AgentRunner` now logs unexpected (non-abort) errors via `logError`, so agent failures are no longer silent.

## 1.25.4

- **Provider settings fully synced to the catalog (36 providers)**: The `Coda: Provider` dropdown now lists every catalog provider — including `gateway`, `together`, `fireworks`, and the 20 enterprise / OpenAI-compatible providers added earlier — in catalog order. A new regression test enforces catalog ↔ settings alignment and per-provider adapter registration, so the two can never drift again.
- **`Coda: Configure` refactor**: Extracted `configureProviderConnection`; the command now accepts a preferred provider and only prompts for a base URL when the provider actually needs one (`providerNeedsBaseUrlPrompt`). In-webview "Change provider" now routes through `coda.configure` so it runs the full connection + model-fetch flow instead of just setting the key.
- **Azure adapter** now passes `baseURL` to `createAzure`, so a custom Azure endpoint is respected.
- **Ollama Cloud** correctly marked `needsKey: true` (it requires an API key).
- **Fixed** the inverted "missing API key" warning in the connection check (it previously warned only when a key was *not* needed).
- **README/eslint**: clarified that only endpoint-requiring providers prompt for a base URL; added `NodeFilter` browser global.

## 1.25.3

- **Quick Start button fix**: The in-chat "Quick Start" button silently did nothing when a provider was already configured (it just opened the sidebar). It now always opens the free-first provider picker, so the button reliably configures Coda and lets an already-configured user switch to a free model (Pollinations, GitHub Models, Ollama, Gemini/Groq/DeepSeek free tiers).

## 1.25.2

- **Settings fix — all 36 providers now configurable**: `Coda: Configure` only collected a base URL for the OpenAI-compatible and Ollama providers, so cloud endpoints (Snowflake Cortex, Cloudflare AI Gateway/Workers AI, GitLab, SAP AI Core, Kilo, ZenMux, Dynamic) and Azure/Vertex endpoints could not be entered. Added a `coda.providerBaseUrls` map (one endpoint per provider) so setting one provider's URL never leaks into another; the configure flow, model list, and connection check all resolve per-provider now.

## 1.25.1

- **Marketplace README update**: Documented the full 36-provider lineup (commercial clouds, open-weight hosts, local/free tiers, and OpenAI-compatible/gateway endpoints), added key-source links for the new enterprise providers, and refreshed the "broad provider support" highlight. No code changes.

## 1.25.0

- **Provider parity with opencode (architecture + coverage)**:
  - **Single provider catalog**: `src/agent/providers.ts` is now the one source of truth for every provider (metadata, key requirements, base URLs, default models, free-tier). The command quick-pick, webview selector, health checks, model factory, and free-tier badge all derive from it.
  - **Central config resolver**: `src/agent/configResolver.ts` unifies API-key / base-URL / mode resolution across chat, inline chat, inline completion, and the agent graph.
  - **Unified permission engine**: `src/agent/permissionPolicy.ts` centralizes `allow`/`deny`/`ask`/`defer` decisions; the agent graph is now a thin adapter over it.
  - **Per-provider adapter modules**: `src/agent/providerPlugins/` holds one module per provider (opencode-style plugin registry). `createModel` and `fetchModels` delegate to it, removing the large switch statements.
  - **Provider coverage**: added **Azure OpenAI**, **Google Vertex AI**, **Perplexity**, **Cerebras**, **Alibaba (DashScope)**, **DeepInfra**, **NVIDIA NIM**, **Venice**, **LLM Gateway**, **GitHub Copilot**, **Cloudflare AI Gateway**, **Cloudflare Workers AI**, **GitLab Duo**, **SAP AI Core**, **Kilo**, **Dynamic**, **ZenMux**, and **Snowflake Cortex** (custom `LanguageModelV4` provider). Coda now supports 36 providers, covering all meaningful opencode providers.

## 1.24.0

- **Per-hunk diff accept/reject**: Diff cards in chat now expose granular review controls. Alongside the existing per-file **Keep** / **Undo**, each changed file can be expanded to accept or reject individual hunks. Hunk boundaries come from the same `generateDiff` the agent's accept/reject logic indexes into (requested live via `requestHunks` → `hunkList`), so indices always line up with what's on disk. Accepting a hunk keeps the rest of the file editable; rejecting a hunk reverts just that change. README updated with the new capability and a stronger marketplace pitch.

## 1.23.1

- **UX polish (market-leader parity)**:
  - **Per-turn "Sources" chips**: assistant turns now show which workspace files were retrieved as context (Perplexity/Claude-style), surfaced from the BM25 retrieval that already powers each reply.
  - **In-conversation search**: `Cmd/Ctrl+F` opens a find bar that highlights matches across the chat and steps through them.
  - **Inline confirmation**: editing a message that discards later turns now uses a non-blocking in-chat confirm toast instead of a native browser dialog.
  - **Accessibility**: theme-aware `:focus-visible` ring, `prefers-reduced-motion` support, and an ARIA live region on the agent status pill.
  - **Theme hardening**: remaining hardcoded diff/terminal colors now resolve from VSCode theme tokens, so the UI adapts cleanly to Light / High-Contrast themes.

## 1.23.0

- **Opencode-style retrieval (no embeddings)**: Removed the entire embeddings pipeline — embedding model factory (`createEmbeddings`/`CodaEmbeddingModel`/`EMBEDDING_LIMITS`/`DEFAULT_EMBEDDING_MODELS`), the `@orama/orama` vector index, `initEmbeddings`/`embedAllChunks`, and the binary embedding cache. Code retrieval is now pure **BM25 over symbol-boundary chunks + workspace-symbol (LSP) lookups**, exactly like opencode. `hybridSearch` keeps recency and exact-identifier boosts; zero network calls, zero token cost, fully offline.

## 1.22.0

- **Free-model providers (opencode-style "free models just work")**: Added **Pollinations** (completely free, no API key, no install — instant) and **GitHub Models** (free with your existing GitHub token) as first-class providers via the OpenAI-compatible path.
- **Free-tier-aware Quick Start**: `Coda: Quick Start` now leads with a free-first choice (Pollinations, GitHub Models, Ollama, Gemini/Groq/DeepSeek free tiers) before "bring your own key".
- **"Free" affordance**: A `Free` badge shows in the chat-header mode pill and status bar when the active provider is in the free tier; `FREE_TIER_PROVIDERS` exported from `llmFactory` drives it.
- Provider enum descriptions now note which providers are free/free-tier.

## 1.21.0

- **Mode & preset indicator**: Status bar item plus an in-chat pill showing the active mode (Plan / preset) and permission state, updating live with `coda.planMode`, `coda.agentPreset`, and `coda.permissions`.
- **Auto permission mode**: New `coda.permissions` setting (`ask` | `auto`). `auto` trusted mode auto-approves every non-blocked tool without prompting; plan-mode denials and the command blocklist remain enforced.
- **Chat UI contract repair**: Greeting replies and per-hunk accept/reject now render; `ExtensionToWebviewMessage` union reconciled with runtime and enforced at compile time.
- **Per-step revert**: Revert a single agent step from the checkpoint panel or the `coda.revertCheckpointStep` command.
- **Auto-titling**: Conversation titles now ignore a leading `/slash` command.

## 1.18.3

- **Fix: Diff Renderer Hunk Grouping**: Resolved a bug where disjointed edits separated by more than `2 * CONTEXT_RADIUS` lines were incorrectly grouped into a single giant hunk with broken line numbers.
- **Fix: Chat View Message Duplication**: Prevented potential message duplication/overlap when reconstructing session messages for long conversation histories.
- **Fix: Progress Summary Failed Tool Filtering**: Filtered out failed tool calls from the auto-computed progress summary to ensure accurate step counts.

## 1.18.2

- **Stateful Agent Session Pause/Resume**: Captures full LangGraph agent execution state (complete message buffer, step count, tool action log, provider/model configuration, and auto-computed progress summary) for reliable pause/resume across step-limit hits, user stops, and error recovery.
- **Structured Diff Rendering**: Added an enhanced diff view in the webview with per-hunk accept/reject buttons, word-level diff highlighting, and side-by-side/unified view toggles.

## 1.18.0

- **Streaming Command Output**: Real-time streaming of command execution output in the agent panel.
- **Auto-Checkpointing**: Automatic checkpointing and checkpoint management for agent runs.

## 1.17.3

- **Fix: empty final response across all providers.** When the agent ran tools but never produced a final text answer (a common Gemini / OpenAI / Anthropic failure mode after long tool-heavy runs), Coda now issues one provider-agnostic follow-up call to the unbound model asking it to summarize — so you get a real analysis instead of a synthetic "Completed N of M tool call(s)" bullet list.
- **Fix: token counter showing `0 in`.** The token aggregator now reads all four LangChain provider conventions: `usage_metadata` (OpenAI / Anthropic / Gemini / DeepSeek), `response_metadata.usage` (`prompt_tokens` / `promptTokenCount` / `input_tokens`), `response_metadata.tokenUsage` (older integrations), and `additional_kwargs.usage` / `token_usage` (OpenRouter, OpenAI-compatible endpoints). Input count is now accurate for every supported provider.
- **Improved fallback message.** If the summary call also fails, the UI now shows the last meaningful tool result excerpt instead of just a list of tool names.

## 1.17.2

- **UI: diff card buttons restyled.** Per-card `Accept` / `Reject` buttons were rendering as unstyled browser defaults; they now match the rest of the panel as pill-shaped ghost buttons with green/red accents on hover and after action.
- **UI: `Keep` / `Undo` change-bar buttons toned down.** Switched from the loud theme primary colour to a subtle green pill for `Keep` and a ghost button for `Undo`, matching the chip / action aesthetic used elsewhere.

## 1.17.1

### Bug fixes

- **Critical: slash commands no longer freeze the agent.** Fixed a mutex deadlock where every slash command (`/plan`, `/run`, `/ship`, `/pr`, `/codebase`, `/parallel`, `/swarm`, `/init`, `/preview`, `/test`, `/fix`, `/review`, `/commit`, `/resume`, etc.) would acquire the chat lock, then call back into the chat handler which tried to acquire the same lock — hanging silently with no response, no error, and no processing.

## 1.17.0

### Reasoning visualization

- **New: streaming Thought blocks for reasoning-capable models.** When a model has a hidden reasoning channel (Anthropic extended thinking, OpenAI o1 / o3, Google Gemini 2.5 / 3.x, DeepSeek R1), Coda now streams the model's thoughts into collapsible **Thought** sections inline with the agent narrative — so you can see *why* the agent decided what it did, not just the result.
- **Gemini thinking enabled by default.** Coda now opts in to `thinkingConfig.includeThoughts` for Gemini 2.5 and 3.x models so thought summaries are actually returned by the API, not just opaque signature tokens.
- **No more fake CoT on non-reasoning models.** Models without a reasoning channel (e.g. `gpt-4o`, `gemini-flash` without thinking) no longer produce spurious "Thinking" blocks that duplicated the final answer.
- **Per-provider extractor.** Reasoning text is recognized in all four LangChain provider conventions — Anthropic `thinking` / `thinking_delta` content blocks, OpenAI `additional_kwargs.reasoning_content` and `reasoning.summary` (string or array), Gemini `thought: true` content parts, and DeepSeek R1 `reasoning_content`.

### UI polish

- Inline agent narrative redesigned to a Cursor-style stream: tool chips, thought blocks, and the final answer flow as a single continuous transcript instead of separate panels.
- Token-count footer now always renders a number (provider tokens → reasoning estimate → answer character estimate as fallback).
- Empty thought blocks are removed on seal; sealed thoughts display "Thought for Xs" with the body still expandable.
- Read-chip grouping flushes correctly when interrupted by other tool actions.

### Bug fixes

- Fixed text-content extractor to ignore Gemini parts marked `thought: true` so reasoning text no longer double-counts as answer text.
- Fixed chip insertion to seal any active Thought block before the chip appears, preserving correct ordering.
- Fixed token estimate when the final response has no reasoning tokens to fall back on.

---

Older versions: see git history.
