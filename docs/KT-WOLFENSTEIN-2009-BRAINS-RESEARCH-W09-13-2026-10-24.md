# Wolfenstein 2009 — W09-КТ13: исследованы все .brain из brains.rar

**Дата:** 24.10.2026  
**Статус:** статическая инвентаризация PASS; создание новых .brain с нуля / выполнение изменённых графов **не подтверждено**.  
**Основание:** пользовательский `brains.rar` (RAR5, ~406 КБ); игровые файлы в GitHub не загружались.

## Проведённые проверки

Распаковано **555 файлов** через libarchive, сверены размеры с каталогом RAR; 0 ошибок чтения. Из них **554 `.brain`** суммарно **3 300 798 байт** и **1 необычный `.html`** размером 5 052 байта с содержимым brain-like. Исходный RAR остался нетронутым.

Все 554 brain проанализированы побайтово в режиме read-only. Обнаружено:
- **22 папки/группы**, включая `farm` (**41 brain**), `common` (48), `mtw` (89), `dtn` (83);
- **161 различных имени поля** вида `mUseKey`, `mActor`, `mLine`, `mMover`, `mDuration`, ...;
- **28 945 вхождений** пар «имя поля → следующий NUL-терминированный токен» с зафиксированными смещениями;
- **553 файла** начинаются с `h1023`; **1** (`mtw_off/elevator.brain`) с `g1023`. Не утверждать, что это повреждение или что сигнатуры полностью разгаданы;
- `mtw/mt1_5200_ab.html` имеет такой же `h1023`-заголовок и не является обычной HTML-разметкой;
- **две пары побайтово одинаковых brain**: `castle/removeactive.brain` ≡ `farm/midwaybrain.brain`; `common/basic_template_partner_speak_anims.brain` ≡ `mte/gd3_1000_a.brain`.

## Примеры реально найденных полей

- `farm/start_conversation.brain`: `mLine=lipsync_farm_0501_wi1`, `mSpeaker=front_patrolman1`, `mDebugText` — английский отладочный текст. KT09/10 ранее подтверждали, что замена mDebugText при сохранении длины не ломает загрузку вступительной сцены, но русские субтитры берутся из отдельного `assets.pk4 / strings/russian.lang`.
- `farm/getawaytrain.brain`: `mMover=layer35_func_train_1`; параметры `mSpeed`, `mTime`, `mAccelTime`.
- `farm/elevator_controller.brain`: `mUseKey=goToBarn/goToUpperLabs/goToLowerLabs/goToCaverns`, `mMover=mvr_elevator`, `mDestination=tgt_elevator_...`, `mSpeed=60.00` и другие параметры. **Высокий риск сломать миссию; без игрового эксперимента не менять.**
- `digsite/alien_amulet.brain`: `mAnimName=pedestal_close_idle`, `mActor=digsite_pedestal`.
- `common/conversation_duo.brain`: параметризованный диалог `mLine=@cn_line1`, `mSpeaker=MyTarget 1/2`.
- `cannery/drive_to_firingrange.brain`: последовательность `mPassenger`, `mVehicle`, `mMover`, `mAnglularSpeed`; имя `mAnglularSpeed` **именно так написано в файлах**, не «исправлять» орфографию ключа.

## Переданные пользователю аналитические артефакты (в беседе, НЕ загружены в GitHub)

- `Wolfenstein_2009_BRAIN_Research_W09_13.xlsx` — **10 листов**: ОБЗОР, ГРУППЫ, КАТАЛОГ (554 файла), ПОЛЯ (161 ключ), ФЕРМА (41), ПРИМЕРЫ, ЭКСПЕРИМЕНТЫ, КАК ИЗМЕНЯТЬ, АНОМАЛИИ, ИСТОЧНИКИ. Все листы с табличной фильтрацией; ZIP/XLSX CRC PASS.
- `W09_Brains_Field_Occurrences.csv` — **28 945** строк с именем файла, ключом, значение, байтовым смещением; UTF-8 BOM, `;` separator, пригодно для поиска в Excel и обмена с DrZenk.
- `W09_Brains_Analysis_Data.json` — машинный индекс, по запросу; не обязателен для пользователя.

## Что можно пробовать менять, и что пока нельзя обещать

1. **Уже испытано в игре:** равнобайтовая замена `mDebugText` и пересборка MPK (начало сцены проходит без краша, но субтитры не меняются).
2. **Кандидаты на поведенческие тесты, НЕ проверены:** существующие значения `mSpeed`/`mTime` движущихся объектов, `mAnimName`, `mLocation`, `mUseSubtitle` с верными ссылками и отдельно изолированным изменением.
3. **Рискованные места:** `mUseKey`/`mUseTarget`/условия, лифты, цели, удаление NPC, переходы между картами, `mRootTask`/`mTask`, загрузка SPK; без карты ссылок не экспериментировать.
4. **Нельзя считать .brain полностью текстовым скриптом.** NUL-токены позволяют видеть много пар свойств, но заголовки, переходы/узлы, адресация и сериализация графа ещё не расшифрованы. Создание новых графов с нуля **не доказано**; обучаться на копиях действующих шаблонов.
5. 0x00 и ASCII '0' (0x30) различны; не удалять нулевые байты. Сохранение длины строки снижает риск смещений, но не гарантирует корректной семантики.

## Следующий шаг

Согласовать один **некритический поведенческий эксперимент**, проверить связь brain↔entity на карте и определить один изменяемый параметр с верификацией byte-diff/всех других ресурсов/запуском игры пользователем. Промежуточные правки в Test Copy и Draft PR; без слияния в `main`. **Four Old Legends остаётся на паузе.**

## Внешние материалы

- [ModDB: original scripts](https://www.moddb.com/games/wolfenstein/downloads/original-scripts)
- [ModDB: Pre-SDK Modding Wolfenstein 2009](https://www.moddb.com/tutorials/tut-pre-sdk-modding-of-wolfenstein-2009-single-player-v3)
- [GitHub: Wolfenstein SPK Tool, BRAI resources](https://github.com/dortkoldantaciz/Wolfenstein-SPK-Tool)
