Create a global Codex skill named `claude-codex-translate`.

Purpose:
This skill translates Claude Code-oriented Markdown files into Codex-compatible files.

It should support both:
1. Claude Code subagent Markdown/spec files -> Codex custom-agent TOML guidance or installed Codex custom-agent TOML
2. Claude Code skill/workflow Markdown files -> Codex-compatible SKILL.md files

Install location for this skill:
$HOME/.agents/skills/claude-codex-translate/SKILL.md

Codex compatibility context:
- Codex skills use a folder with a required `SKILL.md` file.
- `SKILL.md` must include `name` and `description` in frontmatter.
- Global user skills should live under:
  $HOME/.agents/skills/<skill-name>/SKILL.md
- Codex custom agents should live under:
  ~/.codex/agents/<agent-name>.toml
- Project-level Codex custom agents should live under:
  .codex/agents/<agent-name>.toml
- Codex project instructions use AGENTS.md, not CLAUDE.md.

Invocation style:
$claude-codex-translate <source-markdown-path-or-url>

Examples:
$claude-codex-translate chatgpt-codex/agents/task-completion-validator.md
$claude-codex-translate /mnt/c/Users/yanzh/Desktop/chatgpt-codex/agents/task-completion-validator.md
$claude-codex-translate https://github.com/example/repo/blob/main/agent.md

Optional invocation style:
$claude-codex-translate <source> --target agent
$claude-codex-translate <source> --target skill
$claude-codex-translate <source> --install
$claude-codex-translate <source> --output <path>

Core behavior:
- Read the source Markdown from a local path, WSL `/mnt/...` path, GitHub URL, raw URL, or other HTTP/HTTPS Markdown source.
- If the source is a GitHub blob URL, convert it to a raw URL before fetching.
- Detect whether the source is primarily intended to create:
  1. an agent/subagent, or
  2. a skill/reusable workflow.
- If detection is uncertain, infer the safest target from the file content and state the assumption.
- Preserve the original intent, wording, tone, structure, and style as much as possible.
- Only rewrite sentences, commands, paths, examples, and steps when necessary to make the file Codex-compliant or Codex-suitable.
- Do not over-modernize, rebrand, simplify, or improve the writing unless required for Codex compatibility.
- Do not change the core purpose of the original file.
- Do not remove useful behavioral instructions just because they came from the Claude version.
- Do not invent unavailable Codex agents, tools, skills, or commands.

Agent conversion rules:
- Claude Code subagent Markdown/YAML/frontmatter format should be rewritten into Codex custom-agent TOML guidance.
- If installing an agent, write:
  ~/.codex/agents/<agent-name>.toml
- The generated Codex agent TOML must include:
  - name
  - description
  - developer_instructions
- Optional TOML fields may include:
  - model
  - model_reasoning_effort
  - sandbox_mode
  - nickname_candidates
- Replace Claude-specific references:
  - CLAUDE.md -> AGENTS.md
  - Claude Code subagent -> Codex custom subagent / custom agent
  - Claude-only commands -> Codex-compatible prompts or commands
  - Claude sibling-agent references -> "recommend spawning equivalent Codex agents if they are configured"
- Mention that Codex subagents must be explicitly spawned by the user.
- Do not assume sibling agents exist.

Skill conversion rules:
- Claude Code skill/workflow Markdown should be rewritten into a Codex-compatible SKILL.md.
- If installing a skill, write:
  $HOME/.agents/skills/<skill-name>/SKILL.md
- The generated SKILL.md must include valid frontmatter:
  ---
  name: <skill-name>
  description: <clear trigger description>
  ---
- The description should clearly explain when Codex should use the skill.
- Keep supporting scripts under:
  $HOME/.agents/skills/<skill-name>/scripts/
- Keep references under:
  $HOME/.agents/skills/<skill-name>/references/
- Keep assets under:
  $HOME/.agents/skills/<skill-name>/assets/
- If the original file describes a reusable workflow, preserve it as a skill instead of incorrectly converting it into an agent.

WSL and `/mnt` requirements:
- This skill must work correctly inside WSL.
- Treat `/mnt/c/...`, `/mnt/d/...`, and other `/mnt/<drive>/...` paths as valid local paths.
- Support Windows-style paths if provided, such as:
  C:\Users\yanzh\Desktop\file.md
  D:\Projects\repo\file.md
- Convert Windows paths to WSL paths before reading or writing:
  C:\Users\yanzh\Desktop\file.md -> /mnt/c/Users/yanzh/Desktop/file.md
  D:\Projects\repo\file.md -> /mnt/d/Projects/repo/file.md
- Correctly handle spaces in paths by quoting variables in shell commands.
- Do not assume the repo is under the Linux home directory.
- Do not break when the current project is under `/mnt/c/...` or `/mnt/d/...`.
- Prefer POSIX shell and Python standard library for helper scripts.
- Avoid dependencies that may not exist on a fresh WSL installation.

Codex compatibility and WSL validation:
- After updating translated instructions, explicitly verify that they are Codex-compatible.
- Confirm the updated instructions can be followed from WSL workspaces under `/mnt/c/...`, `/mnt/d/...`, or another `/mnt/<drive>/...` path.
- Confirm install targets use the Linux `$HOME` for global Codex agents and skills, not a Windows-mounted home path, unless the user explicitly asks for project-local output.
- Validate the updated instructions with the `karen` Codex agent when Karen is available and the environment supports spawning custom agents.
- If Karen is unavailable, perform the same verification checklist directly and clearly state that Karen did not run.
- Do not claim Karen validated the instructions unless Karen actually ran.

Output behavior:
- By default, edit or generate a converted Markdown/TOML file beside the source file using a safe suffix:
  - <name>.codex.md for translated Markdown
  - <name>.toml for Codex agents
  - SKILL.md inside a skill folder for Codex skills
- Only overwrite the original source file if the user explicitly asks for in-place editing.
- Only install into ~/.codex/agents or $HOME/.agents/skills if the user uses `--install` or explicitly asks to install.
- Always show the final output path.

Validation:
- For generated TOML agent files, validate TOML syntax.
- For generated SKILL.md files, validate that frontmatter exists and includes:
  - name
  - description
- Check that the output no longer instructs users to create Claude Code-only agents or skills.
- Check that Codex paths are correct.
- Check that WSL `/mnt/...` paths are handled safely.
- If a `karen` Codex agent exists, spawn Karen to verify the translated file.
- If Karen is unavailable, perform the same verification checklist directly and clearly say Karen was unavailable.
- Do not claim Karen verified the work unless Karen actually ran.

Also create an optional helper script:
$HOME/.agents/skills/claude-codex-translate/scripts/read_markdown_source.py

The helper script should:
- Accept one argument: source path or URL.
- Convert GitHub blob URLs to raw URLs.
- Convert Windows paths to WSL `/mnt/<drive>/...` paths.
- Read local files.
- Fetch HTTP/HTTPS sources.
- Print Markdown content to stdout.
- Use only Python standard library.

After creating the skill, show me:
1. The final file tree.
2. The full SKILL.md content.
3. The helper script content.
4. One example command using:
   $claude-codex-translate chatgpt-codex/agents/task-completion-validator.md --target agent
5. One example command using:
   $claude-codex-translate chatgpt-codex/skills/example-skill.md --target skill
