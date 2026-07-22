Medusa QA Project

Тестовый проект для backend/API-слоя e-commerce платформы Medusa: аутентификация, создание товаров, работа с корзиной.

🏗️ Известное ограничение: архитектура заложена и под UI-тестирование (Page Object в pages/), но в активном прогоне сейчас только API-слой. Локальная сборка фронтенда была нестабильной — вместо UI-тестов, которые падают не из-за багов продукта, а из-за окружения, оставлен только надёжный API-слой.

✅ Что тестируется
🔐 Аутентификация — вход администратора через Admin API, валидные и невалидные данные
📦 Создание товара — вариант (например, размер) + цена через Admin API, с уникальным названием на каждый прогон
🛒 Корзина — создание пустой корзины через Store API (publishable key, sales channel, регион)

Тестовые данные создаются напрямую через API — быстрее и стабильнее, чем через интерфейс, и не зависит от состояния фронтенда.

⚙️ Стек
Инструмент	Роль
🐍 Python 3.11	язык тестов
✅ Pytest	тест-раннер
🎭 Playwright	API-клиент (playwright.request); Page Object классы для UI реализованы, но не подключены к прогону
📊 Allure	отчётность, публикация на GitHub Pages
🔄 GitHub Actions	CI/CD: по расписанию и при пуше
📩 Telegram Bot API	уведомления о результатах прогона
🐳 Docker / PostgreSQL / Redis	окружение для локального запуска backend
📁 Структура
├── api/              # API-клиент — активно используется
├── pages/            # Page Object для UI — заготовка, не подключена к тестам
├── tests/            # тест-кейсы (API)
├── docs/             # тест-план, тест-кейсы, баг-репорты
├── config.py         # конфигурация через переменные окружения
├── conftest.py       # фикстуры, генерация тестовых данных, screenshot-on-failure
└── .github/workflows/
🚀 Как запустить

1. Клонировать и установить зависимости

bash
git clone https://github.com/Nikolay-Potapov-QA/Medusa-qa-project.git
cd Medusa-qa-project
pip install -r requirements.txt
playwright install chromium

2. Поднять backend (Medusa)

bash
git clone https://github.com/medusajs/medusa-starter-default backend
cd backend
npm install

docker run -d -p 5432:5432 -e POSTGRES_PASSWORD=postgres postgres:15
docker run -d -p 6379:6379 redis:alpine

export DATABASE_URL=postgres://postgres:postgres@localhost:5432/medusa
export REDIS_URL=redis://localhost:6379

npx medusa db:migrate
npx medusa user -e admin@medusa-test.com -p supersecret
npx medusa develop

3. Настроить .env

BASE_API_URL=http://localhost:9000
ADMIN_EMAIL=admin@medusa-test.com
ADMIN_PASSWORD=supersecret
API_TIMEOUT=10000

4. Запустить тесты

bash
pytest -v                              # все тесты
pytest -v --alluredir=allure-results   # с отчётом Allure
allure serve allure-results
📊 Отчётность и диагностика
Отчёты Allure публикуются на GitHub Pages после каждого прогона в CI → https://nikolay-potapov-qa.github.io/Medusa-qa-project/
📩 Результаты дублируются в Telegram — падение тестов видно сразу, без захода в CI
📸 При падении UI-теста в отчёт автоматически прикладывается скриншот (сработает, когда UI-слой будет подключён к прогону)
🔧 Настройка Telegram-уведомлений
Создать бота через @BotFather, получить токен
Получить chat_id через @userinfobot
Добавить в GitHub Secrets (Settings → Secrets and variables → Actions):
TELEGRAM_TOKEN
TELEGRAM_TO
📚 QA-документация

В docs/ — тест-план и тест-кейсы (с пометкой, что автоматизировано, а что проверяется вручную) + примеры баг-репортов.
