# 🛒 E-Commerce Customer Behavior & Sales (2020–2026)

[![Python](https://img.shields.io/badge/Python-3.13-blue.svg)](https://www.python.org/)
[![License](https://img.shields.io/badge/License-CC%20BY--SA%204.0-green.svg)](https://creativecommons.org/licenses/by-sa/4.0/)

Сквозной аналитический проект по данным интернет-магазина: аудит качества данных, EDA, сегментация клиентов (RFM) и товарной матрицы (ABC-XYZ), модель прогнозирования оттока.

> Датасет синтетический (Kaggle, CC BY-SA 4.0). Выводы демонстрируют применение методик и не являются рекомендациями для реального бизнеса.

## 📑 Оглавление
- [Цели и задачи](#-цели-и-задачи)
- [Описание данных](#-описание-данных)
- [Ключевые инсайты](#-ключевые-инсайты)
- [BI-дашборд](#-bi-дашборд)
- [Структура проекта](#-структура-проекта)
- [Как запустить проект](#-как-запустить-проект)
- [Отчеты и артефакты](#-отчеты-и-артефакты)
- [Ограничения проекта](#-ограничения-проекта)
- [Автор](#-автор)

## 🎯 Цели и задачи
1. Аудит качества данных и очистка (ETL).
2. Разведочный анализ (EDA): распределения, выбросы, сезонность, корреляции.
3. Сегментация: RFM для клиентов, ABC-XYZ для товаров.
4. Модель предсказания оттока (`churned`) с учётом дисбаланса классов.
5. Подготовка витрин данных и сборка BI-дашборда в Yandex DataLens.

## 📊 Описание данных
Источник: [Kaggle — E-Commerce Customer Behavior & Sales 2020–2026](https://www.kaggle.com/datasets/meruvakodandasuraj/e-commerce-customer-behavior-and-sales-20202026), автор Meruva Kodanda Suraj, лицензия CC BY-SA 4.0.
- `customers.csv` — 8 000 клиентов: профили, RFM-метрики, метка оттока.
- `orders.csv` — 25 000 заказов: цены, скидки, доставка, статусы, сессии.
- `product_summary.csv` — 140 товаров: агрегированные продажи и рейтинги.
- `monthly_revenue.csv` — 75 месяцев: выручка и метрики месяца.

Период: заказы 2020-01 … 2026-03; регистрации 2011-08-03 … 2026-04-01.

## 💡 Ключевые инсайты

### 1. Качество данных
- `monthly_revenue` агрегирует только Delivered-заказы (20 497 из 25 000); поля `return_rate` и `new_customers` признаны невалидными и исключены из анализа.
- `product_summary` агрегирует все заказы кроме Cancelled (23 544).
- Пропуски `customer_rating` (63% заказов) содержательные: оценки есть только у ~45% Delivered; добавлен флаг `has_rating`, импутация не применялась.

### 2. Сегментация (RFM и ABC-XYZ)
- RFM: Loyal Customers — 69,2% выручки; At Risk — 24,0% (целевая группа реактивационных кампаний).
- ABC: группа A (62 товара) — 79,7% выручки.
- XYZ: 100% товаров в группе Z (медианный CV = 0,93 при пороге 0,25) — следствие волатильности спроса на уровне SKU; матрица вырождена, решения опираются на ABC.

---

## 📊 BI-дашборд

**Публичная ссылка:** https://datalens.yandex/gia0wfc25ex2z

Дашборд в Yandex DataLens: 4 вкладки, 5 KPI-индикаторов и 19 чартов по спецификации `dashboards/chart_specs.md`.

- **Executive** — KPI, динамика выручки (Delivered), категории, RFM-сегменты, структура статусов, топ-10 товаров.
- **Customers** — сегменты, давность At Risk, отток по тарифам, топ-10 стран, возраст.
- **Products** — ABC-группы, топ-20 товаров, рейтинг × выручка, return rate, распределение CV.
- **Trends** — статусы по месяцам (100 %), динамика AOV, новые vs возвращающиеся, сезонность год × месяц.

Артефакты BI:
- [Паспорт дашборда](dashboards/dashboard_passport.md) — назначение, KPI, селекторы, ограничения, контроль качества.
- [Спецификации чартов](dashboards/chart_specs.md) + страницы [Executive](dashboards/specs_page1_executive.md), [Customers](dashboards/specs_page2_customers.md), [Products](dashboards/specs_page3_products.md), [Trends](dashboards/specs_page4_trends.md).
- [Контекст передачи проекта](dashboards/BI_HANDOVER_CONTEXT.md) — формулы, контрольные числа, журнал решений для продолжения работы.

### 3. Машинное обучение (churn prediction)
- RandomForestClassifier с `class_weight='balanced'`: ROC-AUC = 0.737, PR-AUC = 0.261.
- Бизнес-применение — Top-10% Risk Scoring: маркетинг отрабатывает дециль с максимальной вероятностью оттока, где концентрация оттока кратно выше базовой.

---

## 📊 BI-дашборд

**Публичная ссылка:** https://datalens.yandex/gia0wfc25ex2z

Дашборд в Yandex DataLens: 4 вкладки, 5 KPI-индикаторов и 19 чартов по спецификации `dashboards/chart_specs.md`.

- **Executive** — KPI, динамика выручки (Delivered), категории, RFM-сегменты, структура статусов, топ-10 товаров.
- **Customers** — сегменты, давность At Risk, отток по тарифам, топ-10 стран, возраст.
- **Products** — ABC-группы, топ-20 товаров, рейтинг × выручка, return rate, распределение CV.
- **Trends** — статусы по месяцам (100 %), динамика AOV, новые vs возвращающиеся, сезонность год × месяц.

Артефакты BI:
- [Паспорт дашборда](dashboards/dashboard_passport.md) — назначение, KPI, селекторы, ограничения, контроль качества.
- [Спецификации чартов](dashboards/chart_specs.md) + страницы [Executive](dashboards/specs_page1_executive.md), [Customers](dashboards/specs_page2_customers.md), [Products](dashboards/specs_page3_products.md), [Trends](dashboards/specs_page4_trends.md).
- [Контекст передачи проекта](dashboards/BI_HANDOVER_CONTEXT.md) — формулы, контрольные числа, журнал решений для продолжения работы.


## 📂 Структура проекта

```text
ecom_customer_behavior_2020_2026/
├── data/
│   ├── raw/          # Исходные CSV (не изменяются)
│   ├── processed/    # Очищенные данные и витрины (RFM, ABC-XYZ)
│   └── marts/        # Витрины для BI: mart_customers, mart_orders, mart_products, mart_monthly
├── notebooks/
│   ├── 01_etl_and_preprocessing.ipynb
│   ├── 02_eda.ipynb
│   ├── 03_rfm_abc_xyz.ipynb
│   ├── 04_ml_churn_prediction.ipynb
│   └── 05_bi_marts.ipynb      # построение витрин для DataLens + 13 проверок качества
├── reports/
│   ├── data_quality_report.md
│   └── analysis_report.md
├── dashboards/
│   ├── chart_specs.md             # паспорт и спецификации 19 чартов
│   ├── specs_page1_executive.md
│   ├── specs_page2_customers.md
│   ├── specs_page3_products.md
│   ├── specs_page4_trends.md
│   ├── dashboard_passport.md      # паспорт BI-дашборда
│   └── BI_HANDOVER_CONTEXT.md     # контекст передачи проекта
├── README.md
├── METADATA.md
└── requirements.txt
```

## 🚀 Как запустить проект

1. Клонируйте репозиторий:
   ```bash
   git clone https://github.com/eliaska-web/ecom_customer_behavior_2020_2026.git
   cd ecom_customer_behavior_2020_2026
   ```
2. Создайте и активируйте окружение:
   ```bash
   python3 -m venv .venv
   source .venv/bin/activate
   ```
3. Установите зависимости:
   ```bash
   pip install -r requirements.txt
   ```
4. Запускайте ноутбуки последовательно 01 → 05:
   ```bash
   jupyter notebook
   ```
   После запуска `05_bi_marts.ipynb` витрины сохраняются в `data/marts/` и готовы к загрузке в DataLens (13 проверок качества должны быть OK, snapshot 2026-03-30).

## 📈 Отчеты и артефакты
- [Отчет о качестве данных](reports/data_quality_report.md) — аудит 4 таблиц, проверенные гипотезы, бизнес-правила.
- [Итоговый аналитический отчет](reports/analysis_report.md) — бизнес-резюме и рекомендации.
- [BI-дашборд (публичный)](https://datalens.yandex/gia0wfc25ex2z) — 4 вкладки, 5 KPI, 19 чартов; паспорт и спецификации в папке `dashboards/`.

## ⚠️ Ограничения проекта
1. Данные синтетические; паттерны (включая 100% товаров в XYZ-Z) — артефакт генерации.
2. Окно анализа 2020–2026: у 191 клиента, зарегистрированного до 2020 года, нет полной истории в `orders`.
3. Поля `customers.total_orders`, `total_spend_usd`, `days_since_last_purchase` не согласованы с `orders.csv` за окно и не используются как метрики периода.
4. Метка `churned` сгенерирована; внешние факторы оттока модель не учитывает.
5. Базы агрегатов различаются: выручка по умолчанию = Delivered (20 497 заказов), продуктовые метрики = все заказы кроме Cancelled (23 544); сравнивать эти базы напрямую нельзя.
6. Клиентские метрики дашборда — снимок на 2026-03-30; селектор периода на них не влияет.
7. Поле `monthly_revenue.new_customers` невалидно; в дашборде используется `new_customers_calc` (первый заказ в окне 2020-01…2026-03).
## 👤 Автор
**Быкова Алина**, Data Analyst
- GitHub: [eliaska-web](https://github.com/eliaska-web)