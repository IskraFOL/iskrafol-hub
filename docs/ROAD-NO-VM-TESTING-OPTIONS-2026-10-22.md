# Four Old Legends — проверка Minecraft/Fabric без виртуальной машины

Контрольная дата: **22.10.2026**. Исследование возможностей, НЕ отчёт о запущенных игровых тестах.
Связанная КТ: [Hub Draft PR #38](https://github.com/IskraFOL/iskrafol-hub/pull/38);
исходники: [FOL Draft PR #70](https://github.com/Anwite/four_old_legends/pull/70)
и [FOL Draft PR #72](https://github.com/Anwite/four_old_legends/pull/72).
Ветка #72 HEAD на момент анализа: `0548f0e03a5293d95d58e481d8fb5a8aedc3c990`.

## Что есть в текущем FOL

`gradle.properties`: Minecraft **26.3**, JDK **25**, Fabric Loader **0.19.5**,
Fabric Loom **1.17.18**, Fabric API **0.161.0+26.3**.
`build.gradle`: `loom { splitEnvironmentSourceSets() }`, `JavaCompile.options.release = 25`;
`fabricApi.configureTests` и `fabric-loader-junit` **пока отсутствуют**.
Поэтому нельзя обещать, что GameTest готов к немедленному запуску: сначала отдельный
тестовый коммит, синхронизация Gradle и совместимость API конкретной 26.3.

## Проверенные варианты

| Уровень | Способ | Нужна VM | Нужен рендер | Что доказывает |
| --- | --- | --- | --- | --- |
| 0 | Python + изолированная Java-модель (уже существует `scripts/check_road_p0_regression.py`) | Нет | Нет | Подмодель алгоритмов; НЕ реальный Minecraft |
| 1 | **Fabric Loader JUnit** | Нет | Нет | Код компонентов, валидация и сериализация там, где допустимы test-bootstrap/registries |
| 2 | **Fabric/Minecraft GameTest (сервер)** | Нет | Нет | Реальные ServerLevel, NBT/placeInWorld, BlockState, BlockEntity, отказ/rollback, 4 направления, Save/Load (где тест позволяет) |
| 3 | **Fabric Client GameTest** (обычная ОС + рабочий стол) | Нет | Да | Реальный клиент, отображение NBT/ItemDisplay, рекордер/UI, снимки |
| 4 | Клиентский тест на Linux с **Xvfb** и рабочим OpenGL-драйвером | Нет, Xvfb — виртуальный **экран**, не VM | Да, виртуальный дисплей + GL | Smoke test + клиентские скриншоты; 3D зависит от GL |
| 5 | Собственный **GitHub Actions self-hosted runner** на Windows/физическом Linux | Нет | По выбранным тестам | Регулярный запуск уровней 0–4 без GitHub-hosted VM/minutes |
| 6 | **HeadlessMC / MC-Runtime-Test** | Нет | Необязательно | Автозапуск клиента/заход в мир; сторонний экспериментальный инструмент, НЕ гарантия 26.3/Java25 |

### Официальные доказательства

- [Fabric Automated Testing](https://docs.fabricmc.net/develop/automatic-testing):
  Loader JUnit, Minecraft server game tests, Fabric client game tests,
  `fabricApi { configureTests { createSourceSet=true; ... } }`,
  `build` и `runClientGameTest`, возможность сохранять скриншоты.
- [Fabric Loom — Fabric API DSL / Tests](https://docs.fabricmc.net/develop/loom/fabric-api):
  `configureTests` создаёт конфигурации серверных и клиентских GameTest;
  тестовый source set и `eula` настраиваются явно.
- [Fabric GameTest API package](https://maven.fabricmc.net/docs/fabric-api-0.100.3%2B1.21/net/fabricmc/fabric/api/gametest/v1/package-summary.html):
  GameTest работает как специальный dedicated server, поддерживает результат тестов.
- [GitHub self-hosted runners](https://docs.github.com/en/actions/concepts/runners/self-hosted-runners):
  собственные ОС/железо, бесплатное использование runner со стороны GitHub
  (пользователь оплачивает своё железо), VM необязательна.
- [GitHub security](https://docs.github.com/en/actions/reference/security/secure-use):
  self-hosted не изолирован как одноразовая VM, НЕ запускать непроверенные PR
  и хранить токены/ключи с минимальными правами.
- [HeadlessMC](https://github.com/headlesshq/headlessmc),
  [MC-Runtime-Test](https://github.com/headlesshq/mc-runtime-test):
  внешние средства запуска; опубликованная матрица MC-Runtime-Test пока
  документирует Java только до 21, а наш проект требует 25.

**Версионное ограничение:** открытая на дату обзора страница официальной документации
помечена как **26.2**, а FOL таргетирует **26.3**. Реальные сигнатуры API,
Loom 1.17.18 и поведение 26.3 надо подтвердить компиляцией отдельного тестового
source set. Это план пилота, а НЕ утверждение о готовой интеграции.

## Рекомендованный пилот (не расширять релиз новыми функциями)

1. Существующий `scripts/check_road_p0_regression.py`: исполнить в подходящей
   JDK-среде, устранить найденные ошибки модели, опубликовать stdout и diff.
   Этот тест НЕ заменяет Java25/Fabric.
2. Отдельная ветка поверх Draft #72 только с тестовым source set
   (`fabricApi.configureTests` / `src/gametest`) и JUnit при необходимости.
   Не менять исходные NBT и геймплей ради прохождения тестов.
3. Серверные тесты 26.3: `road_segment`, `road_corner`,
   `bridge_segment` для всех 4 направлений, junction 1-блоковый gap,
   `structure_block` маркеры, AIR-кромка, чужие BlockEntity,
   `RoadTerrain` cut/finish, rollback после частичного false и сохранение
   RoadEnd при отказе. Все измерения по real-world `BlockPos`.
4. Отдельный стресс на `RoadNetwork`: импорт повреждённых/старых записей,
   попытки роста, reload и проверка `roadOccupied`; **шестипольные
   RoadEnd без версии геометрии не мигрировать автоматически**.
5. Клиентский `runClientGameTest` для скриншотов/визуальных проверок
   дороги, остановки и ItemDisplay в ИЗОЛИРОВАННОЙ тестовой директории.
   Нужен рабочий GPU/OpenGL (или подходящий Linux software renderer).
6. Если локальные тесты повторяемы, привязать **self-hosted runner** на
   личном Windows 11 (RTX 3060 Ti, JDK25) или выделенном физическом
   Linux-компьютере; запускать только по доверенному событию/manual dispatch,
   с жёсткими правами и отдельной директории. Runner должен быть в сети и
   включён, иначе никакого автономного исполнения не произойдёт.

## Чего нельзя обещать

- Серверный GameTest — **настоящая логика мира Minecraft**, но не проверка
  рендера, визуальных артефактов, звука, мыши или UI.
- Xvfb не является VM, однако ему всё ещё нужны хост-ОС и рабочий OpenGL.
- Self-hosted runner НЕ требует оплачиваемых GitHub-hosted minutes, но
  требует подготовленного, включённого компьютера/хоста. При полной
  блокировке Actions на аккаунте доступность задания надо подтвердить отдельно,
  не обходить ограничения вслепую.
- HeadlessMC с замещённым LWJGL не подтверждает корректность реального
  рендера. Для Java25/Minecraft26.3 совместимость не доказана.
- Оригинальные миры не использовать; GitHub Actions quota исчерпана и
  существующие задания **не запускались**.
- Ни один GameTest **ещё не написан/не запущен** в ходе этого исследования.
  Решение о новой тестовой ветке и запуске остаётся за владельцем по
  протоколу IskraFOL.


## Реализация после исследования — КТ №20

Первый этап предложенного плана реализован отдельной веткой
**[Four Old Legends Draft PR #73](https://github.com/Anwite/four_old_legends/pull/73)**,
`test/road-server-gametest-26-3-2026-10-22`, HEAD
`3ad625362c2c77cb88ebb3f7e62f510b4c20fb96`.
Ветка основана на Draft #72, не на `main`.

Созданы Gradle `configureTests`, отдельный модуль `src/gametest`,
площадка 48×16×48, девять серверных тестов дорожного кода и инструкция
[ROAD-SERVER-GAMETEST-PILOT-2026-10-22.md](https://github.com/Anwite/four_old_legends/blob/test/road-server-gametest-26-3-2026-10-22/docs/ROAD-SERVER-GAMETEST-PILOT-2026-10-22.md).

**Статические проверки 10/10 PASS; Java25/Gradle/GameTest пока NOT RUN.**
Предыдущая запись «ни один GameTest не написан» относится к стадии
исследования ДО PR #73, не является текущим состоянием.
EULA без согласия владельца не принимается; автоматический CI не запускали.
Результаты реального серверного теста появятся только после фактического
запуска на совместимой Java25/Fabric среде.
