# Platform V Architecture Benchmark: Spine vs универсальные кодовые харнессы

![Прогонов](https://img.shields.io/badge/прогонов-861-1f6feb)
![Задачи](https://img.shields.io/badge/задачи-24-8250df)
![Модели](https://img.shields.io/badge/модели-4-1a7f37)
![Судья](https://img.shields.io/badge/LLM--судья-верификация_цитат-bf8700)
![Гипотезы](https://img.shields.io/badge/гипотезы-пререгистрированы-cf222e)

> Профессиональный бенчмарх архитектурных задач по документации **Platform V
> (СберТех)**: измерение роли **харнесса и формата Spine** по сравнению с
> универсальными кодовыми харнессами (Claude Code, Kimi Code, Theseus,
> OpenClaw, Qwen Code, pi-coding-agent и др.).

---

> [!IMPORTANT]
> ## 🎯 Ключевые выводы для архитекторов
>
> 1. **Специализированный харнесс архитектора работает.** Spine со спайн-пакетом значимо обходит Theseus (**+18.3 балла [+7.5; +29.0]**) и идёт **в паритете с лучшими универсалами**: Claude Code (−0.5 [−5.9; +5.0]) и Kimi Code (−4.1 [−8.4; −0.1]).
> 2. **Но отрыва от универсалов нет.** Хороший универсальный харнесс + качественный `CONTEXT.md` закрывает те же задачи на 92–95 баллов без доменной специализации. Специализация поднимает Spine над самим собой, а не над рынком.
> 3. **Доменный формат (architecture-spine + CONSTRAINTS.yaml) — слабый плюс, а не серебряная пуля:** +3–4 балла поверх голого харнесса. Без ризонинга модели формат не конвертируется в качество.
> 4. **Кастомизация универсалов под архитекторов не окупается** (H2 ✗): arch-контексты у Theseus и Claude Code эффекта не дали, у Kimi Code — ухудшили результат (90.7 vs 94.7). Не тратьте время на «архитектурные» AGENTS.md для кодовых агентов.
> 5. **Агентный контур важнее выбора харнесса:** голая модель (raw-llm) — 77 баллов против 90+ у любого харнесса. Разница между харнессами вторична.
> 6. **Надёжность — главный риск агентных прогонов:** 30–33% ответов Theseus оборваны лимитом ходов; 18 «обрывов» Spine оказались дефектом извлечения, а не модели (D18); связка arch-be × glm-5.3-flash частично несовместима (D14). **Проверяйте пайплайн извлечения ответов прежде, чем судить модель.**
> 7. **Всё модель-зависимо (H3 ✓):** эффект любого харнесса надо мерить на вашей целевой модели — выводы с DeepSeek V4.1 Flash на GLM-5.3 переносятся лишь частично.

> [!TIP]
> **EN — Key takeaways for architects:** (1) the specialized architect harness (Spine) beats Theseus by +18.3 [+7.5; +29.0] and is at parity with the best general-purpose harnesses (Claude Code, Kimi Code); (2) but there is no breakaway — good universals with a solid `CONTEXT.md` reach 92–95 without domain specialization; (3) the domain format (architecture-spine) is a small +3–4 on top of the harness, not a silver bullet; (4) architect customization of coding harnesses does not pay off (H2 rejected); (5) the agentic loop itself matters more than harness choice (raw model 77 vs 90+); (6) reliability is the main risk — turn-limit truncations and extraction defects, check your answer-extraction pipeline before judging a model; (7) everything is model-dependent — measure on your target model.

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
| **spine-arch** (Spine + формат) | 89.6 ± 14.6 | 94.4 ± 17.7 | 82.2 ± 16.0 |
| **spine-min** (только харнесс) | 84.4 ± 15.5 | **98.5 ± 3.3** | — |
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
| spine-arch − claude-plain | **−0.5 [−5.9; +5.0]** | **паритет** с Claude Code (после фикса D18) |
| spine-arch − kimi-plain | **−4.1 [−8.4; −0.1]** | минимальное отставание от Kimi Code |
| spine-arch − theseus-plain | **+18.3 [+7.5; +29.0]** | **Spine значимо сильнее Theseus (H1 ✓)** |
| spine-arch − spine-min | +3.0 [−2.1; +8.0] | вклад формата — слабый плюс поверх харнесса |
| spine-arch-think − theseus-plain | **+21.7 [+11.0; +32.9]** | Spine + ризонинг значимо сильнее Theseus |
| spine-arch-think − claude-plain (фабричный) | +2.9 [−3.0; +8.7] | премия над фабричным Claude Code (паритет) |
| spine-arch-think − claude-arch (arch-контекст) | +1.5 [−4.3; +7.2] | премия над Claude Code с кастомизацией (паритет) |
| spine-arch-think − kimi-plain | −0.7 [−5.6; +3.5] | паритет с Kimi Code |
| spine-arch-think − spine-arch | +3.4 [−2.3; +8.8] | премия ризонинга после фикса обрывов |

![Средний балл по условиям](report/01_total_by_condition.png)
![Эффекты с доверительными интервалами](report/02_effects_forest.png)

### Честная интерпретация

0. **Важно: исправление D18 (обрывы).** Первая редакция результатов
   занижала Spine: 18 ячеек получили hard-fail из-за дефекта извлечения
   (arch-be писал полный документ в `work/answer.md`, а на stdout отдавал
   краткое подтверждение). После восстановления документов средний балл
   spine-arch на dsf вырос с 78.2 до 89.6, а выводы изменились:
1. **H1 подтверждена против Theseus и в паритете против лучших
   универсалов:** spine-arch − theseus-plain = **+18.3 [+7.5; +29.0]**;
   против Claude Code — **паритет** (−0.5 [−5.9; +5.0]); против Kimi Code —
   минимальное отставание (−4.1 [−8.4; −0.1]). На GLM-5.3-Flash Spine —
   на уровне лучших универсалов (spine-min 98.5).
2. **Формат спайна — слабый плюс поверх харнесса** (+3.0 [−2.1; +8.0];
   в парном по задачам анализе — около +4), а не ноль, как в первой
   редакции (там вклад формата тонул в обрывах).
3. **Специализация даёт прирост относительно самого Spine, но не отрыв
   от хороших универсалов** — Kimi/Claude Code закрывают те же задачи
   на 92–95 баллов без доменного формата.
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
> Status: 2026-09-13 19:43 snapshot — 861/973 cells completed and
> judged; spine truncation defect fixed (D18); claude/kimi arch arms and
> the 16-task GLM extension are partial (D17).

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
- **Important: D18 truncation fix.** The first data edition understated
  Spine: 18 cells hard-failed due to an extraction defect (arch-be wrote
  the full document to `work/answer.md` but printed only a short
  confirmation to stdout). After restoring the documents, spine-arch on
  dsf rose from 78.2 to 89.6 and the conclusions changed:
- **H1 confirmed vs Theseus, parity vs the best universal harnesses:**
  spine-arch − theseus-plain = **+18.3 [+7.5; +29.0]**; vs Claude Code —
  **parity** (−0.5 [−5.9; +5.0]); vs Kimi Code — a minimal gap
  (−4.1 [−8.4; −0.1]). On GLM-5.3-Flash Spine is on par with the best
  (spine-min 98.5).
- **The spine format is a small plus on top of the harness**
  (+3.0 [−2.1; +8.0], ~+4 in the task-paired view), not zero as in the
  first edition.
- **Specialization lifts Spine over itself but does not break away from
  good universal harnesses** — Kimi/Claude Code score 92–95 without the
  domain format.
- Spine with reasoning beats Theseus by +21.7 [+11.0; +32.9].
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
