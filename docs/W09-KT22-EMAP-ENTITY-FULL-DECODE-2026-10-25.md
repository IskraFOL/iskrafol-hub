# W09-КТ22 — полностью прочитана запись сущности бинарной карты EMAP

**Дата:** 25.10.2026. **Проект:** Wolfenstein 2009. Только read-only анализ ранее предоставленного `farm.zip`, без модификаций игр и новых конвертеров.

## Основной результат

Исходник: `farm/farm.mpk` → ресурс `EMAP:maps/game/farm/farm` (Version 4). В блоке `entities` объект №12 (нулевая индексация; 13-й физически):

- Начало в блоке entities: **1872**, от начала распакованного EMAP: **126272**.
- Размер: **68 байт** = `uint32 field_count=4` + 4 дескриптора по 16 байт (`<IIII` little-endian).
- SHA-256 байтов записи: `e3da2de812c0c6293df49add2a9056390136d31748e7337e416a08090da933c1`.

| Поле | Сырые 4×u32 | Декодированное значение |
|---|---|---|
| `classname` | `(0, 0, 1, 76)` | `func_scriptrunner` |
| `name` | `(61, 0, 1, 59)` | `scriptrunner_elevator_controller` |
| `origin` | `(62, 1, 3, 33)` | `(344.0, -488.0, 208.0)` |
| `brain` | `(69, 0, 1, 77)` | `farm/elevator_controller.brain` |

Дескриптор `(key_index, type_code, arity, payload)`. Код `0`: payload — строковый индекс `values`; код `1`: payload — offset `floatValues`; код `3`: четвёртый u32 побитно представляют inline float32; код `4`: непосредственное значение uint32. `arity` на имеющихся оригиналах = 1, 3 или 9. Тип `3`, raw bits `1065353216` → float32 `1.0`.

## Сквозная доказанная связь EMAP → BRAIN

Имеются шесть сущностей `target_brainuse`, чьё поле `target` в точности равно `scriptrunner_elevator_controller`:

| № в EMAP | Имя | usekey |
|---:|---|---|
| 1 | `brainuse_elevator_goto_caverns` | `goToCaverns` |
| 10 | `brainuse_elevator_goto_barn` | `goToBarn` |
| 11 | `brainuse_elevator_goto_upperLabs` | `goToUpperLabs` |
| 13 | `brainuse_elevator_goto_lowerLabs` | `goToLowerLabs` |
| 15 | `brainuse_elevator_reinforcements` | `elevatorPuzzleComplete` |
| 17 | `brainuse_elevator_barn_entry` | `elevatorArriveInBarn` |

Контроллер `brain=farm/elevator_controller.brain` соотносится с реальным ресурсом `BRAIN:brains/farm/elevator_controller`, 14 643 байта, SHA256 `fa831d2549485585ae1322778a406f02c49a85bb7a46097a6e40bdcd0d2111a8`. Все шесть `usekey` действительно найдены в байтах этого BRAIN (у `elevatorPuzzleComplete` — 3 вхождения). Это подтверждает ссылочную связь, **не** доказывает реальный порядок запуска и игровую механику.

## Валидация 6 EMAP (6 310 сущностей) — PASS

Read-only декодером `W09_EMAP_Entity_Decode_KT22_readonly.py` реально проверены:

| EMAP ресурс | Число entities |
|---|---:|
| `maps/game/farm/farm` | 1912 |
| `maps/game/farm/caverns` | 4370 |
| `maps/game/farm/farm@tpd` | 1 |
| `maps/game/farm/farm@tpd_atmoslight` | 26 |
| `maps/game/farm/caverns@tpd` | 1 |
| `maps/game/farm/caverns@tpd_atmoslight` | 0 |
| **Всего** | **6310** |

Для каждой сущности прочтены `uint32 field_count` и все 16-байтовые дескрипторы; пройдены проверки смещений по таблицам `keys`, `values` и `floatValues`, типы 0/1/3/4, арифметическая точность общей длины секции, конечный LF, а также частоты использования ключей и строк относительно встроенных счётчиков. При декодировании не осталось непрочитанных байтов или неизвестных типов **в этих 6 EMAP**. Это не означает, что произвольный другой файл обязательно содержит только эти типы.

## Выданные артефакты

В текущем разговоре подготовлены:
- `W09_KT22_EMAP_Complete_Entity_Research_RU.md` — полный русскоязычный отчёт с побайтовыми u32 и hex каждой записи поля;
- `W09_KT22_EMAP_Entity_Full_Proof.json` — машинно-проверяемый отчёт с сырой 68-байтовой записью, SHA-256, 6 входящими связями и 6 проверенными EMAP;
- `W09_EMAP_Entity_Decode_KT22_readonly.py` — read-only форензический скрипт; **не конвертер или сериализатор на запись**.

Никакие оригинальные игровые MPK/SPK/BRAIN не публиковались в Hub, экспериментальных патчей нет. **Игровой тест отсутствует**, формат записи карты обратно в контейнер не реализован. Правило проекта: ввод пользователя в скриптах независим от регистра и обрамляющих пробелов; успешный завершающий текст не заменяет проверку факта публикации.

Продолжаем по существующим проверенным исходникам, не гоняем пользователя по кругу. В `main` не сливать без разрешения. Предыдущая КТ — [W09-21](https://github.com/IskraFOL/iskrafol-hub/pull/79).
