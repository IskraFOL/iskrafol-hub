# IskraFOL — КТ: REC_0003 «Дом», интерьер Wave 1

**Дата:** 22.10.2026. **Статус:** первая реализация на отдельной ветке, **не** проверена запуском Minecraft.

## Согласованное направление

По прямому разрешению владельца реализуем модели интерьера из сценария
исследователя: старый стол, четыре стула, чашки и старая фотография.
В новых Java-файлах обязательно **русские комментарии**.
Рабочая **гипотеза (не канон)**: неназванный исследователь — игрок,
а узнавание лица на фото в REC_0003 подкрепляется его собственным скином.

Использовать физические блоки, совместимые с Blockbench JSON. Не повторять
проблему Z-fighting у остановки, не менять существующий её атлас.

## Текущая реализация

[Four Old Legends Draft PR #74](https://github.com/Anwite/four_old_legends/pull/74)
из ветки `feature/interior-models-player-photo-wave1-2026-10-22`,
**base: main** (чтобы не смешивать интерьер с Draft ROAD #70/#72
и автоматическими GameTest #73).

- `ModInteriorBlocks`: физические `table_block`, `chair_block`,
  `cup_block`, `photo_frame_block`; отдельная регистрация Block и BlockItem,
  предметы в Creative-вкладке Four Old Legends.
- `InteriorFurnitureBlock`: стол и чашка с собственной коллизией,
  стул со спинкой и поворотом коллизии во все четыре стороны.
- `PhotoFrameBlock`: настенная рамка с опорой и направлением.
- `PhotoFrameBlockEntity`: сохраняет только UUID установившего игрока,
  безопасное восстановление из `ValueInput`, синхронизация клиенту.
- `PhotoFrameRenderer`: актуальный Minecraft 26.3
  `createRenderState / extractRenderState / submit`,
  `PlayerSkinRenderCache` и отдельные отступы плоскостей от рамки.
  При наличии UUID — портрет установившего; без UUID — текущего зрителя
  на каждом клиенте; до загрузки скина нейтральная основа, без
  подмены случайным стандартным лицом.
- 4 пространственные Blockbench-совместимые JSON-модели, item models,
  item definitions, blockstates и loot tables — **20 новых ресурсных JSON**.
  Пока используется палитра ванильных текстур Minecraft (без PNG-атласа).
- RU/EN локализация; `/legends interior test` создаёт безопасную
  сцену из четырёх стульев, стола, чашки и фото на каменной стене
  **после проверки 7×9×4 пустого пространства и устойчивого грунта**.
- `scripts/check_interior_wave1.py` — подготовительный Python-аудит;
  `docs/REC-0003-INTERIOR-WAVE1-2026-10-22.md` — инструкция приёмки.

## Проверки на момент КТ

- 4 block models: 34 пространственных cuboid элемента;
  **4/4 проверки формы, текстурных переменных, отсутствия
  удалённого MC26.3 поля `shade` — PASS**.
- 4 item definitions + 4 blockstates (по 4 стороны) + 4 loot tables:
  **12/12 связных файлов PASS** (анализ JSON).
- **6/6 интеграционных проверок PASS**: RU/EN, регистрация блоков,
  регистрация рендера, команда тестового стенда,
  `PhotoFrameBlockEntity` UUID serializer.
- **Python скрипт NOT RUN, JDK25 Fabric build NOT RUN, actual Minecraft26.3
  render NOT RUN**. Никакой вывод о визуальной/игровой приёмке пока невозможен.
- Работа в **Draft PR**, без слияния в main, без изменений оригинальных
  пользовательских миров, без запусков Actions (квота ранее исчерпана).

## Следующее действие

Подтвердить `Gradle JVM = JDK25`, сделать Gradle Sync, запустить
`Fabric Tasks → runClient` зелёной кнопкой в IntelliJ IDEA в **новом
тестовом мире**. При ошибке сборки исправить конкретные API 26.3
только в PR #74. В Creative найти четыре предмета,
проверить коллизию, несколько направлений и
`/legends interior test`, фото и fallback, затем Save/Load.
**Только после успешной приёмки** интегрировать четыре объекта
в настоящую структуру дома REC_0003, без перезаписи дорожных NBT.

Постоянная связь с автотестами:
[ISKRAFOL-AUTOMATED-MINECRAFT-TESTING-PROTOCOL.md](https://github.com/IskraFOL/iskrafol-hub/blob/docs/kt-road-2-0-audit-close-1-3-0-2026-10-21/docs/ISKRAFOL-AUTOMATED-MINECRAFT-TESTING-PROTOCOL.md):
серверная проверка блоков/UUID в будущем и client GameTest для
визуальной фотографии.
