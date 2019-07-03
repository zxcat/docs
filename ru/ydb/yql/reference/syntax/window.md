# OVER, PARTITION BY и WINDOW

Механизм оконных функций, появившийся в стандарте SQL:2003 и расширенный в стандарте SQL:2011, позволяет выполнять вычисления над набором строк таблицы, который некоторым образом соотносится с текущей строкой.

С помощью `PARTITION BY` таблицу можно разбить на партиции по значению переданного следом выражения, по умолчанию вся таблица считается одной партицией. Строки внутри партиции могут быть дополнительно отсортированы с помощью [ORDER BY](select.md#order-by).

{% note alert %}

По стандарту в рамках партиции должна быть возможность определить набор строк, над которыми будут выполняться сами оконные функции (положение окна), но на данный момент доступно только положение по умолчанию: **от начала партиции и до текущей строки включительно**.

{% endnote %}

На данный момент все настройки окна можно указывать только с помощью ключевого слова `WINDOW`, создающего именованное окно (то есть анонимные пока не поддержаны). Для вызова функций на окне используется ключевое слово `OVER`. Примеры приведены ниже.

В отличие от [GROUP BY](group_by.md), группировки по ключам партиционирования не происходит, таким образом при вызове оконных функций остается полноценный доступ к текущей строке.

{% if audience == "internal" %}

В терминах MapReduce оконные функции физически выполняются через Reduce по ключам партиционирования, что может означать длительное выполнение для партиций большого размера, а также жёсткий лимит в 200Гб на партицию для основных кластеров YT.

{% endif %}

Список доступных оконных функций приведен в разделе [Встроенные функции](../builtins/window.md).

**Примеры**

``` yql
SELECT
    COUNT(*) OVER w AS rows_count_in_window,
    some_other_value -- доступ к текущей строке
FROM my_table
WINDOW w AS (
    PARTITION BY partition_key_column
    ORDER BY int_column
);
```

``` yql
SELECT
    LAG(my_column, 2) OVER w AS row_before_previous_one
FROM my_table
WINDOW w AS (
    PARTITION BY partition_key_column
);
```
