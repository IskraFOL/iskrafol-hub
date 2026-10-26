# КТ65 — FOL 1.3.0: исправление ложного успеха установки NBT Лика

**Дата:** 26.10.2026  
**Ветка мода:** `fix/lik-structure-placement-confirmation-kt65-2026-10-26`  
**Код:** [FOL Draft PR #105](https://github.com/Anwite/four_old_legends/pull/105), основан на [КТ64 / PR #104](https://github.com/Anwite/four_old_legends/pull/104), который продолжает КТ63/КТ62.

## Найденный дефект

`LikLegend.placeStructure` проверял наличие NBT, но **не считывал результат** `StructureTemplate.placeInWorld` и всегда возвращал `true`. Поэтому библиотека и склеп могли давать реплики и достижения даже при отказе игрового API.

## Реально исправлено

1. `placeStructure` теперь возвращает `boolean placed = template.placeInWorld(...)`; при `false` — запись в лог и отказ. Существующие NBT, повороты и координаты не трогались.
2. `likLibrary` и `likCrypt` передают в `LegendContext` `confirmEffect()` **только после успешного размещения** и до `Achievements.unlock`. Если NBT недоступна/не установлена, прежний запасной эффект остаётся, а фиктивное достижение за библиотеку/склеп не выдаётся.
3. `likCampfire` подтверждает либо удачную установку NBT, либо изменение главного блока запасного костра по `setBlockAndUpdate=true`.
4. Проверены реально существующие ресурсы `lik_library.nbt`, `crypt.nbt`, `campfire_scene.nbt`. Создан `scripts/check_lik_structure_confirmation_kt65.py`.

**Доказательства:** 11/11 статических инвариантов на актуальных Java-файлах в GitHub и 8/8 сценариев модели (наличие NBT × возвращаемый результат API × запасной блок). **НЕ выполнено:** Python в полном клоне, сборка JDK25/Gradle, настоящий Minecraft 26.3, безопасность/визуал размещения.

**Руины и финал:** настоящей NBT руин пока нет. Сюжетный флаг `lik_ruins_found` сохранён как известный технический долг, иначе нельзя получить все семь этапов `LikChronicler`. Отдельно исправлять вместе с реализацией руин. Остальные крупные ручные строения (`likTower/hall/vault/mansion`) ещё не получают доказанного API статуса.

**Риски:** метод установки больших NBT пока не делает исчерпывающего preflight мира и может менять соседние блоки. Не объявлять безопасным для личных миров до дальнейшей проверки, не делать merge. Напряжение/общий ambient-бюджет не менялись, новых легенд нет.

[Подробный технический документ FOL KT65](https://github.com/Anwite/four_old_legends/blob/fix/lik-structure-placement-confirmation-kt65-2026-10-26/docs/KT65-LIK-NBT-PLACEMENT-RESULT-CONFIRMATION-2026-10-26.md).

**Release First:** `main` не менялся, GitHub Actions не запускались, никто не трогал существующие миры или публикации.
