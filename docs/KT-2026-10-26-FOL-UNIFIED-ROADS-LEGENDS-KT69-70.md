# КТ69–70 — Four Old Legends 1.3.0: атмосферный директор и единая ветка для игрового теста

**Дата:** 26 октября 2026 г. **По запросу владельца:** завершить следующую часть аудита директора и объединить все связанные ветки в **один checkout**, чтобы можно было испытать Minecraft. **В основной main НЕ сливать до игрового теста.**

## КТ69 — шесть атмосферных систем на одном ограничителе

- [FOL Draft PR #109](https://github.com/Anwite/four_old_legends/pull/109), `fix/ambient-cue-extended-kt69-2026-10-26` от КТ68.
- К `AmbientCueBudget` (80 тиков / 4 сек на игрока) подключены дополнительно BoundaryDirector (ложные системные сообщения), ObsessionDirector (эпизоды под поведение игрока), OblivionDirector (эффект стирания, появление NPC, первая встреча).
- Профиль одержимости продолжает измеряться во время сюжетной сцены, но эпизод с шагами/подменой мира требует свободного слота. Забвение делает допуск до изменения блока и выдачи `oblivion_found`; при занятом бюджете эффект и награда не создаются. Уборка прежних временных NPC идёт независимо от бюджета.
- Ранее три системы КТ68 (AmbientDirector, PhantomSound, AmbientSound) тоже используют этот же экземпляр. Сердцебиение не ломается.
- **Фактические проверки:** 17/17 статических инвариантов исходников GitHub PASS; добавлен `scripts/check_extended_ambient_budget_kt69.py`, но Python из полного checkout **NOT RUN**.

## КТ70 — исполнено объединение на тестовый стенд

- **Единая игровая ветка:** `test/fol-1-3-0-roads-legends-unified-kt69-2026-10-26`.
- Создана от `experiment/road-flush-seed-2026-10-26` (Road 2.0 KT59–60, все предыдущие объединённые дорожные работы, интерьер/автобус).
- [FOL PR #110](https://github.com/Anwite/four_old_legends/pull/110) реально **MERGED INTO TEST BRANCH ONLY** из `fix/ambient-cue-extended-kt69-2026-10-26`; merge SHA `dd98dc6f09f2af157f5f107ab5695df43c15188f`. Это вложенно содержит код КТ62,63,64,65,66,67,68,69. Никакой merge в main не было.
- [FOL Draft PR #111](https://github.com/Anwite/four_old_legends/pull/111) — общий обзор **test → main**, намеренно только **Draft** до сдачи пользовательского теста.
- GitHub Compare непосредственно после объединения (до фиксации документа): `main...test`: **ahead=259, behind=0, 220 изменённых файлов**. Относительно КТ69 и дорожной КТ60 статус **ahead, behind=0**. Сверены в общей ветке `OldLegends` (6 общих атмосферных директоров), `LegendDirector` и WeightedPool, `RoadDirector` (flush seed Y-1), `RoadBuilder` (правый поворот включён), `LegendCommands` (`/legends road test`/`/legends road`; явный запрет `/legends build ruins`), `LikLegend` (руины не регистрируются). Отсутствие конфликтов на уровне GitHub **не гарантирует** успешную компиляцию Java.
- [FOL КТ70 — подробный чеклист теста 30–45 минут](https://github.com/Anwite/four_old_legends/blob/test/fol-1-3-0-roads-legends-unified-kt69-2026-10-26/docs/KT70-UNIFIED-ROADS-LEGENDS-GAMEPLAY-TEST-2026-10-26.md).

## Что пользователь проверит

IntelliJ IDEA `Git → Fetch`, выбрать только **test/fol-1-3-0-roads-legends-unified-kt69-2026-10-26**, JDK25 и `Fabric runClient`. Новый одноразовый Creative Superflat с командами, личные миры не открывать. Сначала старт/команды, затем `/legends road test` (NBT-стенд) и **отдельно** `/legends road` (новая высота -1), затем tension 15/30/50/75 (разнообразие легенд), атмосферный бюджет, `ambientEnabled`, запрет руин и Save/Quit/reopen. Сохранять первую ошибку `latest.log`/первый crash/`EMPTY_NBT_FOOTPRINT`, кадры и координаты; ранее изменённые настройки возвращать.

**Серьёзное ограничение:** КТ60 — эксперимент по высоте полотна только для нового посева, не все мосты и старые RoadEnd переведены на Y-1. Не путать успех `/legends road test` со способностью новой сети расти на рельефе. Мультиплеерное глобальное подавление ambient во время сцены ещё надо проверить.

**NOT RUN:** compileJava/Gradle на JDK25, реальный Minecraft 26.3, все репозиторные Python из клона, сохранение мира, multiplayer. Не объявлять игровой PASS. GitHub Actions НЕ запускались.

**Правило проекта:** Release First, нет новых легенд и неподтверждённых NBT; руины Лика **запрещены** согласно КТ66. `main` и пользовательские сохранения не тронуты; следующий merge возможен только после локального игрового теста владельца и согласования.