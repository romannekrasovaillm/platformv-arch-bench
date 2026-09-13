# Platform V Architecture Benchmark: Spine vs универсальные кодовые харнессы

> Профессиональный бенчмарх архитектурных задач по документации **Platform V
> (СберТех)**: измерение роли **харнесса и формата Spine** по сравнению с
> универсальными кодовыми харнессами (Claude Code, Kimi Code, Theseus,
> OpenClaw, Qwen Code, pi-coding-agent и др.).
>
> **831 прогон · 24 задачи · 19 условий · 4 модели · LLM-судья с
> верификацией цитат · пререгистрация гипотез**
>
> Статус: снапшот 13.09.2026 — 831/973 ячеек завершено и отсуждено
> (arch-руки claude/kimi и glm-расширение до 16 задач частичные, D17).

---

## 🇷🇺 Русский

### Что это

Бенчмарк отвечает на вопрос, который встаёт перед каждым архитектором,
внедряющим агентные инструменты: **даёт ли специализированный харнесс
архитектора (Spine, banking edition) и его формат рабочих артефактов
(architecture-spine, CONSTRAINTS.yaml, fitness-гейты) измеримый выигрыш
в качестве архитектурных документов против универсальных кодовых агентов?**

24 архитектурные задачи построены строго по официальной документации
Platform V (Pangolin DB, Corax, Works::Architect Hub, DataMarts, SEI и др.):
целевая архитектура платёжного хаба, CDC-pipeline, комплаенс-маппинг
719‑П/683‑П/851‑П, защищённый контур, DR, ёмкостное планирование и т.д.
Каждая задача — это `TASK.md` (роль, deliverables D1–D10, ограничения),
`CONTEXT.md` (факты о продукте с метками `[DOC]`/`[BENCH]`/`[ASSUME]`,
ландшафт, INF/NFR/SEC-требования) и `RUBRICS.md` (взвешенные критерии 0–4
и правила hard-fail).

### Главный вопрос и гипотезы (пререгистрированы до прогонов)

- **H1**: Spine со спайн-пакетом (`spine-arch`) превосходит кодовые
  харнессы без кастомизации (`*-plain`).
- **H2**: кастомизация кодовых харнессов под архитекторов (arch-контекст)
  помогает, но не догоняет Spine.
- **H3**: эффект Spine зависит от модели.

### Ключевые результаты

**Сводка по условию × модели** (балл LLM-судьи 0–100, выше = лучше):

> **Обозначения условий:** `spine-arch` — харнесс Spine + спайн-пакет
> (architecture-spine + CONSTRAINTS.yaml, fitness-гейт) — «харнесс и
> формат»; `spine-min` — тот же харнесс Spine **без** спайн-пакета,
> минимальный системный промпт — изолирует вклад харнесса;
> `spine-arch-think` — `spine-arch` с включённым ризонингом модели
> (бюджет 64K); `<харнесс>-plain` — универсальный кодовый харнесс
> (Claude Code, Kimi Code, Theseus, OpenClaw, Qwen Code, omp)
> **в заводской конфигурации**, без архитектурной кастомизации;
> `<харнесс>-arch` — тот же харнесс с arch-кастомизацией
> (AGENTS.md/CLAUDE.md для архитекторов); `raw-llm` — голая модель
> (одиночный API-вызов, без агентного контура).

| Условие | DeepSeek V4.1 Flash | GLM-5.3 Flash | DeepSeek V4 Pro |
|---|---|---|---|
| **spine-arch** (Spine + формат) | 78.2 ± 26.3 | **98.7 ± 2.0** | 73.4 ± 24.2 |
| **spine-min** (только харнесс) | 79.4 ± 22.1 | **99.1 ± 2.1** | — |
| **spine-arch-think** (Spine + ризонинг) | **94.3 ± 14.7** | — | — |
| claude-plain (Claude Code) | 91.9 ± 14.3 | 89.6 ± 21.5 | 96.8 ± 3.9 |
| kimi-plain (Kimi Code) | **94.7 ± 9.1** | 95.7 ± 5.1 | — |
| claude-arch (Claude Code + кастом.) | 92.5 ± 15.7 | 95.3 (n=6) | 92.1 (n=20) |
| kimi-arch (Kimi Code + кастом.) | 90.7 (n=18) | 96.4 (n=11) | 92.8 (n=20) |
| openclaw-plain | 93.7 ± 9.3 | — | — |
| omp-plain | 93.5 ± 14.5 | — | — |
| qwen-plain | 91.3 ± 9.7 | — | — |
| theseus-plain | 72.1 ± 39.6 | 75.2 ± 42.7 | — |
| raw-llm (голая модель) | 77.0 ± 14.1 | 93.5 ± 6.5 | 83.9 ± 6.1 |

**Эффекты** (разность средних, bootstrap 95% CI, dsf-канал):

| Сравнение | Δ баллов | Что это значит |
|---|---|---|
| spine-arch − claude-plain | **−8.0 [−15.2; −0.9]** | Spine без ризонинга уступает Claude Code |
| spine-arch − kimi-plain | **−11.5 [−18.1; −5.3]** | …и Kimi Code |
| spine-arch − spine-min | −0.5 [−8.5; +7.5] | вклад формата отдельно от харнесса ≈ 0 |
| **spine-arch-think − spine-arch** | **+10.9 [+3.8; +18.2]** | ризонинг даёт Spine +11 баллов |
| **spine-arch-think − theseus-plain** | **+21.7 [+10.7; +32.8]** | Spine + ризонинг значимо сильнее Theseus |
| spine-arch − theseus-plain \| **glm** | **+23.5 [+1.4; +55.2]** | на GLM Spine — топовый харнесс |

![Средний балл по условиям](report/01_total_by_condition.png)
![Эффекты с доверительными интервалами](report/02_effects_forest.png)

### Честная интерпретация

1. **Роль Spine — модель-зависима (H3 подтверждена).** На GLM-5.3-Flash
   Spine — **на уровне лучших универсалов**: на первых 8 задачах лидировал
   (98.7/99.1), но при расширении до 10 задач преимущество над Claude Code
   (+4.8 [−7.8; +19.0]) и Kimi Code (−1.3 [−10.8; +4.7]) перестало быть
   статистически значимым; spine-min сохраняет небольшое значимое
   преимущество (+8.9 [+1.7; +20.6] и +2.9 [+0.3; +5.4]).
   На DeepSeek V4.1-Flash без ризонинга Spine уступает обоим универсалам.
2. **Формат спайна отдельно от харнесса не конвертируется в баллы**
   (spine-arch ≈ spine-min): одного документального контекста недостаточно —
   нужна связка «формат + ризонинг».
3. **Ризонинг — главный усилитель Spine на DeepSeek** (+10.9), снимающий
   отставание от универсалов.
4. **Кастомизация универсалов под архитекторов (H2) эффекта не дала**:
   Theseus −4.3 [−19.0; +10.3], Claude Code +1.1 [−5.0; +6.9].
5. Побочные находки: hard-fail rate у Theseus 30–33% (обрезка на лимите
   ходов); зафиксирована несовместимость arch-be × glm-5.3-flash в
   агентном режиме на части задач (D14).

![Тепловая карта по задачам](report/03_task_heatmap.png)
![Completeness и hard-fail](report/04_completeness_hf.png)
![Время прогона](report/05_time.png)

### Методология

- **Матрица**: 973 ячейки (завершено 831) = 24 задачи × 19 условий ×
  до 2 повторов × 4 модели (DeepSeek V4.1 Flash / V4 Pro, GLM-5.3 Flash /
  5.3). kimi×glm прогонялся через OpenRouter (та же модель, D17).
- **Условия**: spine-arch, spine-min, spine-arch-think, theseus-plain/arch,
  claude-plain/arch, kimi-plain, openclaw-plain, qwen-plain, omp-plain,
  raw-llm + свип dsh/codewhale/hermes (по 2 задачи).
- **Судья**: deepseek-v4-pro, анонимизированные ответы, JSON-вердикт по
  рубрикам, **верификация цитат** (доля неверифицированных evidence
  публикуется), hard-fail → потолок 39 баллов. Полная формула — в
  `runners/judge.py`.
- **Пререгистрация**: `PREREGISTRATION.md` (+ хэши задач
  `prereg_hashes_v2.txt`). Все отклонения задокументированы:
  `DEVIATIONS.md` (D1–D14).
- **Воспроизводство**: `runners/` — подготовка ячеек
  (`prepare_cells.py`), прогон (`run_matrix.py`), механические скореры
  (`mech_score.py`), судья (`judge.py`, пул — `judge_merge.py`), анализ
  (`analyze.py`), отчёт (`report_docx.py`). Полный отчёт с диаграммами:
  `report/otchet_platformv_arch_bench_20260913_1454.docx`.

### Ограничения

- Судья одиночный (deepseek-v4-pro) из семейства одного из решателей —
  смещение декларируется, но не измерено (пул судей отменён по стоимости).
- Сравнение dsf↔glm (H3) — на пересечении 8 задач; glm53-канал не закрыт.
- 2 ячейки из 750 выведены из-за хронических таймаутов (D14).
- Результаты — про режим «один документ на задачу»; они не оценивают
  многошаговую работу архитектора (гейты, трассировка, handoff) в проде.

### Состав репозитория

- `tasks/` — 24 задачи (TASK/CONTEXT/RUBRICS);
- `spine/`, `customization/` — спайн-пакеты и arch-кастомизация;
- `runners/` — весь код прогона, судейства и анализа (Python, stdlib);
- `results/` — `results.jsonl` (все 748 записей) и `summary.json`;
- `report/` — финальный docx + диаграммы PNG;
- `PREREGISTRATION.md`, `DEVIATIONS.md` — пререгистрация и D1–D14.

Источник истины и развитие бенчмарка — монорепозиторий Spine
(каталог `benchmarks/platformv-arch-bench/`).

---

## 🇬🇧 English

### What this is

A professional benchmark measuring whether a **specialized solution-architect
harness (Spine, banking edition)** and its working-artifact format
(architecture-spine, CONSTRAINTS.yaml, fitness gates) deliver a measurable
quality gain on architecture documents versus **general-purpose coding
harnesses** (Claude Code, Kimi Code, Theseus, OpenClaw, Qwen Code,
pi-coding-agent), on 24 architecture tasks built from the official
**Platform V (SberTech)** documentation.

**831 runs · 24 tasks · 19 conditions · 4 models · evidence-verified LLM judge · preregistered hypotheses**
>
> Status: 2026-09-13 snapshot — 831/973 cells completed and judged
> (claude/kimi arch arms and the 16-task GLM extension are partial, D17).

### Headline results

> **Condition glossary:** `spine-arch` — Spine harness + spine pack
> (architecture-spine + CONSTRAINTS.yaml, fitness gate), i.e. "harness and
> format"; `spine-min` — same Spine harness **without** the spine pack
> (isolates the harness contribution); `spine-arch-think` — `spine-arch`
> with model reasoning enabled (64K budget); `<harness>-plain` — a
> general-purpose coding harness in its **stock configuration**, no
> architect customization; `<harness>-arch` — same harness with architect
> customization; `raw-llm` — bare model (single API call).
>
- **Spine's role is model-dependent (H3 confirmed).** On GLM-5.3-Flash,
  Spine is **on par with the best general-purpose harnesses**: it led on
  the first 8 tasks (98.7/99.1), but after extending to 10 tasks the edge
  over Claude Code (+4.8 [−7.8; +19.0]) and Kimi Code (−1.3 [−10.8; +4.7])
  is no longer statistically significant; spine-min keeps a small
  significant lead (+8.9 [+1.7; +20.6], +2.9 [+0.3; +5.4]). On DeepSeek
  V4.1-Flash without reasoning it trails both (−8.0 [−15.2; −0.9] vs
  Claude Code, −11.5 [−18.1; −5.3] vs Kimi Code).
- **The spine format alone adds ~0** over the bare harness
  (−0.5 [−8.5; +7.5]); the working combination is "format + reasoning".
- **Reasoning is Spine's main amplifier on DeepSeek** (+10.9
  [+3.8; +18.2]); Spine with reasoning beats Theseus by +21.7
  [+10.7; +32.8].
- **Customization of coding harnesses for architects (H2) showed no
  effect** (Theseus −4.3, Claude Code +1.1, CI spans zero).
- Side findings: Theseus hard-fail rate 30–33% (turn-limit truncation);
  an arch-be × glm-5.3-flash agentic-mode incompatibility on some tasks.

### Methodology

973 cells (831 completed) = 24 tasks × 19 conditions × up to 2
repetitions × 4 models; kimi×glm ran via OpenRouter (same model, D17).
Each task ships `TASK.md` (role, D1–D10 deliverables), `CONTEXT.md`
(doc-labeled product facts, INF/NFR/SEC requirements) and `RUBRICS.md`
(weighted 0–4 criteria, hard-fail rules). Judge: deepseek-v4-pro over
anonymized answers, JSON verdicts, **evidence-quote verification**,
hard-fail caps the score at 39. Hypotheses preregistered
(`PREREGISTRATION.md` + task hashes); all deviations documented
(`DEVIATIONS.md`, D1–D14). Full pipeline in `runners/` (stdlib-only
Python); per-cell data in `results/`; final report with diagrams in
`report/`.

### Limitations

Single-family judge (deepseek-v4-pro) — declared, not measured
(judge pool cancelled for cost); dsf↔glm comparison on an 8-task
intersection; 2/750 cells dropped (chronic timeouts, D14). The benchmark
evaluates single-document architecture work, not the full gated,
traceable, handoff-driven architect workflow.

### Repository layout

`tasks/` — 24 benchmark tasks · `spine/`, `customization/` — spine packs
and arch customization · `runners/` — generation, judging, analysis,
report code · `results/` — `results.jsonl` + `summary.json` ·
`report/` — final docx + PNG diagrams · `PREREGISTRATION.md`,
`DEVIATIONS.md`.

Canonical source: the Spine monorepo (`benchmarks/platformv-arch-bench/`).

---

## License

MIT (benchmark code and materials). Platform V product names and
documentation excerpts belong to SberTech and are used for evaluation
purposes only.
