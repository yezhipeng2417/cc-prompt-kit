# Claude Code Internal Prompts

Raw prompt source files extracted from Claude Code's TypeScript source (512K+ LOC). Each file contains the complete original source with full prompt text.

These are the actual prompts that power Claude Code — the patterns analyzed to build cc-prompt-kit's 5 skills.

## File Index

### Core System Prompts
| # | File | Source | Description |
|---|------|--------|-------------|
| 01 | system-prompt | `constants/prompts.ts` | Main system prompt builder (914 lines) — identity, constraints, tools, style, output |
| 02 | cyber-risk-instruction | `constants/cyberRiskInstruction.ts` | Security testing / CTF / defensive guidance |
| 36 | system-prompt-builder | `utils/systemPrompt.ts` | Priority-based prompt assembly (override > coordinator > agent > custom > default) |
| 37 | system-prompt-sections | `constants/systemPromptSections.ts` | Dynamic section manager (cached vs volatile) |

### Service Prompts
| # | File | Source | Description |
|---|------|--------|-------------|
| 03 | compact-prompts | `services/compact/prompt.ts` | Context compression — full and partial summary templates |
| 04 | session-memory-prompts | `services/SessionMemory/prompts.ts` | Session notes template + update instructions |
| 05 | magic-docs-prompts | `services/MagicDocs/prompts.ts` | Auto-updating project documentation |
| 06 | memory-extraction-prompts | `services/extractMemories/prompts.ts` | Auto-memory extraction (user/feedback/project/reference types) |
| 07 | dream-consolidation-prompt | `services/autoDream/consolidationPrompt.ts` | Memory consolidation during "dream" passes |

### Agent Prompts
| # | File | Source | Description |
|---|------|--------|-------------|
| 08 | agent-tool-prompt | `tools/AgentTool/prompt.ts` | When/how to spawn sub-agents, fork vs fresh, prompt writing guide |
| 09 | general-purpose-agent | `AgentTool/built-in/generalPurposeAgent.ts` | Default agent — search, analyze, investigate |
| 10 | explore-agent | `AgentTool/built-in/exploreAgent.ts` | READ-ONLY codebase search specialist (haiku) |
| 11 | plan-agent | `AgentTool/built-in/planAgent.ts` | READ-ONLY architecture planning (4-phase) |
| 12 | verification-agent | `AgentTool/built-in/verificationAgent.ts` | "Try to break it" — named failure patterns, adversarial probes |
| 13 | claude-code-guide-agent | `AgentTool/built-in/claudeCodeGuideAgent.ts` | Documentation helper for Claude Code/SDK/API |
| 38 | statusline-setup-agent | `AgentTool/built-in/statuslineSetup.ts` | PS1 → statusLine converter |

### Tool Prompts
| # | File | Source | Description |
|---|------|--------|-------------|
| 14 | bash-tool-prompt | `tools/BashTool/prompt.ts` | Shell execution — git safety, dedicated tool preference |
| 15 | brief-tool-prompt | `tools/BriefTool/prompt.ts` | User communication in proactive mode |
| 19 | skill-tool-prompt | `tools/SkillTool/prompt.ts` | Skill invocation and discovery |
| 20 | file-read-tool | `tools/FileReadTool/` | File/image/PDF/notebook reading + malware detection |
| 21 | file-edit-tool | `tools/FileEditTool/` | Exact string replacement editing |
| 22 | file-write-tool | `tools/FileWriteTool/` | File creation with pre-read requirement |
| 23 | grep-tool | `tools/GrepTool/` | Ripgrep-based content search |
| 24 | glob-tool | `tools/GlobTool/` | File pattern matching |
| 25 | web-fetch-tool | `tools/WebFetchTool/` | URL fetching + markdown conversion |
| 26 | lsp-tool | `tools/LSPTool/` | Language Server Protocol operations |
| 27 | todo-write-tool | `tools/TodoWriteTool/` | Task list management |
| 28 | plan-mode-prompt | `tools/EnterPlanModeTool/` | Plan mode entry |
| 29 | exit-plan-mode-prompt | `tools/ExitPlanModeTool/` | Plan mode exit + approval |
| 30 | tool-search-prompt | `tools/ToolSearchTool/` | Deferred tool schema discovery |
| 31 | task-create-prompt | `tools/TaskCreateTool/` | Task creation |
| 32 | task-update-prompt | `tools/TaskUpdateTool/` | Task status updates |
| 33 | ask-user-question-prompt | `tools/AskUserQuestionTool/` | Multiple choice question UI |
| 34 | send-message-prompt | `tools/SendMessageTool/` | Inter-agent messaging |
| 35 | config-tool-prompt | `tools/ConfigTool/` | Settings management |
| 39 | web-search-prompt | `tools/WebSearchTool/` | Web search |
| 40 | notebook-edit-prompt | `tools/NotebookEditTool/` | Jupyter cell editing |

### Special Mode Prompts
| # | File | Source | Description |
|---|------|--------|-------------|
| 16 | chrome-automation-prompt | `utils/claudeInChrome/prompt.ts` | Browser automation — GIF recording, alerts, tabs |
| 17 | coordinator-mode-prompt | `coordinator/coordinatorMode.ts` | Multi-worker orchestration system prompt |
| 18 | undercover-mode | `utils/undercover.ts` | Public/OSS repo safety — no internal info leakage |

## Notable Patterns to Study

- **01**: The `SYSTEM_PROMPT_DYNAMIC_BOUNDARY` cache split pattern
- **08**: "Never delegate understanding" principle for agent prompts
- **10 vs 11**: How READ-ONLY constraints are enforced differently for search vs planning
- **12**: Named failure patterns ("Verification Avoidance", "Seduced by 80%")
- **14**: Tool preference hierarchy (dedicated tools > Bash)
- **17**: Coordinator's "don't peek at fork output" rule
- **18**: Undercover mode — real-world prompt security in action
