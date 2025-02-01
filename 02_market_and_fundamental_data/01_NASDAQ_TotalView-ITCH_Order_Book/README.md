# Работа с рыночными данными (Working with Market Data): книга заявок NASDAQ_TotalView-ITCH (NASDAQ_TotalView-ITCH Order Book)  

Хотя **FIX-протокол (FIX Protocol)** занимает **доминирующую долю рынка (Dominant Market Share)**, биржи также предлагают **нативные протоколы (Native Protocols)**.  

Nasdaq предоставляет **прямой поток данных TotalView ITCH (TotalView ITCH Direct Data-Feed Protocol)**, который позволяет подписчикам **отслеживать индивидуальные ордера (Track Individual Orders) на акции (Equity Instruments) с момента их размещения до исполнения или отмены (From Placement to Execution or Cancellation)**.  

### Восстановление книги заявок (Order Book Reconstruction)  

Этот протокол позволяет **воссоздать книгу заявок (Reconstruct the Order Book)**, которая **отслеживает список активных лимитных ордеров (List of Active-Limit Buy and Sell Orders)** для конкретного **финансового инструмента (Financial Instrument) или ценной бумаги (Security)**.  

**Книга заявок (Order Book)** раскрывает **глубину рынка (Market Depth)**, показывая **объём заявок на покупку и продажу (Number of Shares Being Bid or Offered) на каждом уровне цены (At Each Price Point)** в течение торгового дня.  

Она также может **идентифицировать участников рынка (Identify the Market Participants)**, разместивших **определённые ордера (Specific Buy and Sell Orders)**, если сделки **не были размещены анонимно (Unless Placed Anonymously)**.  

**Глубина рынка (Market Depth)** является **ключевым индикатором ликвидности (Key Indicator of Liquidity)** и помогает оценить **возможное влияние крупных рыночных ордеров (Potential Price Impact of Sizable Market Orders)**.  

### Аукционы и кросс-сделки Nasdaq (Nasdaq Auctions and Crosses)  

Помимо **сопоставления рыночных (Market Orders) и лимитных ордеров (Limit Orders)**, Nasdaq также проводит **аукционы (Auctions) или кросс-сделки (Crosses)**, которые позволяют исполнять **большое количество сделок (Large Number of Trades)** при **открытии и закрытии рынка (At Market Opening and Closing)**.  

**Кросс-сделки (Crosses)** приобретают **всё большее значение (Becoming More Important)** по мере роста **пассивного инвестирования (Passive Investing)**, поскольку трейдеры ищут **возможности для исполнения крупных пакетов акций (Opportunities to Execute Larger Blocks of Stock)**.  

**TotalView** также **распространяет (Disseminates)** **Индикатор дисбаланса заявок (Net Order Imbalance Indicator, NOII)**, который используется для:  
- **Открывающих и закрывающих кросс-сделок Nasdaq (Nasdaq Opening and Closing Crosses)**  
- **Кросс-сделок IPO и остановленных торгов (Nasdaq IPO/Halt Cross)**  

### Требования к памяти  

> Данный пример **требует значительного объёма оперативной памяти (Requires Plenty of Memory)**, вероятно, **более 16GB** (я использую **64GB** и пока не проверял **минимальные требования (Minimum Requirement)**).  
>  
> Если у вас **ограниченные ресурсы (Capacity Constraints)**, помните, что **запуск кода не является критически важным для дальнейшего изучения книги (Not Essential for Anything Else in This Book)**.  
>  
> Этот пример **призван продемонстрировать (Aims to Demonstrate)**, **какие данные используются в институциональном инвестировании (What Kind of Data You Would Be Working With in an Institutional Investment Context)**, где системы **обрабатывают значительно большие объёмы данных (Manage Data Much Larger)**, чем **в данном примере за один торговый день (This Single-Day Example)**. 

## Разбор бинарных сообщений ITCH (Parsing Binary ITCH Messages)  

**Спецификация ITCH v5.0 (ITCH v5.0 Specification)** определяет **более 20 типов сообщений (Over 20 Message Types)**, относящихся к:  
- **Системным событиям (System Events)**  
- **Характеристикам акций (Stock Characteristics)**  
- **Размещению и модификации лимитных ордеров (Placement and Modification of Limit Orders)**  
- **Исполнению сделок (Trade Execution)**  

Также содержится информация о **дисбалансе ордеров (Net Order Imbalance)** перед **открытием (Before the Open) и закрытием рынка (Closing Cross)**.  

Nasdaq предоставляет **образцы (Samples)** **ежедневных бинарных файлов (Daily Binary Files)** за несколько месяцев.  

Репозиторий на **GitHub** для этой главы содержит **Jupyter Notebook** `build_order_book.ipynb`, который демонстрирует:  
- **Как разобрать (Parse) файл с ITCH-сообщениями (ITCH Messages)**  
- **Как воссоздать (Reconstruct) как исполненные сделки (Executed Trades), так и книгу заявок (Order Book) для любого тикера (Any Given Tick)**  

---

### Частота наиболее распространённых типов сообщений  

В следующей таблице показана **частота встречаемости (Frequency) наиболее распространённых типов сообщений (Most Common Message Types)** для **образца файла за 29 марта 2018 года (Sample File Date: March 29, 2018)**.  
Код был обновлён для работы с новым файлом от **27 марта 2019 года (March 27, 2019)**.  

| Тип сообщения (Message Type) | Влияние на книгу заявок (Order Book Impact)                                       | Количество сообщений (Number of Messages) |
|:----------------------------:|---------------------------------------------------------------------------------|----------------------------------------:|
| A                            | Новый лимитный ордер без атрибуции (New Unattributed Limit Order)                | 136,522,761                            |
| D                            | Отмена ордера (Order Canceled)                                                  | 133,811,007                            |
| U                            | Отмена и замена ордера (Order Canceled and Replaced)                            | 21,941,015                             |
| E                            | Полное или частичное исполнение (Full or Partial Execution)                      | 6,687,379                              |
| X                            | Модификация после частичной отмены (Modified After Partial Cancellation)        | 5,088,959                              |
| F                            | Добавление ордера с атрибуцией (Add Attributed Order)                           | 2,718,602                              |
| P                            | Торговое сообщение (не кросс) (Trade Message, Non-Cross)                        | 1,120,861                              |
| C                            | Полное или частичное исполнение по цене, отличной от первоначально указанной (Executed at Different Price) | 157,442                                |
| Q                            | Кросс-сделка (Cross Trade Message)                                              | 17,233                                 |

---

### Структура сообщений ITCH  

Каждое сообщение содержит **набор полей (Components)** со **своей длиной (Length) и типом данных (Data Type)**:  

| Название (Name)            | Смещение (Offset) | Длина (Length) | Значение (Value) | Описание (Notes) |
|---------------------------|------------------|--------------|----------------|--------------------------------------------------------------------------------------|
| Тип сообщения (Message Type) | 0                | 1            | S              | Сообщение о системном событии (System Event Message)                                |
| Локатор акции (Stock Locate) | 1                | 2            | Integer        | Всегда 0 (Always 0)                                                                  |
| Номер отслеживания (Tracking Number) | 3 | 2 | Integer | Внутренний номер отслеживания Nasdaq (Nasdaq Internal Tracking Number) |
| Временная метка (Timestamp) | 5 | 6 | Integer | Количество наносекунд с полуночи (Nanoseconds Since Midnight) |
| Номер ордера (Order Reference Number) | 11 | 8 | Integer | Уникальный номер, присвоенный ордеру при получении (Unique Reference Number for New Order) |
| Индикатор покупки/продажи (Buy/Sell Indicator) | 19 | 1 | Alpha | Тип ордера: B = покупка (Buy Order), S = продажа (Sell Order) |
| Количество акций (Shares) | 20 | 4 | Integer | Общее количество акций в ордере (Total Number of Shares in Order) |
| Акция (Stock) | 24 | 8 | Alpha | Символ акции, дополненный пробелами справа (Stock Symbol, Right-Padded with Spaces) |
| Цена (Price) | 32 | 4 | Price (4) | Цена ордера (Display Price of the New Order) |
| Атрибуция (Attribution) | 36 | 4 | Alpha | Идентификатор участника рынка Nasdaq, разместившего ордер (Nasdaq Market Participant Identifier) |

---

### Ноутбуки и код  

В **ноутбуках (Notebooks)**:  
- **[01_build_itch_order_book](01_parse_itch_order_flow_messages.ipynb)**  
- **[02_rebuild_nasdaq_order_book](02_rebuild_nasdaq_order_book.ipynb)**  
- **[03_normalize_tick_data](03_normalize_tick_data.ipynb)**  

Содержится код для:  
- **Загрузки (Download) тиковых данных (Tick Data) из NASDAQ TotalView**  
- **Разбора сообщений (Parse Messages) из бинарного источника данных (Binary Source Data)**  
- **Восстановления книги заявок (Reconstruct the Order Book) для конкретной акции (Given Stock)**  
- **Визуализации данных потока ордеров (Visualize Order Flow Data)**  
- **Нормализации тиковых данных (Normalize Tick Data)**  

Код был обновлён для использования **новейшего образца файла NASDAQ (Latest NASDAQ Sample File) от 27 марта 2019 года (March 27, 2019)**.  

---

### Внимание: требования к ресурсам  

> **Размер тиковых данных (Tick Data Size)** ~12GB.  
> **Некоторые этапы обработки (Processing Steps) могут занять несколько часов** на **4-ядерном процессоре i7 (4-Core i7 CPU) с 32GB оперативной памяти (32GB RAM)**.

## Регуляризация тиковых данных (Regularizing Tick Data)  

**Данные о сделках (Trade Data)** индексируются в **наносекундах (Nanoseconds)** и содержат **значительный уровень шума (Very Noisy)**.  

Например, **эффект колебания бид-аска (Bid-Ask Bounce)** приводит к **осцилляции цены (Price Oscillation)** между **ценами спроса (Bid Prices) и предложения (Ask Prices)**, когда инициирование сделок **чередуется между рыночными ордерами на покупку и продажу (Trade Initiation Alternates Between Buy and Sell Market Orders)**.  

### Цель регуляризации  

Чтобы **улучшить соотношение сигнал-шум (Improve the Noise-Signal Ratio)** и **улучшить статистические свойства данных (Improve Statistical Properties)**, необходимо **переобразовать (Resample) и регуляризовать (Regularize) тиковые данные (Tick Data)**, агрегируя торговую активность (Aggregating the Trading Activity).  

### Методы агрегации  

Обычно мы собираем:  
- **Открытие (Open)** — первая цена периода (First Price of the Aggregated Period)  
- **Минимум (Low)** — самая низкая цена периода (Lowest Price)  
- **Максимум (High)** — самая высокая цена периода (Highest Price)  
- **Закрытие (Close)** — последняя цена периода (Last Price)  
- **Средневзвешенную цену по объёму (VWAP, Volume-Weighted Average Price)**  
- **Общее количество акций, участвовавших в сделках (Number of Shares Traded)**  
- **Временную метку (Timestamp), связанную с данными (Associated with the Data)**  

### Пример нормализации тиковых данных  

В ноутбуке **[03_normalize_tick_data](03_normalize_tick_data.ipynb)** показано, **как нормализовать (Normalize) шумные тиковые данные (Noisy Tick Data)** с использованием **временных баров (Time Bars)** и **объёмных баров (Volume Bars)**, применяя **различные методы агрегации (Different Aggregation Methods)**.