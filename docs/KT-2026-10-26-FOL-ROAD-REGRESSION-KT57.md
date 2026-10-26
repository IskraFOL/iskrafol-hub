# КТ 57 — FOL 1.3.0: дорожные регрессионные тесты синхронизированы с новым кодом

Дата: 26.10.2026

**Состояние:** продолжена работа в [FOL Draft PR #98](https://github.com/Anwite/four_old_legends/pull/98); `main` без изменений, GitHub Actions не запускались.

## Две устранённые ошибки тестового контура

1. **Семисегментный стенд:** `scripts/check_road_bench_preflight.py` не содержал размер `road_corner_right.nbt` и после КТ55 при моделировании стенда возникал бы `KeyError`. Размер (13,8,17) внесён, список 7 сегментов и правильный правый угол теперь проверяются явно.
2. **Фонари:** `scripts/check_road_lantern_rotation.py` был рассчитан на старую, уже удалённую сигнатуру обхода прямоугольника. Обновлён под безопасный NBT-ограниченный обход `RoadPlacementSafety.templateLanternPositions`. Проверяются 5 вызовов, ориентация, тип фонаря, сохранение `HANGING` и отсутствие сканирования чужих фонарей.

## Проверено / не проверено

- **PASS:** повторно прочитаны исходники GitHub и сопоставлены 11 статических инвариантов нового Python/актуального Java.
- **PASS (независимая модель):** все 4 направления дорожного стенда сохраняют последовательность 7 сегментов, последний прямой имеет 693 клетки вне предыдущих габаритов. При блокировке этого участка модель останавливается до записи.
- **NOT RUN:** настоящий Python в полной копии репозитория; Minecraft 26.3; JDK25 Gradle; пользовательский мир, Save/Quit/reopen. Модель и чтение исходников не являются игровым PASS.

## Продолжение

Единый локальный тестовый набор: `check_road_palette_format.py`, `check_road_nbt_geometry.py`, `check_road_bench_seams.py`, `check_road_right_geometry.py`, `check_road_bench_preflight.py`, `check_road_lantern_rotation.py`, `mirror_road_corner.py --check`. После этого нужна настоящая компиляция и `/legends road test`.

- [FOL отчёт КТ57](https://github.com/Anwite/four_old_legends/blob/fix/road-26-3-palette-runtime-2026-10-26/docs/KT57-ROAD-2-0-REGRESSION-SUITE-2026-10-26.md)
- [FOL Draft PR #98](https://github.com/Anwite/four_old_legends/pull/98)
- [КТ56 в Hub](https://github.com/IskraFOL/iskrafol-hub/pull/83)

**Release First:** новые механики не добавлялись, только подготовка проверяемости релиза. Никаких merge.
