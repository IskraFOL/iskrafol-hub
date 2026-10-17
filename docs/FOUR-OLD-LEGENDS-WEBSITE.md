# Four-Old-Legends — макет и сайт

**Дата:** 17 октября 2026 года. **Статус:** Verification.

## Утверждённый источник

Исходная ссылка пользователя: https://www.figma.com/design/89PeZn8A9V2rRIg5LImCZV/Four-Old-Legends-%25E2%2580%2594-Homepage?node-id=0-1&p=f

Фрейм реализации: https://www.figma.com/design/89PeZn8A9V2rRIg5LImCZV/Four-Old-Legends-Homepage?node-id=1-2

File key: `89PeZn8A9V2rRIg5LImCZV`. Страница: `0:1`. Главный фрейм: `1:2`, 1438×1751. Эти ссылки сохраняются как источник дизайна, их не нужно восстанавливать по описанию.

## Реализация

Стек: React + TypeScript + Vite + SCSS Modules. Русский интерфейс. Исходные SVG экспортированы из существующих слоёв Figma; постоянные локальные файлы и шрифт Inter включены в сборку.

Сайт: https://four-old-legends.aptcrown8.chatgpt.site — приватный доступ владельцу. Sites подтвердил публикацию со статусом `succeeded`.

PR: https://github.com/Anwite/four-old-legends-website/pull/4 — черновик. Ветка: `feature/sites-figma-homepage`. База: `feature/website-homepage-design`. GitHub-коммит: `e5f7bd14cd76b5e98f524274a418c1237074b21c`.

Sites project ID: `appgprj_6ad3616536a88191b42a980d8db1c7eb`. Публикуемый коммит исходников Sites: `55ade149c796ad71f8770129cc8855636e77a25f`.

Существующие PR не сливались. Коммерческий шаблон остаётся самостоятельной веткой.

## Проверки и ближайший шаг

Подтверждены TypeScript и Vite production-сборка в управляемой Linux-среде, валидность семи SVG, их исходные размеры и отсутствие временных URL Figma в runtime-коде.

Браузерная проверка не выполнена: `control-browser` недоступен. Следующий шаг — визуальный просмотр и проверка кликов на компьютере и телефоне; после этого решение по PR. Полная проверка описана в `docs/VERIFICATION.md` репозитория сайта.
