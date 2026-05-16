# ReportGenerator

Кроссплатформенный CLI-инструмент для автоматической генерации отчётов из PostgreSQL в Excel и PDF с графиками.

**Архив `ReportGenerator-source.tar.gz` в корне репозитория содержит полные исходники проекта — скачайте и распакуйте.**

## Возможности

- Подключение к PostgreSQL, выполнение параметризованных SQL-запросов
- **Встроенная защита от опасных запросов** (`DROP`, `INSERT`, `UPDATE`, `DELETE` блокируются по умолчанию)
- Экспорт результатов в Excel с форматированием таблиц и графиками
- Экспорт в PDF с адаптивными таблицами (автоматический расчёт ширины колонок)
- CLI-интерфейс с гибкими параметрами и флагом `--readonly`
- Поддержка JSON-конфигурации для сложных отчётов (графики, параметры, форматирование)
- Контейнеризация Docker (Dockerfile включён в исходники)
- Модульные тесты с измерением покрытия

## Требования

- **.NET 6.0 SDK** (планируется миграция на .NET 8 LTS)
- **PostgreSQL 12+**
- **Linux (Kali, Ubuntu, Debian):** дополнительные зависимости для QuestPDF (SkiaSharp):
  sudo apt update
  sudo apt install -y libfontconfig1 libfreetype6

    macOS / Windows: дополнительные зависимости не требуются

Быстрый старт

# 1. Скачайте и распакуйте исходники из архива в репозитории
tar -xzf ReportGenerator-source.tar.gz
cd ReportGenerator

# 2. Установите системные зависимости (только Linux)
sudo apt install -y libfontconfig1 libfreetype6

# 3. Восстановите зависимости .NET и соберите проект
dotnet restore
dotnet build -c Release

# 4. Запустите тесты с покрытием
make test-coverage

# 5. Сгенерируйте тестовый отчёт (безопасный режим)
make run

Использование

rptgen [параметры]

Параметры:
  -c, --connection    Строка подключения к PostgreSQL
  -q, --query         SQL-запрос для выполнения (по умолчанию только SELECT)
  -o, --output        Путь к выходному файлу
  -f, --format        Формат отчёта (excel или pdf, по умолчанию excel)
  -t, --title         Заголовок отчёта
  -cfg, --config      Путь к JSON-файлу конфигурации
  -v, --verbose       Подробный вывод
  -ro, --readonly     Блокировать опасные запросы (по умолчанию включено)
  -aw, --allow-write  Разрешить запись в БД (ОПАСНО! Только если уверены)

    ⚠️ ВАЖНО: Безопасность SQL-запросов

    По умолчанию инструмент работает в режиме --readonly и блокирует любые запросы, изменяющие данные: DROP, INSERT, UPDATE, DELETE, TRUNCATE, ALTER, CREATE, EXEC, MERGE, GRANT, REVOKE.

    Если вам действительно нужно выполнить запись — передайте флаг --allow-write, но помните: это может безвозвратно изменить или удалить данные.

Примеры

# Безопасный SELECT-запрос (режим readonly по умолчанию)
rptgen -c "Host=localhost;Database=finance;Username=postgres;Password=postgres" \
       -q "SELECT * FROM monthly_sales WHERE date >= '2024-01-01'" \
       -o /reports/sales.xlsx \
       -t "Продажи за 2024 год"

# PDF-отчёт из файла конфигурации с тремя графиками
rptgen -cfg ./example-config.json -v

# Отключение защиты (ОПАСНО!)
rptgen -c "..." -q "INSERT INTO logs VALUES (NOW())" --allow-write -o report.xlsx

# Через Docker (Dockerfile в корне проекта)
docker build -t report-gen .
docker run -v $(pwd)/reports:/app/reports report-gen \
  -c "Host=host.docker.internal;Database=finance;Username=postgres;Password=postgres" \
  -q "SELECT * FROM accounts" \
  -o /app/reports/output.pdf -f pdf

Структура проекта

ReportGenerator/
├── ReportGenerator-source.tar.gz   # Архив с полными исходниками
├── src/
│   ├── ReportGenerator.Cli/        # CLI-интерфейс (System.CommandLine)
│   │   ├── Program.cs              # Обработка аргументов, валидация SQL
│   │   └── ReportGenerator.Cli.csproj
│   └── ReportGenerator.Core/       # Бизнес-логика
│       ├── Models/ReportConfig.cs  # Модели: конфиг, графики, форматы
│       ├── Data/DatabaseProvider.cs # PostgreSQL через Npgsql + Dapper
│       ├── Exporters/ExcelExporter.cs # Экспорт в Excel (ClosedXML)
│       ├── Exporters/PdfExporter.cs   # Экспорт в PDF (QuestPDF, адаптивные таблицы)
│       ├── SqlValidator.cs         # Валидация SQL-запросов (блокировка DROP и др.)
│       ├── ReportEngine.cs         # Движок генерации отчётов
│       ├── ConfigurationLoader.cs  # Загрузка конфигурации из JSON
│       └── ReportGenerator.Core.csproj
├── tests/
│   └── ReportGenerator.Tests/      # Модульные тесты (xUnit + Moq)
├── example-config.json             # Пример конфига со всеми возможностями
├── appsettings.json                # Настройки подключения
├── Dockerfile                      # Docker-образ
├── Makefile                        # Команды: build, test, test-coverage, docker
└── ReportGenerator.sln             # Solution .NET

Стек технологий
Компонент	Библиотека	Версия
Runtime	.NET 6.0 (LTS до 11.2024)	6.0.400
База данных	Npgsql + Dapper	6.0.11 / 2.1.28
Excel	ClosedXML	0.102.2
PDF	QuestPDF + SkiaSharp	2023.12.6
CLI	System.CommandLine	2.0.0-beta4
Тесты	xUnit + Moq	2.4.2 / 4.18.4
Покрытие	XPlat Code Coverage	встроено в dotnet

    План миграции: Проект будет переведён на .NET 8 LTS (поддержка до 2026) в следующем релизе. Миграция с .NET 6 на 8 практически безболезненна — потребуется только смена TargetFramework.

Устранение неполадок
Ошибка	Решение
NETSDK1045: не поддерживает .NET 8.0	Установите .NET 6.0 SDK: sudo apt install dotnet-sdk-6.0
libSkiaSharp.so not found	sudo apt install libfontconfig1 libfreetype6
ОПАСНЫЙ ЗАПРОС ЗАБЛОКИРОВАН	Используйте --allow-write или замените на SELECT
PostgreSQL connection refused	sudo systemctl start postgresql
Лицензия

MIT
