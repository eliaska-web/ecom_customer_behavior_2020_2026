# Отчёт по качеству данных
Датасет: E-Commerce Customer Behavior & Sales 2020–2026 (Kaggle, CC BY-SA 4.0, данные синтетические)
Источник: data/raw/*.csv | Дата проверки: 2026-09-19 | Ноутбук: notebooks/01_etl_and_preprocessing.ipynb

## 1. Объём и источник
- customers: 8 000 × 20; orders: 25 000 × 28; product_summary: 140 × 9; monthly_revenue: 75 × 10
- Владелец датасета: Meruva Kodanda Suraj (Kaggle); лицензия CC BY-SA 4.0

## 2. Единица наблюдения и ключи
- customers: строка = клиент, ключ `customer_id` (уникальность: **подтверждена**)
- orders: строка = заказ, ключ `order_id` (уникальность: **подтверждена**)
- product_summary: строка = продукт, ключ (`category`, `product_name`) (уникальность: **подтверждена**)
- monthly_revenue: строка = месяц, ключ (`year`, `month`) (уникальность: **подтверждена**)

## 3. Структура и типы
- registration_date, order_date, delivery_date конвертированы в datetime (шаг 15): NaT = 0 во всех трёх полях
- Диапазоны: registration_date 2011-08-03 … 2026-04-01; order_date 2020-01-01 … 2026-03-30; delivery_date 2020-01-03 … 2026-04-12
- Бизнес-правило «delivery_date ≥ order_date»: нарушений 0
- Остальные типы соответствуют смыслу полей

## 4. Пропуски и полнота
- `orders.customer_rating`: 15 749 NaN (63,0% строк orders)
- Причина: рейтинг существует только у заказов со статусом `Delivered` (45,1% от всех Delivered, 0% у остальных статусов)
- Распределение статусов: Delivered = 20 497, Returned = 2 020, Cancelled = 1 456, Processing = 1 027
- Стратегия: **не импутовать**; оставить NaN + добавить флаг `has_rating` (шаг 11)
- В остальных таблицах пропусков нет

## 5. Дубликаты и уникальность
- Полных дубликатов строк: 0 во всех четырёх таблицах
- Дубликатов по ключам: 0 (проверено)

## 6. Допустимость значений
- age 18–75; discount_pct 0–50; tax_pct 0–20; customer_rating 1,5–5,0; delivery_days 1–14 — в допустимых границах
- session_duration_minutes: max 361,2 при медиане 12,1 — кандидат в выбросы, проверка IQR на этапе EDA (шаг 20)

## 7. Временное покрытие
- orders: 2020-01 … 2026-03 (75 месяцев, полное покрытие)
- monthly_revenue: 2020-01 … 2026-03 (75 месяцев)
- **Периоды совпадают полностью**: все месяцы orders присутствуют в monthly_revenue и наоборот
- Все 25 000 заказов orders.csv попадают в покрытие monthly_revenue
- registration_date: 2011-08-03 … 2026-04-01 — шире периода заказов (2020-01 … 2026-03):
  регистраций в окне monthly_revenue (2020-01 … 2026-03) = 7 809; вне окна = 191
- sum(monthly_revenue.new_customers) = 7 284 ≠ 7 809 (регистрации в окне) и ≠ 8 000 (все клиенты);
  разрыв внутри окна = 7 809 − 7 284 = 525
- Гипотеза «new_customers = регистрации по месяцам» ОПРОВЕРГНУТА; определение поля требует проверки
  (кандидаты: первый заказ любого статуса / первый Delivered-заказ)

## 8. Межтабличная целостность
- `monthly_revenue.return_rate = 0,0` во всех 75 строках при `orders.returned = 8,08%` → поле **непригодно**; источник истины по возвратам — `orders.returned`
- `monthly_revenue` агрегирует **только заказы со статусом Delivered** (подтверждено):
- `sum(monthly_revenue.orders)` = 20 497 = количество заказов со статусом Delivered в orders.csv
  - `sum(monthly_revenue.revenue_usd)` = 2 585 126.5 = выручка по заказам со статусом Delivered в orders.csv
  - Заказы со статусами Returned (2 020), Cancelled (1 456), Processing (1 027) не включены в агрегат
- Для точных расчётов по всем заказам использовать `orders.csv`; `monthly_revenue` применять только для динамики Delivered-выручки
- **Природа пропусков `customer_rating`** (15 749 NaN):
  - Рейтинг существует только у заказов со статусом `Delivered`
  - Среди Delivered рейтинг получили ~45% заказов
  - У статусов Cancelled, Processing, Returned рейтинга нет
  - Это содержательный пропуск (оценка не поставлена), не ошибка сбора
- `product_summary` агрегирует **все заказы кроме статуса Cancelled** (подтверждено):
  - `sum(product_summary.total_orders)` = 23 544 = 25 000 − 1 456 (Cancelled)
  - `sum(product_summary.total_revenue_usd)` = выручка по всем заказам кроме Cancelled
  - Для анализа продуктов использовать `product_summary`; для анализа всех статусов — агрегировать `orders.csv`
- new_customers vs регистрации: гипотеза опровергнута, факты и разрыв 525 — см. раздел 7
## 9. Аномалии и бизнес-правила
- churned = 8,94% — дисбаланс классов, учесть на ML-этапе (п. 4* задания)
- Арифметика заказа сходится без расхождений (0 mismatch): subtotal = unit_price × quantity; discount_amount = subtotal × discount_pct / 100; total = subtotal − discount + shipping + tax
- delivery_date ≥ order_date: нарушений 0

## 10. Риски для анализа
- `product_summary` пригодна для продуктовых метрик только с явной оговоркой «агрегат исключает Cancelled» (~6% заказов)
- `monthly_revenue.new_customers` нельзя использовать как метрику регистраций до подтверждения определения поля
- JOIN orders и monthly_revenue по месяцам требует учёта: monthly_revenue агрегирует только Delivered; product_summary агрегирует все кроме Cancelled
- Для когортного анализа retention использовать orders.csv, а не monthly_revenue.new_customers (определение поля не подтверждено, см. разделы 7–8)
- Импутация customer_rating исказит статистику рейтинга

## 11. Рекомендованные исправления
- Исключить `monthly_revenue.return_rate` из анализа, пометить как невалидное
- Добавить флаг `has_rating`, рейтинг не импутовать
- Конвертировать даты в datetime
- Для расчётов по всем заказам использовать `orders.csv`; `monthly_revenue` использовать только для анализа динамики Delivered-выручки с явным указанием фильтра
- В отчётах и дашборде при использовании product_summary явно указывать базу агрегата: все заказы кроме Cancelled
- Для анализа продуктов использовать `product_summary` (все кроме Cancelled); для анализа всех статусов агрегировать `orders.csv`
- Для когортного анализа рассчитывать retention из `orders.csv`, не использовать `monthly_revenue.new_customers`

## 12. Решение о пригодности
- Данные пригодны для EDA, RFM/ABC-XYZ и дашборда с оговорками разделов 8 и 10;
monthly_revenue пригодна для динамики Delivered-выручки: покрытие периодов подтверждено (раздел 7),
база агрегата подтверждена (раздел 8); поля return_rate и new_customers из использования исключены

