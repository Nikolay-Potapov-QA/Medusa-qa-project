README-Medusa-aqa-framework
Medusa E-commerce AQA Framework
Фреймворк для тестирования backend/API-слоя e-commerce платформы Medusa: аутентификация, создание товаров, работа с корзиной. Архитектура заложена и под UI (Page Object), но сейчас в тестовом наборе задействован только API-слой — сборка фронтенда в локальном окружении была нестабильной, и вместо UI-тестов, которые падают не из-за багов продукта, а из-за окружения, в наборе остался только надёжный API-слой.
Что тестируется и зачем
Аутентификация: вход администратора через Admin API с валидными и невалидными данными.
Создание товара: создание товара с вариантами (например, размер) и ценой через Admin API, с автогенерацией уникального названия — чтобы тестовые прогоны не пересекались друг с другом.
Корзина: создание пустой корзины через Store API (publishable key, sales channel, region) — базовый шаг для дальнейших сценариев оформления заказа.
Тестовые данные создаются напрямую через API — это быстрее и стабильнее, чем через интерфейс, и не зависит от состояния фронтенда.
Стек и инструменты
Инструмент
Роль
Python 3.11
язык тестов
Pytest
тест-раннер
Playwright
API-клиент (через playwright.request); также содержит Page Object классы для UI — реализованы, но пока не подключены к активному прогону
Allure
отчётность, публикация на GitHub Pages
GitHub Actions
CI/CD: запуск по расписанию и при пуше
Telegram Bot API
уведомления о результатах прогона
Docker / PostgreSQL / Redis
окружение для локального запуска Medusa backend
Структура проекта
├── api/              # API-клиент для взаимодействия с Medusa API — активно используется
├── pages/            # Page Object для UI (LoginPage, ProductsPage, CreateProductPage, StorefrontPage) — заготовка, не подключена к тестам
├── tests/            # тест-кейсы (API)
├── docs/             # тест-план, тест-кейсы, примеры баг-репортов
├── config.py         # конфигурация через переменные окружения
├── conftest.py       # фикстуры: авторизация, генерация тестовых данных, screenshot-on-failure
└── .github/workflows/ # CI pipeline
​
Известное ограничение
UI-слой (Page Object для логина, каталога, создания товара, витрины) реализован в pages/, но не входит в активный прогон тестов: локальная сборка фронтенда была нестабильной. Вместо того чтобы держать в наборе флакующие UI-тесты, оставил только API-слой, который даёт стабильный и воспроизводимый результат. Расширение до UI — следующий логичный шаг при доступности стабильного фронтенд-окружения.
Как запустить
1. Клонировать репозиторий и установить зависимости
git clone <https://github.com/AQANikolay/Medusa-aqa-framework.git>
cd Medusa-aqa-framework
pip install -r requirements.txt
playwright install chromium
​
2. Поднять тестируемый backend (Medusa)
git clone <https://github.com/medusajs/medusa-starter-default> backend
cd backend
npm install

docker run -d -p 5432:5432 -e POSTGRES_PASSWORD=postgres postgres:15
docker run -d -p 6379:6379 redis:alpine

export DATABASE_URL=postgres://postgres:postgres@localhost:5432/medusa
export REDIS_URL=redis://localhost:6379

npx medusa db:migrate
npx medusa user -e admin@medusa-test.com -p supersecret
npx medusa develop
​
3. Настроить переменные окружения
Создать .env в корне проекта:
BASE_API_URL=http://localhost:9000
ADMIN_EMAIL=admin@medusa-test.com
ADMIN_PASSWORD=supersecret
API_TIMEOUT=10000
​
4. Запустить тесты
# Все тесты
pytest -v

# С отчётом Allure
pytest -v --alluredir=allure-results
allure serve allure-results
​
Что проверяют тесты
Успешная и неуспешная аутентификация администратора через API.
Создание товара с вариантами и ценой через Admin API, проверка, что товар появляется в списке.
Создание пустой корзины через Store API с корректными publishable key, sales channel и регионом.
Отчётность и диагностика
Отчёты Allure публикуются на GitHub Pages после каждого прогона в CI: https://AQANikolay.github.io/Medusa-aqa-framework/
Результаты дублируются в Telegram-чат — узнаёшь о падении тестов сразу, а не когда кто-то откроет CI.
В conftest.py реализован hook для автоприкрепления скриншота к Allure-отчёту при падении UI-теста — активируется, когда UI-слой будет подключён к прогону.
Настройка Telegram-уведомлений
Создать бота через @BotFather, получить токен.
Получить свой chat_id через @userinfobot.
Добавить в GitHub Secrets (Settings → Secrets and variables → Actions):
TELEGRAM_TOKEN
TELEGRAM_TO
QA-документация
В папке docs/ — тест-план и тест-кейсы в читаемом формате (с пометкой, что автоматизировано, а что требует ручной проверки), а также примеры оформления баг-репортов.