
# 🗂️ Metadata (Описание данных)

Техническое описание набора данных, схемы и трансформаций, применённых в проекте.

## 📌 Общая информация
- **Проект:** E-Commerce Customer Behavior & Sales Analysis
- **Источник:** [Kaggle: E-Commerce Customer Behavior & Sales 2020–2026](https://www.kaggle.com/datasets/meruvakodandasuraj/e-commerce-customer-behavior-and-sales-20202026)
- **Автор датасета:** Meruva Kodanda Suraj
- **Лицензия:** CC BY-SA 4.0
- **Характер данных:** синтетический (учебный)
- **Дата последней проверки:** 2026-09-19
- **Окно анализа:** заказы 2020-01 … 2026-03; регистрации 2011-08-03 … 2026-04-01

## 🗄️ Схема данных

### 1. `customers_clean.csv` — 8 000 строк, единица наблюдения: клиент
| Поле | Тип | Описание |
|---|---|---|
| customer_id | str | Первичный ключ, уникален |
| country, gender, membership_tier | str | Категориальные признаки (очищены от скрытых дублей) |
| age | int | 18–75 |
| registration_date | datetime | Дата регистрации |
| total_orders, total_spend_usd, avg_order_value_usd, days_since_last_purchase | int/float | Lifetime-метрики; не согласованы с orders.csv за окно 2020–2026, как метрики периода не используются |
| churned | int | Целевая переменная (доля оттока ≈ 8,94%) |
| recency, frequency, monetary, R/F/M-скоры, Segment | int/float/str | Добавлены в notebook 03 (витрина customers_with_rfm.csv) |

### 2. `orders_clean.csv` — 25 000 строк, единица наблюдения: заказ
| Поле | Тип | Описание |
|---|---|---|
| order_id | str | Первичный ключ, уникален |
| customer_id | str | Внешний ключ к customers |
| order_date, delivery_date | datetime | Правило delivery_date ≥ order_date: нарушений 0 |
| unit_price_usd, quantity, subtotal_usd, discount_pct, discount_amount_usd, shipping_fee_usd, tax_amount_usd, total_amount_usd | float/int | Арифметическая согласованность: 0 mismatch |
| order_status | str | Delivered (20 497), Returned (2 020), Cancelled (1 456), Processing (1 027) |
| customer_rating | float | 15 749 NaN (63%): содержательные пропуски, импутация не применялась |
| has_rating | int | Производный флаг (шаг 11 ETL) |

### 3. `products_clean.csv` — 140 строк, единица наблюдения: товар
- Ключ (category, product_name), уникален.
- База агрегата: все заказы кроме Cancelled (23 544).
- Поле return_rate не используется (невалидно в источнике).

### 4. `revenue_monthly_clean.csv` — 75 строк, единица наблюдения: месяц (year, month)
- База агрегата: только Delivered-заказы (20 497; выручка 2 585 126,50).
- Поля return_rate и new_customers признаны невалидными и исключены из анализа.

## ⚙️ История трансформаций (ETL)
1. Загрузка 4 CSV из data/raw/.
2. Контроль типов: даты конвертированы в datetime (NaT = 0).
3. Очистка категориальных признаков от скрытых дублей (`.str.strip().str.title()`).
4. Пропуски: customer_rating не импутирован (содержательные пропуски), добавлен флаг has_rating; в ML-пайплайне числовые пропуски заполняются медианой, категориальные — модой.
5. Производные признаки: has_rating, registration_month, tenure_days; RFM-скоры и сегменты; ABC-XYZ-категории.
6. Валидация: уникальность ключей, покрытие периодов, межтабличная согласованность агрегатов, арифметика заказов (0 mismatch).

## ⚠️ Ограничения и допущения
- Синтетическая природа данных: XYZ-анализ вырожден (100% товаров в Z, медианный CV 0.93).
- 337 клиентов не имеют заказов в окне анализа (RFM рассчитан для 7 663 клиентов).
- Lifetime-метрики customers не используются как метрики окна 2020–2026.
- Источник истины для расчётов за окно — orders_clean.csv.

## 📊 BI-витрины и дашборд (добавлено 2026-09-19)

### Витрины (data/marts/)

| Файл | Строк | Grain | Назначение |
|---|---|---|---|
| `mart_customers.csv` | 8 000 | клиент | RFM-сегменты, отток, география, демография |
| `mart_orders.csv` | 25 000 | заказ | Основная таблица фактов; денормализована (содержит клиентские атрибуты) |
| `mart_products.csv` | 140 | товар | ABC/XYZ-анализ, рейтинг, возвраты |
| `mart_monthly.csv` | 75 | месяц | Динамика выручки, AOV, новые/возвращающиеся клиенты |

### BI-дашборд

- **Ссылка:** https://datalens.yandex/gia0wfc25ex2z
- **Паспорт:** [`dashboards/dashboard_passport.md`](dashboards/dashboard_passport.md)
- **Контекст передачи:** [`dashboards/BI_HANDOVER_CONTEXT.md`](dashboards/BI_HANDOVER_CONTEXT.md)
- **Спецификации:** [`dashboards/chart_specs.md`](dashboards/chart_specs.md)

### Ключевые изменения ETL
- `05_bi_marts.ipynb`: денормализация `mart_orders` (merge с `mart_customers` по `customer_id`) — для избежания JOIN в DataLens.
- Добавлено поле `order_month_start` (тип Date) для временных осей и селекторов.

### Контрольные числа для QA
- Заказы: 25 000
- Выручка Delivered: 2 585 126,50
- Клиенты с заказами: 7 663
- Churn Rate: ≈ 8,94%
- Доля группы A: 79,7%

🤖 ML-артефакты 
| Артефакт | Описание |
| --- | --- |
| `reports/churn_model_rf.pkl` | Обученный пайплайн: imputer+StandardScaler / imputer+OHE + RandomForestClassifier (`class_weight='balanced'`) |
| Разбиение | 80/20 со стратификацией (random_state=42): X_train 6 400, X_test 1 600 |
| Метрики (test) | ROC-AUC 0.7367; PR-AUC 0.2612; F1 0.1421; baseline Dummy: Accuracy 0.9106, F1 0.0000 |
| Порог по F1 | 0.28 (Recall 55.9%, Precision 30.7%); для кампаний — Top-10% Risk Scoring |