# Four Old Legends — контрольная точка дорожного стенда в GitHub Actions (16.10.2026)

## Подтверждено

- Репозиторий: https://github.com/Anwite/four_old_legends
- Рабочая ветка проверки: `fix/road-rotation-anchor-main-2026-10-15`; PR #9 остаётся черновиком, не сливать без одобрения.
- Ранний клиентский smoke-test: https://github.com/Anwite/four_old_legends/actions/runs/38777696283 — успешная инициализация клиента в Xvfb и сохранение диагностического артефакта; тест не доказывает вход в мир или правильность дорожных стыков.
- Более поздняя контрольная точка: коммит `8105d006a17cebf96b7f94fd1741eea66c5c9485`, https://github.com/Anwite/four_old_legends/actions/runs/38860403212 — в артефакте визуально подтверждён вход в отдельный мир `FOL_CI_RoadProbe` с игровым HUD.
- На снимке настройки мира `Allow Commands` оставался OFF; запуск `/legends road test` и проверка геометрии не подтверждены.
- По прежнему статическому NBT-анализу входной рукав `road_corner.nbt` занимает X=0..5 (Z=1..10), тогда как комментарий `RoadGeom` указывает X=2..7; у `road_segment.nbt` полотно X=3..8. Несовпадение требует проверки фактических стыков, а не догадочного изменения NBT.

## Следующие действия

1. Проверить артефакт меню паузы и координаты Open to LAN либо надёжно включить команды при создании изолированного CI-мира.
2. Подтвердить включение команд по снимку или логам; отправить `/legends road test`.
3. Сохранить журналы, координаты, скриншоты нескольких ракурсов и видео; отдельно оценить прямые сегменты, повороты и зазоры.
4. До получения runtime-доказательств не менять дорожные NBT/формулы и не сливать PR #9.

**Статус:** проверка продолжается; клиент и вход в мир подтверждены, корректность дороги — нет.

## Новый изолированный runtime-прогон — 16.10.2026

- Создана ветка `test/road-runtime-ci-2026-10-16` от `fix/road-rotation-anchor-main-2026-10-15`.
- Коммит [`60b01cd`](https://github.com/Anwite/four_old_legends/commit/60b01cd615d0a46d88c83038981a6647df8dd18b) изменяет только `.github/workflows/minecraft-menu-probe.yml`: запускает workflow на push тестовой ветки и дополнительно сохраняет `road-chat-evidence.txt`.
- Открыт [draft PR №10](https://github.com/Anwite/four_old_legends/pull/10), база — ветка проверки дороги. Ничего не слито.
- Исходники Java и NBT не менялись; в workflow остаются запуск `/legends road test`, снимки мира и семь верхних ракурсов при наличии координат из игрового журнала.
- **Статус на момент записи:** коммит отправлен, но API доступного GitHub-коннектора ещё не вернул run ID или результат нового задания. Новый успешный runtime-прогон **не подтверждён**. Требуется изучить job logs и артефакты нового запуска, прежде чем утверждать устранение зазоров.

## Повторный CI-прогон с обязательными проверками — 16.10.2026

- В тестовой ветке `test/road-runtime-ci-2026-10-16` создан коммит [`65143e57`](https://github.com/Anwite/four_old_legends/commit/65143e57e1ef19bdce2cce73f7b15d9f586adb31).
- CI дополнен триггером pull_request в базовую ветку проверки и строгими критериями: без подтверждения `ROAD_TEST_COMPLETED_CONFIRMED_BY_MINECRAFT_CHAT_LOG=1` или без семи `AERIAL_CAPTURE` job завершается ошибкой. Даже при успешном результате геометрия требует просмотра кадров.
- Открыт [draft PR #10](https://github.com/Anwite/four_old_legends/pull/10), без слияния.
- GitHub зарегистрировал [Minecraft menu automation probe #35 — run 38876196804](https://github.com/Anwite/four_old_legends/actions/runs/38876196804). При проверке job `116891732692` находился в состоянии `in_progress`, шаг установки Java 25. Итог прохождения команды, скриншоты и результат дорожных стыков на момент записи ещё отсутствуют.
- Java и NBT-ассеты дорог не изменялись.

## Run #35 — результат и визуальная проверка артефактов

- [GitHub Actions run #35 / 38876196804](https://github.com/Anwite/four_old_legends/actions/runs/38876196804) завершился `success` (job `116891732692`). [Артефакт 12035971160](https://github.com/Anwite/four_old_legends/actions/runs/38876196804/artifacts/12035971160) содержит `result.txt`, `runClient.log` и семь `12-aerial-segment*.png`.
- `result.txt` подтвердил `BENCH_POINTS_FROM_LOG=7`, все семь `AERIAL_CAPTURE` и `ROAD_TEST_COMPLETED_CONFIRMED_BY_MINECRAFT_CHAT_LOG=1`; итог `ROAD_RUNTIME_VERDICT=COMMAND_AND_CAPTURES_CONFIRMED_GEOMETRY_NEEDS_VISUAL_REVIEW`.
- Координаты размещения стенда: `(-758,64,446)`, `(-768,64,457)`, `(-778,64,464)`, `(-794,64,467)`, `(-794,64,480)`, `(-793,64,491)`, `(-794,64,503)`. Камера во всех семи снимках на Y=106.
- По визуальному осмотру монтажного листа скриншотов: в кадрах 1–5 частично заметна дорога и/или участки рядом; сегменты 6–7 заслонены заснеженным горным рельефом и деревьями. Нельзя подтвердить качество всех стыков даже при зелёном CI. Отдельный файл `road-chat-evidence.txt` пуст (фильтр не нашёл строки), хотя `runClient.log`/`result.txt` подтверждают команду.
- **Вывод:** реальное выполнение команды и появление семи точек в мире подтверждены; исправление дорожных стыков НЕ подтверждено. Требуется стабильная плоская площадка, лучше superflat для CI, кадры каждого соединения без заслонения и проверка фактических краёв NBT. Никаких изменений геометрии/слияний на этом этапе.

## Следующий этап: поиск настройки плоского мира (16.10.2026)

- Коммит [`eb89c22`](https://github.com/Anwite/four_old_legends/commit/eb89c22ca356aa214ed0abdc97e26490b38a9efc) в тестовой ветке PR #10 дополняет CI-сценарий: перед созданием мира открывает вкладку **World**, снимает `04-world-tab-terrain-options.png`, возвращается к вкладке **Game** и затем продолжает дорожный пробник.
- Цель — подтвердить расположение настройки типа мира и в следующем проходе выбрать Superflat. Это диагностика UI, а не заявление об уже включённом Superflat.
- GitHub зарегистрировал [run #37 / 38879325477](https://github.com/Anwite/four_old_legends/actions/runs/38879325477) со статусом `queued` на момент записи. Проверка дорожных зазоров всё ещё открыта.

## Результат #37 и новая попытка выбора Superflat — 16.10.2026

- [Run #37 / 38879325477](https://github.com/Anwite/four_old_legends/actions/runs/38879325477) завершился успешно: `BENCH_POINTS_FROM_LOG=7`, получены 7 снимков и подтверждение дорожного стенда в логе. Артефакт [12036354614](https://github.com/Anwite/four_old_legends/actions/runs/38879325477/artifacts/12036354614).
- Изображение `04-world-tab-terrain-options.png` подтверждает экран настройки World в Minecraft 26.3, переключатель `World Type: Default` с центром около root (480,362), окно клиента начинается root (212,252).
- Коммит [`41934b5`](https://github.com/Anwite/four_old_legends/commit/41934b51990365f200c9acbc394a0a4b79e89bd9) обновил тестовую ветку PR #10: выполняется один клик по World Type и сохраняются новый скриншот и OCR-отчёт о типе мира. Пока Superflat не подтверждён.
- [Run #39 / 38880177174](https://github.com/Anwite/four_old_legends/actions/runs/38880177174) зарегистрирован, при проверке выполнялся. Следующий шаг — прочесть артефакт, подтвердить Superflat, затем оценивать стыки дороги. NBT и Java не менялись; PR не слиты.

## Прогон #39 и исправление переключателя World Type (16.10.2026)

- [Run #39 / 38880177174](https://github.com/Anwite/four_old_legends/actions/runs/38880177174) завершился успешно. Артефакт [12038035339](https://github.com/Anwite/four_old_legends/actions/runs/38880177174/artifacts/12038035339) подтверждает 7 координат стенда, 7 игровых снимков и сообщение `ROAD_TEST_COMPLETED_CONFIRMED_BY_MINECRAFT_CHAT_LOG=1`.
- Ключевое ограничение: `04-world-type-after-click.png` по-прежнему показывает `World Type: Default`. Предыдущий синтетический клик не переключил генерацию; `WORLD_TYPE_OCR=NOT_CONFIRMED_REVIEW_SCREENSHOT`. Дорога частично видна, но рельеф мешает достоверно проверить стыки.
- В коммите [`8664bdc`](https://github.com/Anwite/four_old_legends/commit/8664bdcc9cdcfe20f522f77a117102749ae1ad5f) клик заменён явными `mousedown`/`mouseup`, добавлена увеличенная область кнопки для OCR.
- Зарегистрирован [run #41 / 38881187953](https://github.com/Anwite/four_old_legends/actions/runs/38881187953); на момент записи он выполнялся. Superflat и целостность стыков ещё не подтверждены. PR #10 остаётся draft, Java/NBT не менялись, слияний нет.

## Проверка #41 и следующий тест #43 — 16 октября 2026

- [GitHub Actions #41, run 38881187953](https://github.com/Anwite/four_old_legends/actions/runs/38881187953) — `success`; артефакт [12037817416](https://github.com/Anwite/four_old_legends/actions/runs/38881187953/artifacts/12037817416). В `result.txt`: `BENCH_POINTS_FROM_LOG=7`, семь `AERIAL_CAPTURE`, `ROAD_TEST_COMPLETED_CONFIRMED_BY_MINECRAFT_CHAT_LOG=1`.
- Однако скриншот `04-world-type-after-click.png` по-прежнему показывает **World Type: Default**. Таким образом, клики относительно окна не переключили тип мира; качества всех стыков нельзя подтвердить.
- На изображении #41 кнопка World Type расположена по координатам root X=330..628 и Y=343..381, центр (480,362). Коммит [`f2393b6`](https://github.com/Anwite/four_old_legends/commit/f2393b6fdb93a4d6cacc6d8d48f64fc7f0aae068) переводит нажатие на эти абсолютные координаты, сохраняя снимок и OCR.
- Зарегистрирован [GitHub Actions #43, run 38884410429](https://github.com/Anwite/four_old_legends/actions/runs/38884410429), при проверке `queued`. Superflat пока не подтверждён. Исходники Java/NBT не менялись, PR #10 остаётся draft и не слит.

## Контрольная точка: Superflat и полный дорожный стенд — 16.10.2026

- [GitHub Actions #43, run 38884410429](https://github.com/Anwite/four_old_legends/actions/runs/38884410429) завершился **success**, артефакт [12039292051](https://github.com/Anwite/four_old_legends/actions/runs/38884410429/artifacts/12039292051).
- В `04-world-type-after-click.png` реально выбран **World Type: Superflat**; OCR распознал `World Type: Supertlat` (опечатка автоматического распознавания). В мире плоская трава, сегменты имеют координату `Y=-60`, семь снимков сделаны камерой с `Y=-18`.
- Игровой журнал `runClient.log` показывает семь размещений и сообщение `── стенд готов. Смотри стыки 1-2, 2-3, 3-4, 4-5, 5-6, 6-7.`; `result.txt` подтверждает `BENCH_POINTS_FROM_LOG=7`, семь `AERIAL_CAPTURE` и `ROAD_TEST_COMPLETED_CONFIRMED_BY_MINECRAFT_CHAT_LOG=1`.
- Впервые получены **чистые игровые кадры всех семи сегментов на ровной площадке**. Рельеф больше не мешает. Но обзор сегментов не покрывает каждый стык достаточно близко: зелёный CI подтверждает размещение, **но ещё не идеальные соединения полотна**.
- В тестовой ветке [PR #10](https://github.com/Anwite/four_old_legends/pull/10) создан коммит [`619b640`](https://github.com/Anwite/four_old_legends/commit/619b640aa58a33d29898570043d40972bc8d724f): workflow дополнен шестью прицельными снимками `13-joint1-2.png` … `13-joint6-7.png`, общим обзором `14-road-bench-overview.png` и проверками их наличия. Координаты для кадров вычисляются из фактического runtime-лога, а не из прежнего мира.
- Следующая задача — выполнить новый CI и изучить 6 стыков по отдельности. До доказанного совпадения полотна не сливать PR #9/#10, не менять NBT и Java наугад.

## Контрольная точка #45: шесть камер стыков, обнаружена ошибка прицеливания

- [GitHub Actions #45 / 38885877626](https://github.com/Anwite/four_old_legends/actions/runs/38885877626) — **success**, [артефакт 12040070626](https://github.com/Anwite/four_old_legends/actions/runs/38885877626/artifacts/12040070626).
- Runtime-факт: `/legends road test` построил семь частей в **Superflat**, `BENCH_POINTS_FROM_LOG=7`, сняты 7 сегментов, 6 кадров `13-joint*.png` и `14-road-bench-overview.png`; CI вынес `ROAD_RUNTIME_VERDICT=COMMAND_AND_7_SEGMENTS_6_JOINTS_OVERVIEW_CAPTURED_GEOMETRY_NEEDS_VISUAL_REVIEW`.
- Ручной просмотр снимков показал: **стыки 1–2 и 2–3 попали на край кадра**, поскольку координаты камер были взяты по серединам якорей соседних NBT-структур, которые не совпадают с настоящими точками соединения. **Общий обзор с `Y=30` (прибавка 90 к поверхности −60)** показал преимущественно небо. Это ошибка диагностического сценария, а не доказанный дефект игровых дорог.
- **Новый план:** брать 6 координат прямо из строк runtime-журнала `выход=[x,y,z]`, где геометрия мода явно задаёт центр выходного соединителя; снижать камеру общего вида до `Y + 42`. Не менять Java/NBT до наглядного сравнения шести соединений.
- В тестовой ветке [PR #10](https://github.com/Anwite/four_old_legends/pull/10) отправлен коммит [`31673743`](https://github.com/Anwite/four_old_legends/commit/31673743aaac0ab4f193ca81c668445f49c9ba3e), запущен [GitHub Actions #47 / 38886906000](https://github.com/Anwite/four_old_legends/actions/runs/38886906000). На момент записи он выполняется, результат прицельных снимков **ещё не подтверждён**. Слияний нет.

## Контрольная точка #47: причина красного CI найдена; тест #49

- [GitHub Actions #47 / 38886906000](https://github.com/Anwite/four_old_legends/actions/runs/38886906000) завершился **failure**; диагностический [артефакт 12039902392](https://github.com/Anwite/four_old_legends/actions/runs/38886906000/artifacts/12039902392) сохранён.
- В `runClient.log` зафиксировано: в `10:26:35` игрок `Player386` вошёл в Superflat-мир, а в `10:27:40` — **Player386 was slain by Slime**. На `09-road-test-after-enter.png` виден экран смерти. В `result.txt`: `BENCH_POINTS_FROM_LOG=0`, `ROAD_TEST_COMPLETED_CONFIRMED_BY_MINECRAFT_CHAT_LOG=0`. Причина: создание тестового мира в **Survival / Difficulty: Hard**, а не ошибка геометрии дорог.
- Ранее Superflat был подтверждён, но Creative до появления игрока не выбирался. Синтетическая команда `/gamemode creative` в старом сценарии выполнялась уже после построения стенда, слишком поздно для предотвращения смерти.
- В тестовой ветке PR [№10](https://github.com/Anwite/four_old_legends/pull/10) создан коммит [`895b182`](https://github.com/Anwite/four_old_legends/commit/895b182cca9b011d0a2c2f43d6b7fb54d39b0afd): ещё на экране создания мира автоматически переключать `Game Mode` в **Creative** (не более трёх попыток), сохранять скриншоты и увеличенные области + OCR, отменять тест при неподтверждённом режиме. Дополнительно исправлено экранирование регулярного выражения сбора чата, вызывавшее `grep: Unmatched [`.
- Запущен [GitHub Actions #49 / 38888143945](https://github.com/Anwite/four_old_legends/actions/runs/38888143945), при первой проверке был `queued`. Его исход **не подтверждён**. Следующее действие — проверить скриншоты выбора Creative и игровые кадры всех шести стыков.
- Ни Java, ни NBT-ассеты, ни версия игры не менялись; PR #9 и PR #10 не слиты.

## Контрольная точка #49: Creative найден, OCR ошибался; тест #51

- [GitHub Actions #49 / 38888143945](https://github.com/Anwite/four_old_legends/actions/runs/38888143945) завершился **failure**, артефакт [12040198086](https://github.com/Anwite/four_old_legends/actions/runs/38888143945/artifacts/12040198086). CI остановился **до создания мира**: `GAME_MODE_NOT_CONFIRMED=CREATIVE`, поскольку OCR маленькой пиксельной кнопки ошибочно не распознал режим.
- Вручную рассмотрены все три кадра `04-game-mode-attempt-*.png`: **1-й клик — Hardcore, 2-й — Creative, 3-й — Survival**. На втором кадре в режиме Creative настройка **Allow Commands: ON** уже включена автоматически; при переключении обратно на Survival — OFF. Это фактическое поведение интерфейса Minecraft 26.3, а не догадка.
- В тестовую ветку PR #10 отправлен [коммит `40c542d`](https://github.com/Anwite/four_old_legends/commit/40c542d3ff0e7c8a286ae950f1e78fa2980e16ef): ровно два нажатия Game Mode из Survival, скриншоты до создания мира, без ненадёжной OCR-блокировки; вторично **не трогать** кнопку Allow Commands, которая в Creative уже ON.
- Запущен [GitHub Actions #51 / 38888857579](https://github.com/Anwite/four_old_legends/actions/runs/38888857579), при проверке — `in_progress`. Итог создания мира, команды, шести стыков — **пока не подтверждён**. Геометрия Java и NBT не менялась, PR #9 и #10 не слиты.

## Контрольная точка #51 — Creative, Superflat и доказательства дефекта геометрии (16.10.2026)

- [GitHub Actions #51 / 38888857579](https://github.com/Anwite/four_old_legends/actions/runs/38888857579) завершился **success**, [артефакт 12041143332](https://github.com/Anwite/four_old_legends/actions/runs/38888857579/artifacts/12041143332), коммит CI [`40c542d`](https://github.com/Anwite/four_old_legends/commit/40c542d3ff0e7c8a286ae950f1e78fa2980e16ef).
- На снимке `05-world-setup-commands.png` подтверждено **Game Mode: Creative**, **Allow Commands: ON**. Тестовый мир Superflat. Команда `/legends road test` успешно выполнена.
- `result.txt`: `BENCH_POINTS_FROM_LOG=7`, семь `AERIAL_CAPTURE`, шесть `JUNCTION_CAPTURE` по точным `выход=[x,y,z]`, `OVERVIEW_CAPTURE=1`; завершение `ROAD_RUNTIME_VERDICT=COMMAND_AND_7_SEGMENTS_6_JOINTS_OVERVIEW_CAPTURED_GEOMETRY_NEEDS_VISUAL_REVIEW`.
- Координаты семи сегментов: `(-8,-59,0)`, `(-18,-59,11)`, `(-28,-59,18)`, `(-44,-59,21)`, `(-44,-59,34)`, `(-43,-59,45)`, `(-44,-59,57)`; камеры шести стыков: `(-5,-31,11)`, `(-18,-31,21)`, `(-29,-31,21)`, `(-41,-31,34)`, `(-41,-31,45)`, `(-41,-31,57)`.
- **Ручной осмотр игровых снимков:** участки 5–6 (прямая → мост) и 6–7 (мост → прямая) выглядят **сдвинутыми вбок**, с видимой полосой травы между полотнами. Это уже предметное наблюдение в игровом мире, а не ошибка Xvfb и не сбой запуска. На других соединениях возможны ступени/неполное перекрытие, требующие точного анализа блоков.
- **Важно:** CI success означает **команда построила стенд и сохранила кадры**, а не то, что геометрия дорог исправлена. Слияния PR #9/#10 преждевременны.
- Следующий этап: сопоставить настоящий footprint `road_segment.nbt`, `road_corner.nbt`, `bridge_segment.nbt` с координатами `RoadGeom` и поворотами `Structures.place`; отдельно проверить мостовой настил на Y=1. Исправлять только доказанное конкретное несоответствие, тестировать повторно, не сливать без согласования.
