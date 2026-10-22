# КТ 34 — NBT-структуры Four Old Legends, Minecraft Java 26.3

**Дата:** 22 октября 2026 года. Продолжение КТ 32 и КТ 33.

## Репозиторий и проверенный результат

**Мод:** `Anwite/four_old_legends`.

- [Draft PR #79](https://github.com/Anwite/four_old_legends/pull/79) — новые большие столы и структуры панелек Лика.
- [Draft PR #80](https://github.com/Anwite/four_old_legends/pull/80) — исходная пятиэтажка с 20 большими столами.
- **[Draft PR #81](https://github.com/Anwite/four_old_legends/pull/81)** — адресное исправление формата NBT, база PR #80, ветка `fix/nbt-palette-26-3-2026-10-22`.

### Причина

Найден и исправлен дефект генератора NBT: для палитры структур Java 26.3 требуются **`Name` / `Properties`**, а в прежнем генераторе ошибочно использовались `id` / `properties`. Предыдущий статический валидатор проверял тот же неверный формат.

### Изменения

1. Исправлены **все 9 затронутых NBT**: одна обычная пятиэтажка и восемь советских структур Лика (4 типа × 2 состояния). Изменены только имена тегов палитры, блочные состояния и позиции сохранены.
2. Исправлен генератор `scripts/generate_lik_soviet_nbt.js`; обновлены валидаторы `check_lik_soviet_nbt.py`, `check_soviet_panel_tables.py`, `check_interior_wave1.py`.
3. Добавлен `scripts/check_structure_palette_26_3.py`.
4. **9/9 NBT** повторно прочитаны из GitHub: NBT-структура, GZIP CRC32, `DataVersion=5023`, палитры и уникальность координат — PASS.
5. Обычная пятиэтажка — **16 813 блоков, 20 столов**, панельные корпуса Лика — **по 8 столов в заброшенной и сонной версиях**.

### Реальные ограничения

**NOT RUN:** Gradle compile JDK 25, настоящий Minecraft 26.3, коллизии, визуальный тест и Save & Quit. Нет изменений в worldgen. Ни один PR и ни один `main` не слит.

### Место хранения

Сохранено в библиотеке ChatGPT:
- `Four Old Legends/Аудит и документы/KT34-STRUCTURE-PALETTE-MC26-3-2026-10-22.md`
- `Four Old Legends/Сборки и исходники/check_structure_palette_26_3.py`

### Приёмка

После Fetch выбрать `fix/nbt-palette-26-3-2026-10-22` в IDEA и запустить привычный зелёный **Play**. В новом Creative-мире проверить размещение `/place template oldlegends:soviet_panel_5story ~ ~ ~`, 20 больших столов, двери, проходимость, сохранения и перезаход. **Release First**: сначала проверка и исправления, затем разрешённое слияние и релиз.
