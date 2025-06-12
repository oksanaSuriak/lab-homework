# 📄 1_extract_data_description.md

##  Опис логіки вигрузки даних з Yahoo Finance

### Мета

Отримати історичні та щоденні біржові дані про ціни акцій обраних компаній з Yahoo Finance, у форматі, придатному для подальшої обробки та збереження.

---

## Список компаній (tickers)

```
| Ticker | Company                  |
|--------|--------------------------|
| META   | Meta Platforms, Inc.     |
| AAPL   | Apple Inc.               |
| GOOG   | Alphabet Inc.            |
| TSLA   | Tesla, Inc.              |
| AMZN   | Amazon.com, Inc.         |
| MSFT   | Microsoft Corporation    |
| NVDA   | NVIDIA Corporation       |



```

Це — список найбільших публічних технологічних компаній, дані яких завантажуються щоденно.

---

## Періоди завантаження

Код реалізує **інкрементальне завантаження** за допомогою умовного блоку:

```
if today_date > fixed_end_date:
    # Інкрементальне оновлення
else:
    # Повне історичне завантаження
```

- **Історичне завантаження:**

  - Виконується до певної фіксованої дати (`fixed_end_date = "2025-06-10"`).
  - Охоплює період з `2020-01-01` до `2025-06-10`.
  - Дані зберігаються у файлі з заголовками.

- **Інкрементальне оновлення:**
  - Запускається, якщо сьогоднішня дата перевищує `fixed_end_date`.
  - Завантажує тільки **дані за вчорашній день** (`start=yesterday_date`, `end=today_date`).
  - Дані додаються у CSV без заголовків, щоб уникнути дублювання схем.

---

## Формат даних

Використовується API бібліотеки `yfinance`:

```
data = yf.download(tickers, start=..., end=..., group_by="ticker")
```

- Дані групуються по тикерах.
- Кожна компанія має свої стовпці: `Open`, `High`, `Low`, `Close`, `Volume`, `Adj Close`.

---

## Об’єднання в єдиний DataFrame

```
for ticker in tickers:
    df = data[ticker].copy()
    df["Company"] = company_names[ticker]
    formatted_data[ticker] = df
final_df = pd.concat(formatted_data.values())
```

- Дані з кожної компанії обробляються окремо.
- Додається колонка `Company` з повною назвою компанії.
- Всі дані об’єднуються у фінальний датафрейм.

---

## Фінальна очистка

```
final_df.reset_index(inplace=True)
final_df.rename(columns={"index": "Date"}, inplace=True)
final_df = final_df[["Date","Company","Open","High","Low","Close","Volume"]]
```

- Переназвано колонку з датою (`index` → `Date`)
- Вибрано ключові поля для аналізу

---

## Збереження

```
final_df.to_csv(filename, index=False, header=header_required)
```

- CSV-файл створюється з іменем: `stock_prices_YYYY_MM_DD.csv`
- Наявність заголовків залежить від типу завантаження:
  - Історичне — `header=True`
  - Інкрементальне — `header=False`

---

## Бізнес-цінність

- **Історичне завантаження:** формує базову базу для моделювання, аналітики, ML.
- **Інкрементальні оновлення:** дозволяють оновлювати дані щодня без дублювання.
- Збереження у CSV робить дані сумісними для подальшого використання в ETL-процесах, Apache Spark, Databricks або BI-системах.
