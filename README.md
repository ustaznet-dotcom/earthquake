#### بسم الله الرحمن الرحيم


# 📦 E-commerce Data Pipeline (Airflow + DuckDB + PostgreSQL)
![SQLite Version](https://img.shields.io/badge/data-engineering-pink)
![Python Version](https://img.shields.io/badge/python-3.12-blue)
![Python Version](https://img.shields.io/badge/postgres-SQL-blue)
![Docker](https://img.shields.io/badge/docker-26A5E4?style=flat&logo=docker&logoColor=white)
![Apache Airflow](https://img.shields.io/badge/apache%20airflow-017CEE?style=flat&logo=apacheairflow&logoColor=white)
![DuckDB](https://img.shields.io/badge/duckdb-FFF000?style=flat&logo=duckdb&logoColor=black)

![Architecture Diagram](images/E_commerce_Data.png)

## 🎯 Задача проекта
Построить надёжный end-to-end data pipeline для загрузки, обработки и аналитики e-commerce данных.

### Проект решает:
- Централизованная загрузка данных из внешнего API
- Хранение сырых данных в Data Lake
- Формирование операционного слоя (ODS)
- Построение аналитических витрин (Data Mart)
- Оркестрация зависимостей между этапами

### Архитектура ориентирована на:
- Идемпотентность
- Воспроизводимость расчётов
- Масштабируемость

## 🏗 Архитектура решения

```
External API (DummyJSON)
        ↓
RAW layer (MinIO / S3, Parquet)
        ↓
ODS layer (PostgreSQL)
        ↓
Data Mart (PostgreSQL)
```

## Слои данных

### RAW
- **Источник**: DummyJSON API
- **Хранилище**: MinIO (S3-совместимое)
- **Формат**: Parquet
- **Назначение**: Сохранение сырых данных

### ODS (Operational Data Store)
- **Схема**: `ods`
- **Таблица**: `fct_products`
- **Загрузка**: Идемпотентная (upsert)
- **Назначение**: Нормализованные операционные данные

### Data Mart (DM)
- **Схема**: `dm`
- **Таблица**: `fct_products_stats`
- **Назначение**: Аналитические агрегаты для BI

## 🧩 Используемые технологии
- **Apache Airflow** — оркестрация пайплайнов
- **DuckDB** — ETL-движок
- **PostgreSQL** — ODS и Data Mart
- **MinIO** — S3-совместимое хранилище
- **Docker Compose** — контейнеризация

## 📁 Структура проекта
```
dags/
├── raw_s3_api.py              # RAW → S3
├── ecommerce_ods_pg.py        # S3 → ODS
└── FCT_dm_ecommerce.py        # ODS → DM
```

## 🔁 Оркестрация (DAG'и)

### `raw_s3_api`
- **Задача**: Загрузка данных из API в S3
- **Формат**: Parquet
- **Особенность**: Daily snapshots

### `ecommerce_ods_pg`
- **Задача**: Загрузка данных из S3 в PostgreSQL ODS
- **Особенность**: Идемпотентность через `ON CONFLICT`
- **Зависимость**: `raw_s3_api`

### `FCT_dm_ecommerce`
- **Задача**: Создание аналитической витрины
- **Особенность**: Агрегация по категориям
- **Зависимость**: `ecommerce_ods_pg`

## 📊 Аналитическая витрина

**Таблица:** `dm.fct_products_stats`

**Example from metabase**

<img src="images/metabase_image.png" alt="MetaBase Analytics Dashboard" width="400" height="200">



**Назначение витрины:**
- Анализ ассортимента
- Мониторинг остатков
- Отслеживание цен и рейтингов
- Интеграция с BI-системами

## ▶️ Запуск проекта

### 1. Запуск инфраструктуры
```bash
docker-compose up -d
```
### 1. Подготовка MinIO
1. Откройте MinIO UI: http://localhost:9000
2. Создайте bucket:
   - Нажмите **Create Bucket**
   - Введите имя (например: `supfun`)
   - Подтвердите создание
3. Создайте Access Keys:
   - Перейдите в **Access Keys** → **Create access key**
   - Скопируйте `Access Key` и `Secret Key`
   - Сохраните в безопасное место

### 2. Настройка Airflow Variables
Перейдите в Airflow UI: http://localhost:8080

**Admin → Variables → +**:

| Key | Value | Описание |
|-----|-------|----------|
| `access_key` | `ваш_access_key` | MinIO Access Key |
| `secret_key` | `ваш_secret_key` | MinIO Secret Key |
| `pg_password` | `postgres` | Пароль PostgreSQL |

### 3. Настройка Airflow Connections
**Admin → Connections → +**:

**PostgreSQL Connection:**
- **Connection ID**: `postgres_dwh`
- **Connection Type**: `Postgres`
- **Host**: `postgres_dwh`
- **Database**: `postgres`
- **Login**: `postgres`
- **Password**: `postgres`
- **Port**: `5432`
- **Schema**: `postgres`

Нажмите **Save**.

### 4. Дополнительно: `.env` файл (рекомендуется)
Создайте файл `.env` в корне проекта:
```bash
# MinIO
MINIO_ACCESS_KEY=ваш_access_key
MINIO_SECRET_KEY=ваш_secret_key

# PostgreSQL
POSTGRES_PASSWORD=postgres

# Airflow
AIRFLOW_UID=1000
```

### 5. Активация DAG'ов
1. Включить DAG'и в Airflow UI
2. Запустить в порядке:
   - `raw_s3_api`
   - `ecommerce_ods_pg`
   - `FCT_dm_ecommerce`

## ⚙️ Ключевые принципы
1. **Разделение слоёв**: RAW / ODS / DM
2. **Идемпотентность**: Все загрузки перезапускаемы
3. **Логическая дата**: Использование `ds` вместо `CURRENT_DATE`
4. **Зависимости**: Чёткий контроль через `ExternalTaskSensor`
5. **Минимальные трансформации**: В RAW сохраняем сырые данные

## 🛠️ Что реализовано
- ✅ End-to-end пайплайн от API до аналитики
- ✅ Идемпотентные загрузки
- ✅ Ежедневные агрегаты
- ✅ Локальный Data Lake (MinIO)
- ✅ Полная оркестрация (Airflow)
- ✅ Готовность к интеграции с BI
