# КТ 43 — FOL 1.3.0: диктофон, события легенд и сохранения

**Дата:** 24.10.2026. **Статическая проверка** исходников `Anwite/four_old_legends` (ветка `fix/road-corner-nbt-palette-2026-10-22`, HEAD `918349d298064d7711dd38b333bac4d75c6ad28b`). **Никакой сборки, игрового теста или изменения исходников в этом этапе.**

## Подтверждено
- Диктофон: 5 кассет, `VoiceRecorderItem`, `VoiceRecorderScreen`, интерфейс по **R**, вставка/извлечение, OGG и звуковые события. Вставленная кассета сериализуется в `ItemStack`/`CUSTOM_DATA`.
- ItemHaunt: production **[PR #47](https://github.com/Anwite/four_old_legends/pull/47) MERGED**. Эффект только переставляет `ItemStack` в слотах PlayerData; старая внешняя очередь из Draft #40 не требуется.
- Сюжетные сцены: production **[PR #50](https://github.com/Anwite/four_old_legends/pull/50) MERGED**, код хранит ожидаемые временные состояния и не затирает чужие изменения при штатном откате.
- Рассудок по UUID игрока сбрасывается на новую сессию **намеренно**, не в `ArgSavedData`.

## Открытые риски
- **P0:** `LegendaryMeetingEvent` и `LikChroniclerEvent` запоминают возвращаемые блоки только в оперативной памяти. При аварийном выходе JVM откат не гарантирован. При штатной остановке `shutdownForServer` вызывает попытку отката, затем `resetForServer`, который очищает **неудавшиеся** записи — необходим сценарий сохранения/восстановления. Draft **[PR #52](https://github.com/Anwite/four_old_legends/pull/52)** содержит прототип журнала, но **не интегрирован с событиями**.
- **P1:** `OldLegends.java` защищает запись ARG флагом `loadedArgServer`, но `LegendDirector::onServerTick` и часть других обработчиков не охраняются единым условием после ошибочного чтения SavedData. Риск побочных эффектов при повреждённой загрузке; в игровом клиенте не проверялся.
- **P2:** в `VoiceRecorderScreen.findInsertedRecording` пустой диктофон в основной руке скрывает заряженный во второй.
- **P2:** статус `activeSound` в GUI не сбрасывается после естественного окончания воспроизведения; кнопка «Стоп» управляет лишь звуком экрана, не звуком серверного ПКМ.
- **P2/контент:** `en_us.json` содержит только названия записей в `subtitles.oldlegends.recording_0001..0005`, **не** переводы реплик русского голоса. Полноценные английские субтитры для сюжета ещё не подтверждены.
- **P2/наследие:** `NarrativeDirector` отслеживает выдачу книг в мире, новым игрокам после дат выдачи книги автоматически не передаются; для одиночной игры может не требовать исправления.

## НЕ считается проверкой
- Не проводились объединение Draft #72/#86, Java25/Gradle, Minecraft26.3, re-open и crash-test.
- Исторический CI PASS отдельного коммита не распространяется автоматически на общий билд 1.3.0.

**Следующая КТ 44:** финальная сверка веток и релизных блокеров, без расширения 1.3.0. **Release First**.

Полный тёмный PDF (3 страницы) и Markdown: ChatGPT Library → `/Four Old Legends/Аудит и документы/KT43-RECORDER-EVENTS-SAVES-2026-10-24.*`

Исходники: [VoiceRecorderItem](https://github.com/Anwite/four_old_legends/blob/fix/road-corner-nbt-palette-2026-10-22/src/main/java/com/oldlegends/item/VoiceRecorderItem.java), [VoiceRecorderScreen](https://github.com/Anwite/four_old_legends/blob/fix/road-corner-nbt-palette-2026-10-22/src/client/java/com/oldlegends/client/VoiceRecorderScreen.java), [OldLegends](https://github.com/Anwite/four_old_legends/blob/fix/road-corner-nbt-palette-2026-10-22/src/main/java/com/oldlegends/OldLegends.java).
