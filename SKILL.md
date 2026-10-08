---
name: model-advisor-local
description: Recommends the most token-efficient Claude setup (model + effort level) for a given task BEFORE executing it, so the user does not burn usage limits or API budget on overkill configurations. Works for any subscription (chat, Claude Code) and for the API. Use this whenever the user invokes the skill to plan execution, asks which Claude model or which effort level to use, says they want to save tokens or avoid overkill, or sends the skill with no task attached (in that case, ask what the task is first). Also flags when to enable web search, a larger context window, brainstorming mode, agents/subagents, or to split a big job across separate chats. Always end with the verdict card and a switch-or-proceed question — never silently start the task.
---

# Model Advisor

Pick the cheapest Claude configuration that still does the job well, then let the user decide whether to switch before running. The point is economy: do not default to the strongest model or the highest effort "to be safe" — recommend the lightest setup that the task signals actually justify.

What "cheap" means depends on where the user works: on subscriptions (chat, Claude Code) it is usage limits; on the API it is money per token.

## Models

Model examples as of October 2026. Use the exact names the client shows. If a model below is not available, pick the closest visible one and say so.

- **Haiku 5.5** (`claude-haiku-5-5`) — fastest, cheapest. Quick, mechanical, well-specified work.
- **Sonnet 5.5** (`claude-sonnet-5-5`) — the everyday workhorse and the default choice for most real tasks.
- **Opus 5.5** (`claude-opus-5-5`) — for genuinely hard, ambiguous, novel, or high-stakes work.
- **Fable 5.1** (`claude-fable-5-1`) — above Opus. Available only on higher-tier plans and may cost extra. Fallback only, see Step 2.

## Controls

There is no separate thinking on/off switch in chat or Claude Code. The only reasoning control is the **effort** level, so recommend **model** and **effort** only. Never recommend turning thinking on or off.

Effort levels depend on the client. Recommend only levels the client shows:
- **Chat:** Low / Medium / High / Extra / Max. Max is the strongest.
- **Claude Code:** the same levels plus **Ultra**, above Max, for all three models.
- **API:** set effort explicitly with `output_config.effort` (`low`, `medium`, `high`, `xhigh`, `max`; Extra corresponds to `xhigh`). Defaults differ by model and client, so do not rely on them.

In chat the default is Medium. Move away from the default only when a task signal justifies it.

Plus optional flags: features, modes, chat-splitting.

---

## Workflow

### Step 1 — Is there a task?

If the user invoked this skill **with** a task in the same message, go straight to Step 2.

If the skill was sent **alone**, with no task, ask exactly one short question and stop, in the user's language. For Russian:

> Какая задача? Опиши коротко, что нужно сделать — подберу модель и уровень.

Do not guess or invent a task.

### Step 2 — Read the complexity signals

First identify the host (chat, Claude Code, API, or another client), the available models, and the supported effort levels. Use exact visible names rather than assuming the example versions are available. If something is unavailable, choose a visible equivalent and disclose the substitution. Do not claim to switch models without runtime confirmation. Treat examples as task-sizing guidance, not measured capability or price guarantees.

Score the task against these signals. Most tasks land on Sonnet at Medium; push up or down only on clear signals.

**Push toward lighter (Haiku / Low):**
- Deterministic, one obvious path, no branching logic
- Short output, single file, small edit
- Formatting, typography fixes, simple reformat
- Short translation, lookup, rephrase, a single known-pattern regex
- The user already gave the structure and just needs it filled in

**Push toward heavier (Opus / Extra and above):**
- Ambiguous or under-specified — needs interpretation before acting
- Multi-step with dependencies between steps
- Architecture or design decisions, building a new tool from scratch
- Novel problem with no obvious template
- Many simultaneous constraints (multilingual + formatting + schema + edge cases)
- Unclear root cause (debugging where the error does not point at the fix)
- Mistake is expensive or hard to reverse

If a task pushes even past these signals — genuinely novel research-grade reasoning, or stakes where an occasional miss would be unacceptable — recommend Opus 5.5 at the highest effort the client offers (Max in chat, Ultra in Claude Code) first, and say so plainly. Offer Fable 5.1 only as a fallback, when it is available in the user's plan and Opus 5.5 at that level clearly is not enough, and state that it may cost extra before anything else.

### Step 3 — Pick the model

| Model | Use when |
|---|---|
| **Haiku 5.5** | Task is trivial or mechanical and speed matters more than depth. |
| **Sonnet 5.5** | Default. Standard coding (scripts, Playwright, pandas, openpyxl), content generation, moderate data processing, debugging with a clear error, drafting docs. If unsure between Sonnet and Opus, start with Sonnet. |
| **Opus 5.5** | Genuinely complex: architecture, large refactors, ambiguous multi-constraint problems, novel tooling, deep root-cause debugging, anything where correctness is high-stakes. For the hardest tasks, pair it with the highest effort first. |
| **Fable 5.1** | Fallback only, when available: the task still fails or is clearly beyond Opus 5.5 at the highest effort. |

### Step 4 — Pick effort

| Effort | Use when |
|---|---|
| **Low** | Mechanical, deterministic, well-specified. |
| **Medium** | The default. Routine task with one clear path: standard caption, simple script, clean data edit, standard coding, agentic coding or multi-step tool use with a clear path. |
| **High** | Complex coding, hard or unfamiliar debugging, high-stakes multi-step work. |
| **Extra** | Hard debugging, ambiguous problems, multi-constraint design, large refactors. |
| **Max** | Novel or research-grade reasoning, high-stakes correctness. The strongest level in chat. Recommend rarely. |
| **Ultra** | Claude Code only, above Max. Only when Max was not enough. Recommend very rarely. |

Never pair Haiku with Extra, Max, or Ultra — if a task needs that much reasoning, the model is wrong.

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
- Also recommend it when a chat has accumulated a long investigation/discussion (research, back-and-forth, exploration) and the next step is a well-defined execution task: carry forward only the distilled conclusion (write it to a file first), start execution in a fresh chat so the investigation's context doesn't inflate every subsequent request.

### Step 6 — Output the verdict, then ask

Print the verdict as a fenced block so it is scannable, give a one to two line reason, then ask whether to switch. Write the labels and the text in the user's language. For Russian, use МОДЕЛЬ / EFFORT / ФИЧИ / РЕЖИМ / ЧАТЫ / ПОЧЕМУ.

ALWAYS use this template:

```
MODEL:     <Haiku 5.5 | Sonnet 5.5 | Opus 5.5 | Fable 5.1 (fallback)>
EFFORT:    <Low | Medium | High | Extra | Max | Ultra>
———
FEATURES:  <Larger context | Web search | —>
MODE:      <Brainstorming | Agents | —>
CHATS:     <One chat | Split into N chats>
———
WHY:       <one or two lines: why this, and why not lighter or heavier>
```

If the verdict is Fable 5.1, add one line under the block before the question that it is only available on some plans and may cost extra.

Then ask, in plain text under the block, in the user's language. For Russian:

> Переключаешься на эту конфигурацию или выполнять как есть в текущей?

Do not start the task until the user answers. If they say proceed, continue in the current model.

---

## Examples

**Example 1 — standard automation**
Input: «Напиши Playwright-скрипт, который собирает комментарии с 4 Instagram-аккаунтов»
Output:
```
МОДЕЛЬ:    Sonnet 5.5
EFFORT:    Medium
———
ФИЧИ:      —
РЕЖИМ:     —
ЧАТЫ:      Один чат
———
ПОЧЕМУ:    Знакомый стек, одна чёткая задача — Medium хватит.
           Opus избыточен, Haiku для логики обхода аккаунтов
           рискован.
```

**Example 2 — trivial fix**
Input: «Поправь типографику в этом посте»
Output:
```
МОДЕЛЬ:    Haiku 5.5
EFFORT:    Low
———
ФИЧИ:      —
РЕЖИМ:     —
ЧАТЫ:      Один чат
———
ПОЧЕМУ:    Механическая замена символов. Глубокие рассуждения тут
           только жгут лимит.
```

**Example 3 — complex design**
Input: «Спроектируй архитектуру тула для нормализации каталога на 50к строк в пяти языках»
Output:
```
МОДЕЛЬ:    Opus 5.5
EFFORT:    Extra
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
МОДЕЛЬ:    Sonnet 5.5
EFFORT:    High
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

**Example 6 — after a long investigation, before execution**
Context: a long research/back-and-forth chat just landed on a final decision (e.g. a terminology canon, an architecture choice), and the next step is to build the actual deliverable from it.
Output:
```
МОДЕЛЬ:    Sonnet 5.5
EFFORT:    Medium
———
ФИЧИ:      —
РЕЖИМ:     —
ЧАТЫ:      Разбить: зафиксировать вывод в файл здесь → сборка в новом чате
———
ПОЧЕМУ:    Неоднозначность уже снята выводом расследования — осталась
           понятная работа по известной структуре. Opus был нужен для
           самого расследования, не для сборки по готовому канону.
           Новый чат не тащит за собой всю историю поиска.
```

**Example 7 — Claude Code, English**
Input: "Refactor the auth module across 40 files without breaking the public API"
Output:
```
MODEL:     Opus 5.5
EFFORT:    Extra
———
FEATURES:  —
MODE:      Agents
CHATS:     One chat
———
WHY:       Large refactor with a hard constraint and costly mistakes.
           Extra is enough; Ultra is only worth it if Max stalls.
```
