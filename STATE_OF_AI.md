# Free AI

## Code Complete

As of 24-07-2026 there is no free unlimited code complete in vscode. Intellicode is long dead, I intend to make something like it, but it is a huge undertaking. And even then I intend to do that for Zed probably. Also thought about making a fork of cline or kilo but for ACP.
I checked most but these are off the top of my head: Gemini Code Assist, Jetbrains Assistant, Windsurf, continue.dev etc.
Current solution is llama.vscode with local FIM models, but this costs resources. Or us the IDEs Antigravity, Windsurf and Jetbrains IDEs.

Current Free Agentic/API model providers:
Opencode,
Kilo Code,
Cline,
Nvidia Nim,
Openrouter,
Antigravity cli (low usage),
cursor (auto is free which is composer)
codex (Unsure, I believe older models are currently free, don't quote me though)

Online chats are nearly all free, perhaps something along the lines of <https://github.com/NiteshSingh17/apibeam> could be a solution, but may risk breaking TOS, so you know, don't use services that require a login, then perhaps using some free VPNs, since you will be tracked using these chats anyway.

Best cheap AI:
ChatGPT, gives the most usage of all providers
Opencode Go (ClinePass seems to be a new option in this category)

## Current thoughts

I like RPI (Research-Plan-Implement) I tend to research frameworks and such before hand, usually using the web-chats to not waste my tokens.
I use a TDD skill and I also mutation test the tests and use codebase complexity analyzers such as fallow and cargo-crap, then pipe results into the LLM.
I provide fallow and cargo commands in agents.md.
I prefer code to be sequential, readable top to bottom. Minimize jumping between files, but LLMs don't tend to respond well to it as an instruction.
I am thinking of adding 2 adversarial Reviewers with different models (clean context) So it becomes RPIR like what is done with the bun rewrite, apparently they give a good result. Haven't done that yet.
I am currently using Opencode as my main driver, I like the pi ecosystem, dislike the lack of subagents and plan mode.
I am currently using coderabbit and codex for pr reviews.
I am using googles devtools for debugging and navigating pages.

Flow:
Plan mode -> Grill me -> Implement TDD -> mutation test -> adversarial review

Note: When I need a persistent plan openspec is my go to i.e. if i need the ability to hand off to a different agent etc.

Apparently you are supposed to automate things with workflows, but it is incredibly hard to know what is going on with grifters out there and nonsense AI generated videos, so so far I am keeping it simple, I dunno if theo t3ddog is using n8n or something.

Even though it is hated due to its vibe coded nature it has some of my opinions: <https://docs.github.com/en/copilot/tutorials/optimize-ai-usage>

## Need to update

Must:
I intend to port ddgrs to npm so I can make it an npx skill and use that instead of exa, since skills are more portable than plugins or MCP.
Update TDD skill to decrease mutations before mutation testing, since mutation testing is time consuming and the agent needs to do it again, meaning more expensive than a proper instruction flow. And No mutations is not a good instruction as an LLM doesn't understand what a mutation is or what a stale test is.

~~Want to see if I can make a fork of pi-blackwhole in v2 of opencode since v1 doesn't have the proper APIs.~~ - OpenCode v2 has a "Good enough" compaction system, not great, but not worth all the work for minimal improvement

Make TUI for dsh using dsh-tui and OpenCode rust port
Port blackwhole to dsh

Need to switch to Obsidian and migrate my docs, migrate to brave origin and uninstall edge. Install harper too.
