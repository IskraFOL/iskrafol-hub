# КТ63 — Four Old Legends 1.3.0: директор различает пропуск, попытку и ошибку события

**Дата:** 26.10.2026  
**Код:** [FOL Draft PR #103](https://github.com/Anwite/four_old_legends/pull/103) на базе [КТ62 / PR #102](https://github.com/Anwite/four_old_legends/pull/102). Main не менялся.

## Реализовано

Добавлен `LegendExecution` с тремя **честными** статусами: `SKIPPED_NO_ELIGIBLE` (ни одно событие не доступно), `ATTEMPTED_UNVERIFIED` (обработчик вызван, но исход неизвестен), `FAILED_EXCEPTION` (получено RuntimeException).

- `WeightedPool.execute(ctx)` проверяет непустой eligible-пул, вызывает событие и возвращает соответствующий статус; при исключении пишет `FOL_LEGEND_EVENT_FAILED` в журнал.
- `Legend.runAlgorithmWithOutcome(ctx)` добавлен как совместимый default-метод; все четыре легенды его переопределяют.
- `LegendDirector` перед `recordEncounter/sanity/achievement/cooldown` проверяет исход. `SKIPPED` не расходует встречу/рассудок/достижение; `FAILED` тоже, но получает 5-секундный защитный backoff.
- `ATTEMPTED_UNVERIFIED` **по-прежнему использует старую схему** подсчёта встречи/рассудка и полного cooldown. Это временно необходимо для сохранения существующего геймплея: у всех 116 обработчиков `void` и большинство не сообщает о реально состоявшемся эффекте.
- Обновлён `scripts/check_legend_diversity_kt62.py`, добавлен `scripts/check_legend_director_outcomes_kt63.py`.

## Проверено

- **PASS**: повторно считаны исходники и проверены **15/15 структурных инвариантов** интерфейса, четырёх легенд, `WeightedPool` и `LegendDirector`.
- **PASS (модель)**: 12 комбинаций статуса и исходного cooldown подтвердили, что пропуск и ошибка не считаются успешной встречей.
- **NOT RUN:** JDK25/Gradle/compileJava, исполнение Python из полного checkout, Minecraft 26.3, настоящая отработка 116 событий, работа нескольких игроков/серверов. Не называть это игровым PASS.

## Дальнейший предел исправления

Следующая независимая задача: перейти от `ATTEMPTED_UNVERIFIED` к доказанному `CONFIRMED_PLAYED / SKIPPED_UNSAFE / FAILED_PLACEMENT` для методов, где можно проверить реальное выполнение: построение NBT, `addFreshEntity`, `setBlock`, выдачу предметов, видимые эффекты. **До миграции** не отключать старый подсчёт для всех событий, иначе перестанут работать прогресс и достижения. Отдельно нужен общий бюджет ambient/легенд, поскольку параллельные звуки и сообщения создают ощущение повторов.

Принцип Release First: новых легенд и сюжетных сцен не добавлялось. Личные сохранения не трогали, не запускали Actions, слияния в main нет.

**Техническая КТ:** [FOL КТ63](https://github.com/Anwite/four_old_legends/blob/fix/legend-director-outcomes-kt63-2026-10-26/docs/KT63-LEGEND-DIRECTOR-EXECUTION-OUTCOMES-2026-10-26.md).
