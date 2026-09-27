# goit-rdb-hw-04 — DML, DDL та складні SQL-вирази (PostgreSQL, Olist)

Домашнє завдання до Теми 4 курсу «Реляційні бази даних».
Робота виконана в Google Colab notebook [`hw4_olist_dml_ddl_joins.ipynb`](hw4_olist_dml_ddl_joins.ipynb), усі клітинки збережено з output.

## Датасет

**Olist Brazilian E-commerce Public Dataset**: ≈ 100 тис. замовлень бразильського маркетплейсу Olist за 2016–2018 роки.

* Kaggle: <https://www.kaggle.com/datasets/olistbr/brazilian-ecommerce>
* Офіційне дзеркало від Olist на GitHub (ті самі CSV): <https://github.com/olist/work-at-olist-data> (папка `datasets/`)

### Спосіб отримання файлів

Notebook отримує дані автоматично й відтворювано, **без Kaggle credentials**:

1. Якщо CSV вже лежать у `data/olist/`, використовуються вони.
2. Інакше виконується `git clone --depth 1 https://github.com/olist/work-at-olist-data.git`, і дані беруться з `work-at-olist-data/datasets/`.
3. Альтернатива (закоментована в notebook): `kagglehub.dataset_download('olistbr/brazilian-ecommerce')`.

CSV не додані в репозиторій: разом це ≈ 60 МБ (без geolocation), а офіційне джерело стабільне.

### Використані таблиці (sample-розмір — повний датасет)

| CSV | raw-таблиця | typed-таблиця | Рядків | Ключ |
|---|---|---|---:|---|
| `olist_customers_dataset.csv` | `olist_customers_raw` | `olist_customers` | 99 441 | `customer_id` |
| `olist_orders_dataset.csv` | `olist_orders_raw` | `olist_orders` | 99 441 | `order_id` |
| `olist_order_items_dataset.csv` | `olist_order_items_raw` | `olist_order_items` | 112 650 | `(order_id, order_item_id)` |
| `olist_products_dataset.csv` | `olist_products_raw` | `olist_products` | 32 951 | `product_id` |
| `olist_sellers_dataset.csv` | `olist_sellers_raw` | `olist_sellers` | 3 095 | `seller_id` |
| `olist_order_reviews_dataset.csv` | `olist_order_reviews_raw` | `olist_order_reviews` | 99 224 | технічний `review_row_id` |

Похідні таблиці: `olist_customer_dim` (96 096, PK `customer_unique_id`), `customer_segments` (96 096), `seller_score` (3 095), `seller_alerts`.

## Як запустити

1. Відкрити `hw4_olist_dml_ddl_joins.ipynb` у Google Colab (File → Upload notebook або «Open from GitHub»).
2. **Runtime → Restart session and run all.**

Перша клітинка встановлює PostgreSQL у VM Colab (`apt-get install postgresql`), запускає сервіс, задає пароль користувачу `postgres` і підключається через SQLAlchemy + psycopg2:

```python
engine = create_engine("postgresql+psycopg2://postgres:postgres@localhost/postgres")
```

Повний прогін займає ≈ 1–2 хв (більшість часу — `apt-get` і завантаження CSV у raw-таблиці). Notebook ідемпотентний: усі таблиці перестворюються через `DROP TABLE IF EXISTS … CASCADE`, а наповнення йде через `INSERT … SELECT` / `ON CONFLICT DO UPDATE`.

## Зміст notebook

| № | Розділ | Що зроблено |
|---|---|---|
| 0 | Setup | PostgreSQL у Colab, `SELECT version()`, отримання CSV |
| 1 | Typed-схема | raw → typed через `INSERT … SELECT`, PK / FK / CHECK / NOT NULL, smoke-test raw vs typed, data-quality перевірки |
| 2 | DML | `INSERT … RETURNING` (seller_alerts), `INSERT … ON CONFLICT DO UPDATE` (seller_score на грануляності seller × order + демонстрація помилки наївного join), `UPDATE … FROM` (`is_late`), `DELETE … RETURNING` |
| 3 | customer_segments | `olist_customer_dim`, DDL з identity PK / FK / UNIQUE / CHECK / NOT NULL / DEFAULT, upsert-сегментація на рівні `customer_unique_id`, перевірка ідемпотентності |
| 4 | JOIN | INNER (4 таблиці), LEFT + COALESCE, FULL OUTER, SELF, anti-join (`LEFT JOIN … IS NULL` та `NOT EXISTS`) у форматі business-question / SQL / interpretation |
| 5 | Агрегати | 5 запитів із GROUP BY, 5 із HAVING, `STRING_AGG`, `ARRAY_AGG`, `JSONB_AGG`, пояснення WHERE vs HAVING |
| 6 | Set ops / window | `UNION`, `INTERSECT`, `EXCEPT`, `ROW_NUMBER()`, `RANK()` + `SUM() OVER`, Reflection |

## Структура репозиторію

```
goit-rdb-hw-04/
├── hw4_olist_dml_ddl_joins.ipynb   # notebook з output
├── README.md
├── .gitignore
└── data/
    └── olist/                      # (опційно) сюди можна покласти CSV Olist
```
