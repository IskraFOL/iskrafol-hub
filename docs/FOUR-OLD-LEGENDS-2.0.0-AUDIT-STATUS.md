# Four Old Legends 2.0.0 — текущий аудит и доказательства

**Контрольная дата:** 16 октября 2026 года  
**Назначение:** отделять прошедшие сборку/реальный Minecraft-тест системы от исходного кода и неподтверждённых предположений. Другие проекты IskraFOL поставлены на паузу — все ресурсы направлены на мод.

## Условные статусы

- **Verified runtime** — есть проверка внутри Minecraft/серверных тиков и артефакт.
- **Verified static/build** — исходники/расчёты/Java сборка проверены, но сценарий не полностью испытан в игре.
- **In test** — GitHub Actions запущен, итог неизвестен.
- **Open** — риск или сценарий не исследован/не подтверждён.

## Фактически проверенное

| Направление | Статус | Доказательства и ограничения |
|---|---|---|
| Java 25: сборка дорожных изменений | Verified static/build | [Build 38892503020](https://github.com/Anwite/four_old_legends/actions/runs/38892503020) — `clean build` и NBT проверки |
| Геометрия 6 соединений в 4 ориентациях | Verified static/build | [NBT regression](https://github.com/Anwite/four_old_legends/actions/runs/38892503020/artifacts/12042063769) — 24 из 24 соединений, ширина 6/6, без перекрытий |
| Road test: 7 сегментов, 6 стыков в Superflat | Verified runtime | [Minecraft #55](https://github.com/Anwite/four_old_legends/actions/runs/38892502973), [скриншоты](https://github.com/Anwite/four_old_legends/actions/runs/38892502973/artifacts/12042717059) — устранены разрывы на углах; это debug-стенд, не natural road |
| Автоматический посев, рост, пауза и возобновление при напряжении 15 | Verified runtime | [Minecraft auto growth #1, run 38895087557](https://github.com/Anwite/four_old_legends/actions/runs/38895087557), [JSON и снимки](https://github.com/Anwite/four_old_legends/actions/runs/38895087557/artifacts/12044345606) — 0→11→44→44→66 занятых клеток без принудительного строительства |
| Правые повороты в естественной генерации | Verified static/build | **Явно отключены**: `RoadBuilder.ALLOW_RIGHT_TURNS = false`. Обоснование в исходнике — фонарные столбы у перевёрнутого левого NBT стоят не по нужному борту; отдельного `road_corner_right.nbt` пока нет |
| Левые повороты | Verified runtime | [Minecraft natural bridge/left run 38897852049](https://github.com/Anwite/four_old_legends/actions/runs/38897852049): после пяти прямых и блокировки движения вперёд автоматически вставлен угол, конец направлен на север, `lastTurn=-1` |
| Мосты | Verified runtime | [Minecraft natural bridge/left run 38897852049](https://github.com/Anwite/four_old_legends/actions/runs/38897852049): без принудительного строительства пересечены 6 мокрых рядов, сеть выросла с 11 до 23 клеток, конец сдвинулся на 12 блоков |

### Текущий тест вариантов

[Draft PR №14](https://github.com/Anwite/four_old_legends/pull/14) — новый CI-сценарий [run 38897852049](https://github.com/Anwite/four_old_legends/actions/runs/38897852049), результат **подтверждён**: [артефакт 12044972237](https://github.com/Anwite/four_old_legends/actions/runs/38897852049/artifacts/12044972237), `bridge=true`, `five_straights=true`, `left_corner=true`, `verdict=PASS`. Исходное поколение строит дорогу только серверными тиками при `tension=15`: для моста тестовый мир содержит 6 водных рядов в пласте травы, а для поворота — 2 водных ряда после пяти прямых (прямая недоступна, мосту недостаточно воды). Тестовая ветка не содержит изменений алгоритма `RoadBuilder`/шаблонов; воспроизводимая карта создаётся только в одноразовом мире Actions.

## Открытые риски — дальнейший аудит

| Приоритет | Подсистема | Проверка |
|---|---|---|
| Высокий | Старые дорожные сохранения | `RoadEnd.pos` сменил семантику после аудита; `RoadEnd.fromKey` понимает старые форматы, **но конвертация старых координат не реализована**. Провести тест на копии старого сохранения перед 2.0.0; не менять пользовательские данные автоматически |
| Высокий | Незагруженные чанки / исчерпание концов сети | `RoadDirector.growFromRandomEnd` удаляет открытый конец, если `RoadBuilder.tryGrow` или вставка структуры не удались. Проверить, может ли временная недоступность чанка необратимо завершить сеть; отдельно исследовать перенос высот/биомы |
| Высокий | Предметы «пропажи» | `ItemHauntDirector` держит отложенные ItemStack в памяти и возвращает их при штатном shutdown/logout; аварийный crash между этими моментами остаётся риском утраты; проверить модель сохранения/возврата |
| Высокий | Откат временных сцен | `LegendaryMeetingEvent`, `LikChroniclerEvent`, `FeudDirector`: полное восстановление блоков и актёров при прерывании/смене измерения, отказе создания NPC, завершении сервера |
| Высокий | Сохранения и холодный перезапуск | ✅ Повторная загрузка одного CI-мира с новым форматом проверена в [38901063381](https://github.com/Anwite/four_old_legends/actions/runs/38901063381). Осталось: полная остановка JVM, исторические схемы/миры, повторный вход без дублирования событий |
| **Высокий** | Защита построек игрока от «Забвения» | Подтверждено чтением `OblivionDirector.java`: механика ограничена `ERASABLE`, но там есть обычные блоки, которые игрок тоже может поставить (`BOOKSHELF`, `MOSSY_STONE_BRICKS`, `SOUL_LANTERN` и др.). Проверяется тип блока и отсутствие block entity, **но не происхождение (создан Ликом или поставлен игроком)**. При `allowGriefing=true` игрок может потерять декорации. Нужна политика защиты/тест; при false мод уже не стирает |
| Средний | Правый дорожный угол | Решить отдельный NBT-ассет с корректным бортом фонарей, только после отдельного согласования визуала/включения. Геометрический debug-угол не заменяет natural runtime |
| Средний | Баланс легенд и контент 2.0.0 | Составить инвентаризацию существующих событий Herobrine, NULL, Entity 303, Лика, затем согласовать и реализовать новый контент для каждого. Не заменять контент только багфиксами |

## Правила изменений

Проверки проходят отдельными ветками, draft PR, удалённой сборкой и CI. Исходники Minecraft-мода и пользовательские миры не меняются без обоснованных проверок. До подтверждения данных и решения владельца **не выполнять merge** PR №9–№14. Целевая версия — **2.0.0**, без преждевременного изменения метаданных.

Подробные исторические результаты: [дорожная контрольная точка](OLD-LEGENDS-ROAD-CI-2026-10-16.md), [старый аудит от 14 октября](OLD-LEGENDS-AUDIT-2026-10-14.md), [план контента 2.0.0](FOUR-OLD-LEGENDS-2.0.0-PLAN.md).

## Контрольная точка: проверка фактического SavedData (16 октября 2026)

- Из `OldLegends.java`: дорожная сеть импортируется при `SERVER_STARTED` посредством `ArgSavedData.applyTo()` и копируется обратно при `SERVER_STOPPING` через `ArgSavedData.copyFrom()`; при длительной игре дополнительный снимок делается раз в 2400 серверных тиков. Сам `ArgSavedData.CODEC` содержит поля `roadEnds` и `roadOccupied`.
- `RoadNetwork` сохраняет занятые клетки как строки `x,z`, а открытые концы — как `x,y,z,direction,runLength,lastTurn`. `RoadEnd.fromKey()` принимает также исторический четырёхпольный формат. При этом **нет метки версии геометрии** и нет миграции исторических координат; корректное чтение строки ещё не означает безопасное продолжение старой дороги.
- Без изменения production-файлов создан [draft PR #16](https://github.com/Anwite/four_old_legends/pull/16) (`test/road-persistence-reload-2026-10-16`) от ветки проверенного естественного роста. CI создаёт исключительно одноразовый Superflat-мир, дожидается реального роста от напряжения 15, останавливает его на 14, читает статистику, выполняет `/save-all flush`, проверяет физический файл `arg_state` и снимает Pause Menu для надёжного последующего Save & Quit / повторной загрузки.
- Запущен [GitHub Actions 38899953547](https://github.com/Anwite/four_old_legends/actions/runs/38899953547). На момент записи **выполнялся**, результат записи/повторного входа не подтверждён.
- **Нельзя** переименовывать старые координаты наугад, очищать `roadEnds` или считать старые миры совместимыми по одному лишь успешному декодированию. После проверки нового формата понадобится отдельный синтетический fixture и тест на копии исторического мира.

## Контрольная точка — проба сохранения/повторной загрузки (16.10.2026)

- Исходники `ArgSavedData`, `ArgState`, `RoadNetwork`, `RoadEnd` и `OldLegends` сопоставлены: дорожные концы и занятые клетки сериализуются через `ArgSavedData`; копия делается раз в 2400 серверных тиков и при `SERVER_STOPPING`. При `SERVER_STARTED` данные импортируются обратно.
- Дорожный формат ключа `RoadEnd.fromKey` допускает 4 и 6 полей, но **версия схемы координат не хранится**. Текущие `RoadEnd.pos` — точки стыковки, старые значения могли иметь другую семантику. Распознавание старой строки **не означает геометрическую миграцию**. Нельзя заменять старые данные молча или сбрасывать сеть игрока.
- Создана ветка `test/road-savedata-reload-2026-10-16` на основе runtime теста естественного роста PR №12; открыт [draft PR №17](https://github.com/Anwite/four_old_legends/pull/17). Изменения — только CI workflow и скрипт сравнения; код Java и NBT мода не изменены.
- Начат [Minecraft RoadNetwork SavedData reload probe / run 38900190778](https://github.com/Anwite/four_old_legends/actions/runs/38900190778): в одноразовом мире при 15 естественно создаётся сеть, при 14 фиксируется `BEFORE_SAVE`, через реальное игровое Save and Quit выполняется `SERVER_STOPPING`/запись `.dat`, повторно открывается тот же мир и сравнивается `AFTER_RELOAD` по напряжению, занятым клеткам и ключам открытых концов. Это **проверка повторной загрузки в одном клиенте**, а не холодный JVM-рестарт.
- На момент записи GitHub выполнял тест; **сохранность RoadNetwork после повторной загрузки ещё не подтверждена**. При сбое GUI автоматизации скриншоты и отчёт позволяют отличить ошибку управления от дефекта SavedData. Следом требуется отдельный cold restart и совместимость старой геометрии.

## Контрольная точка: изоляция состояния между мирами (16.10.2026)

- Дополнительный статический аудит выявил реальный небезопасный путь: `OldLegends.java` в `SERVER_STARTED` раньше вызывал `director.resetForServer()` и `clearAllSanity()`, но **не очищал** долговременные поля `ArgState`, включая сеть дорог, напряжение, встречи и отметки смерти. Эти данные должны были перезаписаться только при успешном `ArgSavedData.applyTo()`. При ошибке чтения `SavedData` новый мир мог остаться с состоянием предыдущего мира, открытого в том же процессе Minecraft.
- Создана отдельная ветка `fix/world-state-isolation-2026-10-16` и [draft PR №18](https://github.com/Anwite/four_old_legends/pull/18). Добавлен `ArgState.resetForWorldLoad()`: сброс в оперативной памяти всех persisted-полей (без изменения файлов сохранения), вызов непосредственно перед `ArgSavedData.applyTo()`. Если чтение штатное, все значения текущего мира загружаются заново. Если ошибка, память не сохраняет данные прежнего мира.
- [Удалённая сборка Java 25 / run 38900409313](https://github.com/Anwite/four_old_legends/actions/runs/38900409313) завершилась **success**: `clean build`, NBT и дорожная регрессия пройдены. Игровая проверка чтения ошибочного SavedData остаётся открытой. Ни PR №16, ни PR №18 не сливались.
- Параллельно [проверка дискового сохранения/экрана выхода 38899953547](https://github.com/Anwite/four_old_legends/actions/runs/38899953547) в отдельной ветке PR №16 также была в процессе. Её цель — воспроизвести штатный Save & Quit и последующее открытие **этого же** тестового мира, а не предположить совместимость по декодеру.

### Проверка сохранений: результаты первого прогона и повторная попытка

- [GitHub Actions 38899953547](https://github.com/Anwite/four_old_legends/actions/runs/38899953547) завершился **failure** при проверке файла SavedData, но Minecraft реально создал 33 занятых дорожных клетки и 1 открытый конец при остановленном напряжении 14; система продолжила работу без игровой ошибки.
- Диагностический код ошибочно ограничил `find run/saves` глубиной 5, тогда как namespaced SavedData лежит глубже. Красный статус ещё **не является доказательством того, что данные мода не сохраняются**. [Артефакт 12046280062](https://github.com/Anwite/four_old_legends/actions/runs/38899953547/artifacts/12046280062) включает точный скриншот Pause Menu. `Save and Quit to Title` расположен в root-координатах примерно `(640,620)`.
- В коммите [`ce0cfb0`](https://github.com/Anwite/four_old_legends/commit/ce0cfb041d11ce987ae2153ecf8f7f227a85575d) ограничение глубины снято и добавлен **полный цикл штатного Save and Quit → выбор единственного существующего CI-мира → повторное открытие → `/legends road stats AFTER_RELOAD` → сравнение напряжения, занятых клеток, концов сети и их ключей**.
- Запущен [GitHub Actions 38901063381](https://github.com/Anwite/four_old_legends/actions/runs/38901063381), результат ещё не подтверждён. Ничего не менялось в собственно `RoadNetwork`, `SavedData` или мирном содержимом, только тестовый workflow PR №16.

### Повторная загрузка — прогон №1 и исправление клика

- [Run 38900190778](https://github.com/Anwite/four_old_legends/actions/runs/38900190778) завершился **failure из-за UI автоматизации**, не из-за формата сохранений: `BEFORE_SAVE tension=14 occupied=22 ends=1 ticks=2601 endKeys=29,-60,0,north,1,0`. Tesseract не распознал кнопку `Save and Quit to Title` на штатном экране паузы, после чего скрипт **остановился до клика**, не нарушая сохранение. [Артефакт 12045921094](https://github.com/Anwite/four_old_legends/actions/runs/38900190778/artifacts/12045921094) содержит реальный снимок кнопки.
- В скриншоте пункт `Save and Quit to Title` находится приблизительно в root `(640,485)`, с проверенным прямоугольником. В тестовой ветке PR #17 [коммит `ed59ade`](https://github.com/Anwite/four_old_legends/commit/ed59adeae4a224053cf3cf89cdfa2411ca41ffc5) заменил ненадёжное распознавание на клик по подтверждённой координате.
- [Повторный run 38901080354](https://github.com/Anwite/four_old_legends/actions/runs/38901080354) зарегистрирован, при фиксации выполнялся. Исход `Save and Quit`, результат `SavedData` и `AFTER_RELOAD` **ещё не подтверждены**. Никаких изменений алгоритмов дорог/сохранения; слияний нет.

## ✅ Повторная загрузка существующей дорожной сети: Minecraft Actions 38901063381

- [GitHub Actions 38901063381](https://github.com/Anwite/four_old_legends/actions/runs/38901063381) — **success** (commit `ce0cfb041d11ce987ae2153ecf8f7f227a85575d`, [артефакт 12046177283](https://github.com/Anwite/four_old_legends/actions/runs/38901063381/artifacts/12046177283)). Тест в [draft PR №16](https://github.com/Anwite/four_old_legends/pull/16), без изменения production Java/NBT.
- В настоящем Minecraft 26.3 на Superflat/Creative **естественно** создана дорожная сеть при напряжении 15, рост остановлен на 14, выполнено `/save-all flush`, затем графическая кнопка **Save and Quit to Title**, выбор уже существующего мира `FOL_CI_RoadProbe` в меню, повторный вход и `/legends road stats AFTER_RELOAD`.
- **Прямое сравнение статистики `road-reload-report.json`:** до штатного выхода и после повторного входа `tension=14`, `occupied=33`, `ends=1`, `endKeys=-5,-60,-8,west,2,0`. `failures=[]`, `verdict=PASS`. В скриншоте `36-after-reopen-attempt.png` подтвержден повторный вход в Superflat мир, а в журнале — `состояние ARG сохранено в мир` и `Stopping singleplayer server`.
- Фактический путь namespaced файла данных: `run/saves/FOL_CI_RoadProbe/dimensions/minecraft/overworld/data/oldlegends/arg_state.dat`. Прежнее ограничение `find -maxdepth 5` скрывало его. Исправленный тест обнаружил файл.
- **Что проверено:** сохранение и повторный запуск **встроенного сервера в том же JVM/клиенте**. Полный cold restart самого Minecraft-процесса, старые четырёхпольные `RoadEnd`, переход с версии старой геометрии и аварийное завершение остаются открытыми.
- Ветку `test/road-persistence-reload-2026-10-16` и PR №16 оставили draft. Не сливать до отдельного решения владельца.

### SavedData: диск подтверждён, повторный вход ещё проверяется

- [Run 38901080354](https://github.com/Anwite/four_old_legends/actions/runs/38901080354) завершился **failure из-за второго промаха GUI**, не из-за ошибки сериализации. [Артефакт 12047210062](https://github.com/Anwite/four_old_legends/actions/runs/38901080354/artifacts/12047210062) содержит `BEFORE_SAVE`, меню паузы и файл `world-savedata-files.txt`.
- Найден **реальный SavedData-файл** Minecraft 26.3: `run/saves/FOL_CI_RoadProbe/dimensions/minecraft/overworld/data/oldlegends/arg_state.dat` (204 байта). Прежний тест неверно искал `<world>/data/oldlegends_arg_state.dat`. Внутреннюю корректность данных и возможность повторного входа ещё предстоит проверить.
- Снимок паузы имеет разрешение **1280×1024**, тогда как auxiliary дисплей Xvfb был 1280×800. Предыдущая координата `(640,485)` попала **выше** `Save and Quit to Title`: игра оставалась на паузе, и `SERVER_STOPPING` в логах не подтверждён. По оригинальному снимку фактический центр кнопки `(640,620)`.
- [Коммиты `5a25c80`](https://github.com/Anwite/four_old_legends/commit/5a25c802b973f63d74f65fb11b4168ad4e683e1f) и [`825d719`](https://github.com/Anwite/four_old_legends/commit/825d719f744e62b7aacfd3890ec45fe53798cbfe) в draft PR #17 исправили путь файла и root-координату клика. Новый [run 38902349699](https://github.com/Anwite/four_old_legends/actions/runs/38902349699) зарегистрирован; успешный Save/Quit/reopen пока не подтверждён. Не заявлять о тестировании холодного перезапуска или миграции до фактических доказательств.

### SavedData: подтверждён штатный Save and Quit; тест перезахода #4

- [Run 38902349699](https://github.com/Anwite/four_old_legends/actions/runs/38902349699) записал в игровой журнал **одну штатную остановку** `[Old Legends ARG] состояние ARG сохранено в мир.`, а в мире `FOL_CI_RoadProbe` появился `dimensions/minecraft/overworld/data/oldlegends/arg_state.dat` размером **267 байт**. [Артефакт 12047291722](https://github.com/Anwite/four_old_legends/actions/runs/38902349699/artifacts/12047291722) содержит скриншот уже открытого **списка миров** с `FOL_CI_RoadProbe`, Creative, Cheats и версией 26.3. Это доказывает штатное сохранение и наличие тестового мира после возврата в меню.
- Результат #3 — **failure только потому, что OCR не распознал видимый текст имени мира**; скрипт безопасно остановился до открытия. Это не доказательство потери сохранений.
- В отдельной CI-ветке [коммит `4051d69`](https://github.com/Anwite/four_old_legends/commit/4051d69ff1260019a1c7097228db05444131f140) использует подтверждённые из изображения координаты единственной строки `(520,408)` и кнопки `Play Selected World` `(480,669)`; после загрузки проверяет `AFTER_RELOAD`. [Run 38903505468](https://github.com/Anwite/four_old_legends/actions/runs/38903505468) запущен; его исход **ещё не подтверждён**. Никаких реальных миров, исходников или NBT не изменено.

### ✅ RoadNetwork SavedData: проверен реальный повторный вход в мир (16.10.2026)

- [Minecraft RoadNetwork SavedData reload run **38903505468**](https://github.com/Anwite/four_old_legends/actions/runs/38903505468) завершился **success**, артефакт [12046544515](https://github.com/Anwite/four_old_legends/actions/runs/38903505468/artifacts/12046544515) содержит `road-savedata-reload-report.json`, `errors=[]`, `verdict=PASS`, логи и игровые снимки.
- Дорожная сеть создана автоматически при напряжении 15, потом рост остановлен при 14; до выхода `BEFORE_SAVE`: `tension=14`, `occupied=22`, `ends=1`, `endKeys=["-16,-60,15,west,1,0"]`, `ticks=2606`. Выполнен **настоящий** `Save and Quit to Title`, после выбора сохранённого мира `AFTER_RELOAD`: `tension=14`, `occupied=22`, `ends=1`, тот же самый `endKeys`, `ticks=751` (новая сессия интегрированного сервера).
- В журнале **две** загрузки `[Old Legends ARG] состояние ARG загружено из мира.` и **одно** штатное сохранение `[Old Legends ARG] состояние ARG сохранено в мир.`. На диске файл `run/saves/FOL_CI_RoadProbe/dimensions/minecraft/overworld/data/oldlegends/arg_state.dat` размером **208 байт**. На снимке `26-after-reopen.png` повторно открыт тот же мир с уже построенной дорогой.
- Принципиальное ограничение: это **Save/Quit/reopen в одном JVM-процессе Minecraft**, а не cold restart, crash recovery или миграция исторического `RoadEnd.pos`. Ничего не доказывает о совместимости старой версии геометрии. Никакие пользовательские сохранения и Java/NBT не менялись.
- [Draft PR №17](https://github.com/Anwite/four_old_legends/pull/17) содержит только скрипт и GitHub Actions тест. Следующий тест — cold restart с теми же файлами в одноразовом CI-мире; миграция старых данных по [защитному плану](ROAD-SAVEDATA-MIGRATION-SAFETY-2.0.0.md). Слияния нет.

## ✅ Контрольная точка: один и тот же мир пережил Save & Quit / Reload (16.10.2026)

- После исправления диагностического ограничения `find -maxdepth 5`, которое ложно скрывало файл, проверка [GitHub Actions run 38901063381](https://github.com/Anwite/four_old_legends/actions/runs/38901063381) успешно завершилась. [Артефакт 12046177283](https://github.com/Anwite/four_old_legends/actions/runs/38901063381/artifacts/12046177283), `road-reload-report.json` — `verdict=PASS`, `failures=[]`.
- Реальный путь SavedData в Minecraft 26.3: `run/saves/FOL_CI_RoadProbe/dimensions/minecraft/overworld/data/oldlegends/arg_state.dat` (новый, отдельно созданный в GitHub Runner мир).
- **До штатного выхода:** `BEFORE_SAVE tension=14 occupied=33 ends=1 endKeys=-5,-60,-8,west,2,0`, серверная стадия ~2978 тиков.
- Клиент выполнил **Save and Quit to Title**; лог подтверждает `состояние ARG сохранено в мир`, `Stopping server`, `Stopping singleplayer server as player logged out`, запись чанков.
- **После открытия того же мира**: `AFTER_RELOAD tension=14 occupied=33 ends=1 endKeys=-5,-60,-8,west,2,0`; новый серверный счётчик ~700 тиков. Журнал содержит **два** `состояние ARG загружено из мира` до/после перезапуска.
- **Вывод:** новая дорожная сеть, напряжение и координаты активного конца переживают холодную перезагрузку встроенного сервера. Это **не** доказывает совместимость старых координат до рефакторинга `RoadEnd.pos`; отдельный риск остаётся.
- При первичной неудачной диагностике [run 38899953547](https://github.com/Anwite/four_old_legends/actions/runs/38899953547) ошибка `FAIL_NO_SAVED_DATA` была вызвана недостаточной глубиной поиска, а не доказанной потерей данных.
