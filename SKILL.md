---
name: model-advisor-local
description: Recommends the most token-efficient Claude setup (model + effort level + thinking on/off) for a given task BEFORE executing it, so the user does not burn tokens on overkill configurations. Use this whenever the user invokes the skill to plan execution, asks which Claude model or which effort/thinking level to use, says they want to save tokens or avoid overkill, or sends the skill with no task attached (in that case, ask what the task is first). Also flags when to enable web search, 1M-token context / Infinite Chat, brainstorming mode, agents/subagents, or to split a big job across separate chats. Always end with the verdict card and a switch-or-proceed question — never silently start the task.
---

# Model Advisor

Pick the cheapest Claude configuration that still does the job well, then let the user decide whether to switch before running. The point is token economy: do not default to Opus or Max "to be safe" — recommend the lightest setup that the task signals actually justify.

Model examples from the original local configuration (not a verified current catalog):
- **Haiku 4.5** — fastest, cheapest. Quick, mechanical, well-specified work.
- **Sonnet 4.6** — the everyday workhorse and the default choice for most real tasks.
- **Opus 4.8** — only for genuinely hard, ambiguous, novel, or high-stakes work.

Controls to recommend: **model**, **effort** (Low / Medium / High / Extra / Max), **thinking** (on/off). Plus optional flags: features, modes, chat-splitting.

---

## Workflow

### Step 1 — Is there a task?

If the user invoked this skill **with** a task in the same message, go straight to Step 2.

If the skill was sent **alone**, with no task, ask exactly one short question and stop:

> Какая задача? Опиши коротко, что нужно сделать — подберу модель и уровень.

Do not guess or invent a task.

### Step 2 — Read the complexity signals

First identify the host (Claude Code, Claude Desktop, web, or another client), available models, and supported controls. Use exact visible names rather than assuming the example versions below are available. If unavailable, choose a visible equivalent and disclose the substitution. Do not claim to switch models without runtime confirmation. Treat examples as task-sizing guidance, not measured capability or price guarantees.


Score the task against these signals. Most tasks land on Sonnet; push up or down only on clear signals.

**Push toward lighter (Haiku / Low / thinking off):**
- Deterministic, one obvious path, no branching logic
- Short output, single file, small edit
- Formatting, typography fixes, simple reformat
- Short translation, lookup, rephrase, a single known-pattern regex
- The user already gave the structure and just needs it filled in

**Push toward heavier (Opus / Extra-Max / thinking on):**
- Ambiguous or under-specified — needs interpretation before acting
- Multi-step with dependencies between steps
- Architecture or design decisions, building a new tool from scratch
- Novel problem with no obvious template
- Many simultaneous constraints (multilingual + formatting + schema + edge cases)
- Unclear root cause (debugging where the error does not point at the fix)
- Mistake is expensive or hard to reverse

### Step 3 — Pick the model

| Model | Use when |
|---|---|
| **Haiku 4.5** | Task is trivial or mechanical and speed matters more than depth. |
| **Sonnet 4.6** | Default. Standard coding (scripts, Playwright, pandas, openpyxl), SMM content generation, moderate catalog processing, debugging with a clear error, drafting docs. If unsure between Sonnet and Opus, start with Sonnet. |
| **Opus 4.8** | Genuinely complex: architecture, large refactors, ambiguous multi-constraint problems, novel tooling, deep root-cause debugging, anything where correctness is high-stakes. |

### Step 4 — Pick effort + thinking

Thinking and effort may be separate controls in some environments. Recommend only controls exposed by the current application and selected model; otherwise write N/A.

**Thinking:**
- **Off** — only for truly mechanical tasks where reasoning just burns tokens (typography fix, trivial reformat, one-line lookup).
- **On** — everything substantive: any logic, planning, debugging, design, or content that needs structure.

**Effort (when thinking is on):**

| Effort | Use when |
|---|---|
| **Low** | Mechanical, deterministic, well-specified. |
| **Medium** | Routine task with one clear path (standard caption, simple script, clean data edit). |
| **High** | Real coding, familiar debugging, multi-step but well-understood work. This is the realistic default for most of the user's work. |
| **Extra** | Hard debugging, ambiguous problems, multi-constraint design, large refactors. |
| **Max** | Novel or research-grade reasoning, high-stakes correctness. Recommend rarely. |

Never pair Haiku with Extra/Max — if a task needs that much reasoning, the model is wrong.

### Step 5 — Flag features, modes, chat-splitting

Add these only when a signal calls for them; otherwise mark them as `—`.

**Features:**
- **Larger context, if supported** — only when relevant material must be considered together. Process large Excel/CSV files with scripts or batches first; file size alone does not justify a larger model context. Do not equate a context window with unlimited chat history.
- **Web search** — task needs current info: service/library/driver versions, status checks, competitor or market research, anything post-knowledge-cutoff.

**Modes:**
- **Brainstorming** — creative or ambiguous task that needs requirements explored before any code is written.
- **Agents / subagents** — large task that splits cleanly into independent parallel parts.

**Chat-splitting:**
- Recommend splitting into separate chats when one job would balloon context (e.g. processing a huge catalog in chunks, or a multi-phase build), to keep each chat lean and cheaper.

### Step 6 — Output the verdict, then ask

Print the verdict as a fenced block so it is scannable, give a one to two line reason, then ask whether to switch.

ALWAYS use this template:

```
МОДЕЛЬ:    <Haiku 4.5 | Sonnet 4.6 | Opus 4.8>
EFFORT:    <Low | Medium | High | Extra | Max>
THINKING:  <Вкл | Выкл>
———
ФИЧИ:      <1М контекст | Веб-поиск | —>
РЕЖИМ:     <Brainstorming | Агенты | —>
ЧАТЫ:      <Один чат | Разбить на N чатов>
———
ПОЧЕМУ:    <одна-две строки: почему именно так, и почему не дороже>
```

Then ask, in plain text under the block:

> Переключаешься на эту конфигурацию или выполнять как есть в текущей?

Do not start the task until the user answers. If they say proceed, continue in the current model.

---

## Examples

**Example 1 — standard automation**
Input: «Напиши Playwright-скрипт, который собирает комментарии с 4 Instagram-аккаунтов»
Output:
```
МОДЕЛЬ:    Sonnet 4.6
EFFORT:    High
THINKING:  Вкл
———
ФИЧИ:      —
РЕЖИМ:     —
ЧАТЫ:      Один чат
———
ПОЧЕМУ:    Знакомый стек, одна чёткая задача. Opus избыточен,
           Haiku не вытянет логику обхода аккаунтов.
```

**Example 2 — trivial fix**
Input: «Поправь типографику в этом посте»
Output:
```
МОДЕЛЬ:    Haiku 4.5
EFFORT:    Low
THINKING:  Выкл
———
ФИЧИ:      —
РЕЖИМ:     —
ЧАТЫ:      Один чат
———
ПОЧЕМУ:    Механическая замена символов. Размышления тут только
           жгут токены.
```

**Example 3 — complex design**
Input: «Спроектируй архитектуру тула для нормализации каталога на 50к строк в пяти языках»
Output:
```
МОДЕЛЬ:    Opus 4.8
EFFORT:    Extra
THINKING:  Вкл
———
ФИЧИ:      Обработка данных скриптом
РЕЖИМ:     Brainstorming
ЧАТЫ:      Разбить на 2 чата (дизайн → реализация)
———
ПОЧЕМУ:    Много одновременных ограничений и архитектурное решение.
           Сначала brainstorming, потом код отдельным чатом, чтобы
           не раздувать контекст.
```

**Example 4 — debugging that needs current info**
Input: «Почему Electron-приложение крашится на внешнем мониторе»
Output:
```
МОДЕЛЬ:    Sonnet 4.6
EFFORT:    High
THINKING:  Вкл
———
ФИЧИ:      Веб-поиск
РЕЖИМ:     —
ЧАТЫ:      Один чат
———
ПОЧЕМУ:    Причина неочевидна и зависит от версий драйвера/GPU —
           нужен веб-поиск по актуальным багам. Opus пока не нужен.
```

**Example 5 — skill sent alone**
Input: (только вызов скилла, без задачи)
Output:
> Какая задача? Опиши коротко, что нужно сделать — подберу модель и уровень.
