# Практические задания из учебника PostgreSQL. Основы языка SQL: учеб. пособие / Е. П. Моргунов.
Данные практические задания выполнены в рамках курса "Язык SQL", который был организован и проведен компанией [PostgresPRO](https://postgrespro.ru/) в онлайн-формате в период с 25 марта 2023г. по 20 июня 2023г. Курс был разработан на основе учебника  [PostgreSQL. Основы языка SQL: учеб. пособие / Е. П. Моргунов.](https://postgrespro.ru/education/books/sqlprimer). Практические задания брались оттуда-же и выполнялись на основе [демонстрационной базы данных "Авиаперевозки".](https://postgrespro.ru/education/demodb)

Ниже представлены мои решения практических заданий курса. Решения проверены лектором. Это означает, что они правильные, хотя и не идеальные. Сами по себе задания являются продолжением и дополнением глав учебника и, зачастую, текст и код задания сильно превышает код решения. Не смотря на это, я включил полный текст задания, чтобы не обращаться дополнительно к учебнику.
## Глава 2. Создание рабочей среды
<details>
<summary>Задание 2.1 (установка PostgreSQL)</summary>
1. Выполните процедуру установки СУБД PostgreSQL в среде выбранной вами операционной системы.

  **Решение 2.1:**
  
```
$ postgres -V
postgres (PostgreSQL) 14.7 (Ubuntu 14.7-1.pgdg22.04+1)
```

</details>
<details>
<summary>Задание 2.4 (подключение к БД)</summary>
4. Разверните учебную базу данных. Попробуйте подключиться к ней с помощью
утилиты psql. Для выхода из утилиты используйте команду \q.
  
**Решение 2.4:**
 
```sql
postgres=# \c demo
You are now connected to database "demo" as user "postgres".
demo=# SET search_path TO bookings;
SET
demo=# \d
                        List of relations
  Schema  |         Name          |       Type        |  Owner
----------+-----------------------+-------------------+----------
 bookings | aircrafts             | table             | postgres
 bookings | aircrafts_seats       | view              | postgres
 bookings | aircrafts_tmp         | table             | postgres
 bookings | airports              | table             | postgres
 bookings | boarding_passes       | table             | postgres
 bookings | bookings              | table             | postgres
 bookings | flights               | table             | postgres
 bookings | flights_flight_id_seq | sequence          | postgres
 bookings | flights_v             | view              | postgres
 bookings | index_stat            | table             | postgres
 bookings | modes                 | table             | postgres
 bookings | nulls                 | table             | postgres
 bookings | routes                | materialized view | postgres
 bookings | seats                 | table             | postgres
 bookings | ticket_flights        | table             | postgres
 bookings | tickets               | table             | postgres
(16 rows)

demo=# \q
postgres@lptp:~$
```

</details>

## Глава 3. Основные операции с таблицами
<details>
<summary>Задание 3.1 (constraint)</summary>
1. Попробуйте ввести в таблицу aircrafts строку с таким значением атрибута «Код самолета» (aircraft_code), которое вы уже вводили, например:

```sql
INSERT INTO aircrafts
VALUES ( 'SU9', 'Sukhoi SuperJet-100', 3000 );
```

Обратите внимание, что в этой команде мы не привели список атрибутов, что вполне допустимо при задании значений атрибутов в том же порядке, в котором атрибуты следуют в определении таблицы. Но в ваших прикладных программах так поступать все же не следует, поскольку в случае возможной реструктуризации таблицы и изменения порядка следования атрибутов в ней ваши команды INSERT могут перестать работать корректно. Вы получите сообщение об ошибке.

```sql
ОШИБКА: повторяющееся значение ключа нарушает ограничение
уникальности "aircrafts_pkey"
ПОДРОБНОСТИ: Ключ "(aircraft_code)=(SU9)" уже существует.
```

Подумайте, почему появилось сообщение. Если вы забыли структуру таблицы aircrafts, то можно вывести ее определение на экран с помощью команды
\d aircrafts

**Решение 3.1:**

```sql
demo=# INSERT INTO aircrafts
demo=# VALUES ( 'SU9', 'Sukhoi SuperJet-100', 3000 );
ERROR:  duplicate key value violates unique constraint "aircrafts_pkey"
DETAIL:  Key (aircraft_code)=(SU9) already exists.

demo=# --Сообщение об ошибке появилось потому что в таблице aircrafts
demo=# --атрибут aircraft_code является первичным ключем с уникальными значениями.
demo=# --Попытка присвоить ему дублирующееся значение 'SU9' приводит к ошибке. 

demo=# \d aircrafts;
                   Table "public.aircrafts"
    Column     |     Type     | Collation | Nullable | Default
---------------+--------------+-----------+----------+---------
 aircraft_code | character(3) |           | not null |
 model         | text         |           | not null |
 range         | integer      |           | not null |
Indexes:
    "aircrafts_pkey" PRIMARY KEY, btree (aircraft_code)
Check constraints:
    "aircrafts_range_check" CHECK (range > 0)
Referenced by:
    TABLE "seats" CONSTRAINT "seats_aircraft_code_fkey" FOREIGN KEY (aircraft_code) REFERENCES aircrafts(aircraft_code) ON DELETE CASCADE
```

</details>
<details>
<summary>Задание 3.2 (ORDER BY)</summary>
2. Предложение ORDER BY команды SELECT позволяет отсортировать данные при выводе. По умолчанию сортировка выполняется по возрастанию значений атрибута, указанного в этом предложении. Но можно упорядочить строки и по убыванию значения атрибута. Для этого нужно после имени атрибута в предложении ORDER BY добавить ключевое слово DESC (это сокращение от слова descendant — убывающий порядок). Самостоятельно напишите команду для выборки всех строк из таблицы aircrafts, чтобы строки были упорядочены по убыванию значения атрибута «Максимальная дальность полета, км» (range).

**Решение 3.2:**

```sql
demo=# SELECT * FROM aircrafts
demo-# ORDER BY range DESC;
 aircraft_code |        model        | range
---------------+---------------------+-------
 773           | Boeing 777-300      | 11100
 763           | Boeing 767-300      |  7900
 319           | Airbus A319-100     |  6700
 320           | Airbus A320-200     |  5700
 321           | Airbus A321-200     |  5600
 733           | Boeing 737-300      |  4200
 SU9           | Sukhoi SuperJet-100 |  3500
 CR2           | Bombardier CRJ-200  |  2700
 CN1           | Cessna 208 Caravan  |  1200
(9 rows)
```

</details>
<details>
<summary>Задание 3.3 (UPDATE)</summary>
3. Команда UPDATE позволяет в процессе обновления выполнять арифметические действия над значениями, находящимися в строках таблицы. Представим себе, что двигатели самолета Sukhoi SuperJet стали в два раза экономичнее, вследствие чего дальность полета этого лайнера возросла ровно в два раза. Команда UPDATE позволяет увеличить значение атрибута range в строке, хранящей информацию об этом самолете, даже не выполняя предварительно выборку с целью выяснения текущего значения этого атрибута. При присваивании нового значения атрибуту range можно справа от знака «=» написать не только числовую константу, но и целое выражение. В нашем случае оно будет простым: range = range * 2. Самостоятельно напишите команду UPDATE полностью, при этом не забудьте, что увеличить дальность полета нужно только у одной модели — Sukhoi SuperJet, поэтому необходимо использовать условие WHERE. Затем с помощью команды SELECT проверьте полученный результат.

**Решение 3.3:**

```sql
demo=# UPDATE aircrafts
demo-# SET range = range * 2
demo-# WHERE model = 'Sukhoi SuperJet-100';
UPDATE 1

demo=# SELECT * FROM aircrafts
ORDER BY range DESC;

 aircraft_code |        model        | range
---------------+---------------------+-------
 773           | Boeing 777-300      | 11100
 763           | Boeing 767-300      |  7900
 SU9           | Sukhoi SuperJet-100 |  7000
 319           | Airbus A319-100     |  6700
 320           | Airbus A320-200     |  5700
 321           | Airbus A321-200     |  5600
 733           | Boeing 737-300      |  4200
 CR2           | Bombardier CRJ-200  |  2700
 CN1           | Cessna 208 Caravan  |  1200
(9 rows)
```

</details>
<details>
<summary>Задание 3.4 (DELETE)</summary>
4. Если в предложении WHERE команды DELETE вы укажете логически и синтаксически корректное условие, но строк, удовлетворяющих этому условию, в таблице не окажется, то в ответ СУБД выведет сообщение
DELETE 0
Такая ситуация не является ошибкой или сбоем в работе СУБД. Например, если после удаления какой-то строки вы повторно попытаетесь удалить ее же, то получите именно такое сообщение. Самостоятельно смоделируйте описанную ситуацию, подобрав условие, которому гарантированно не соответствует ни одна строка в таблице «Самолеты» (aircrafts).

**Решение 3.4:**

```sql
demo=# DELETE FROM aircrafts
demo-# WHERE range <=0;
DELETE 0
```

</details>

## Глава 4. Типы данных СУБД PostgreSQL
<details>
<summary>Задание 4.1 (округление)</summary>
1. Создайте таблицу, содержащую атрибут типа numeric(precision, scale). Пусть это будет таблица, содержащая результаты каких-то измерений. Команда может быть, например, такой:

```sql
CREATE TABLE test_numeric
( measurement numeric(5, 2),
description text
);
```

Попробуйте с помощью команды INSERT продемонстрировать округление вводимого числа до той точности, которая задана при создании таблицы.
Подумайте, какая из следующих команд вызовет ошибку и почему? Проверьте свои предположения, выполнив эти команды.

```sql
INSERT INTO test_numeric
VALUES ( 999.9999, 'Какое-то измерение ' );
INSERT INTO test_numeric
VALUES ( 999.9009, 'Еще одно измерение' );
INSERT INTO test_numeric
VALUES ( 999.1111, 'И еще измерение' );
INSERT INTO test_numeric
VALUES ( 998.9999, 'И еще одно' );
```

Продемонстрируйте генерирование ошибки при попытке ввода числа, количество цифр в котором слева от десятичной точки (запятой) превышает допустимое.

**Решение 4.1:**

```sql
demo=# CREATE TABLE test_numeric
( measurement numeric(5, 2),
description text
);
CREATE TABLE

demo=# -- Округление вводимого числа до заданной в таблице точности
demo=# INSERT INTO test_numeric
demo-# VALUES ( 333.9999, 'Округленное измерение' );
INSERT 0 1

demo=# SELECT * FROM test_numeric;
 measurement |      description
-------------+-----------------------
      334.00 | Округленное измерение
(1 row)

demo=# --Запись числа 999.9999 вызовет ошибку, т.к. при округлении оно окажется слишком большим
demo=# -- для заданной точности и мощности

demo=# INSERT INTO test_numeric
VALUES ( 999.9999, 'Какое-то измерение' );
ERROR:  numeric field overflow
DETAIL:  A field with precision 5, scale 2 must round to an absolute value less than 10^3.

demo=# -- Остальные числа не должны привести к ошибкам. Проверяем.
demo=# INSERT INTO test_numeric
VALUES ( 999.9009, 'Еще одно измерение' );
INSERT 0 1
demo=# INSERT INTO test_numeric
VALUES ( 999.1111, 'И еще измерение' );
INSERT 0 1
demo=# INSERT INTO test_numeric
VALUES ( 998.9999, 'И еще одно' );
INSERT 0 1

demo=# SELECT * FROM test_numeric;
 measurement |      description
-------------+-----------------------
      334.00 | Округленное измерение
      999.90 | Еще одно измерение
      999.11 | И еще измерение
      999.00 | И еще одно
(4 rows)
```

</details>
<details>
<summary>Задание 4.2 (точность)</summary>
2. Предположим, что возникла необходимость хранить в одном столбце таблицы данные, представленные с различной точностью. Это могут быть, например, результаты физических измерений разнородных показателей или различные медицинские показатели здоровья пациентов (результаты анализов). В таком случае можно использовать тип numeric без указания масштаба и точности. Команда для создания таблицы может быть, например, такой:

```sql
CREATE TABLE test_numeric
( measurement numeric,
description text
);
```
Если у вас в базе данных уже есть таблица с таким же именем, то можно предварительно ее удалить с помощью команды

```sql
DROP TABLE test_numeric;
```

Вставьте в таблицу несколько строк:

```sql
INSERT INTO test_numeric
VALUES ( 1234567890.0987654321,
'Точность 20 знаков, масштаб 10 знаков' );
INSERT INTO test_numeric
VALUES ( 1.5,
'Точность 2 знака, масштаб 1 знак' );
INSERT INTO test_numeric
VALUES ( 0.12345678901234567890,
'Точность 21 знак, масштаб 20 знаков' );
INSERT INTO test_numeric
VALUES ( 1234567890,
'Точность 10 знаков, масштаб 0 знаков (целое число)' );
```

Теперь сделайте выборку из таблицы и посмотрите, что все эти разнообразные значения сохранены именно в том виде, как вы их вводили.

**Решение 4.2:**

```sql
demo=# DROP TABLE test_numeric;
DROP TABLE

demo=# CREATE TABLE test_numeric
demo-# (measurement numeric,
demo(# description text
demo(# );
CREATE TABLE

demo=# INSERT INTO test_numeric
VALUES ( 1234567890.0987654321, 'Точность 20 знаков, масштаб 10 знаков' ),
( 1.5, 'Точность 2 знака, масштаб 1 знак' ),
( 0.12345678901234567890, 'Точность 21 знак, масштаб 20 знаков' ),
( 1234567890, 'Точность 10 знаков, масштаб 0 знаков (целое число)' );
INSERT 0 4

demo=# SELECT * FROM test_numeric;

      measurement       |                    description
------------------------+----------------------------------------------------
  1234567890.0987654321 | Точность 20 знаков, масштаб 10 знаков
                    1.5 | Точность 2 знака, масштаб 1 знак
 0.12345678901234567890 | Точность 21 знак, масштаб 20 знаков
             1234567890 | Точность 10 знаков, масштаб 0 знаков (целое число)
(4 rows)
```

</details>
<details>
<summary>Задание 4.3 (numeric)</summary>
3. Тип данных numeric поддерживает специальное значение NaN, которое означает «не число» (not a number). В документации утверждается, что значение NaN считается равным другому значению NaN, а также что значение NaN считается большим любого другого «нормального» значения, т. е. не-NaN. Проверьте эти утверждения с помощью SQL-команды SELECT. В качестве примера приведем команду:

```sql
SELECT 'NaN'::numeric > 10000;
?column?
----------
t
(1 строка)
```

**Решение 4.3:**

```sql
demo-# SELECT 'NaN'::numeric > 10000;
 ?column?
----------
 t
(1 row)
```

</details>
<details>
<summary>Задание 4.4 (real и double precision)</summary>
4. При работе с числами типов real и double precision нужно помнить, что сравнение двух чисел с плавающей точкой на предмет равенства их значений может привести к неожиданным результатам. Например, сравним два очень маленьких числа (они представлены в экспоненциальной форме записи):

```sql
SELECT '5e-324'::double precision > '4e-324'::double precision;
?column?
----------
f
(1 строка)
```
Чтобы понять, почему так получается, выполните еще два запроса.

```sql
SELECT '5e-324'::double precision;
float8
-----------------------
4.94065645841247e-324
(1 строка)
SELECT '4e-324'::double precision;
float8
-----------------------
4.94065645841247e-324
(1 строка)
```

Самостоятельно проведите аналогичные эксперименты с очень большими числами, находящимися на границе допустимого диапазона для чисел типов real и double precision.

**Решение 4.4:**

```sql
demo=# SELECT '5e-324'::double precision > '4e-324'::double precision;
 ?column?
----------
 f
(1 row)

demo=# SELECT '5e-324'::double precision;
 float8
--------
 5e-324
(1 row)

demo=# SELECT '4e-324'::double precision;
 float8
--------
 5e-324
(1 row)

demo=# -- Эксперименты с очень большими числами, находящимися на границе 
demo=# -- допустимого диапазонадля чисел типов real и double precision.

demo=# -- В рассматриваемом диапазоне очень малентких чисел,
demo=# -- приведенных к real и double precision, после округления
demo=# -- несколько чисел становятся одинаковыми.
demo=# -- При записи в double precision чисел 2e-324 и менее
demo=# -- появляется ошибка, т.к. они становятся слишком маленькими.

demo=# CREATE TABLE precision
( precision6 text,
precision15 text);
CREATE TABLE
demo=# INSERT INTO precision(precision6, precision15)
VALUES ('9e-45', '9e-324'),
('8e-45', '8e-324'),
('7e-45', '7e-324'),
('6e-45', '6e-324'),
('5e-45', '5e-324'),
('4e-45', '4e-324'),
('3e-45', '3e-324'),
('2e-45', '2e-324'),
('1e-45', '1e-324');
INSERT 0 9

demo=# SELECT precision6, to_char(precision6::real, '9D99999EEEE') FROM precision;

 precision6 |   to_char
------------+--------------
 9e-45      |  8.40779e-45
 8e-45      |  8.40779e-45
 7e-45      |  7.00649e-45
 6e-45      |  5.60519e-45
 5e-45      |  5.60519e-45
 4e-45      |  4.20390e-45
 3e-45      |  2.80260e-45
 2e-45      |  1.40130e-45
 1e-45      |  1.40130e-45
(9 rows)

demo=# SELECT precision15, to_char(precision15::double precision, '9D99999999999999EEEE')
FROM precision;
ERROR:  "2e-324" is out of range for type double precision
demo=# DELETE FROM precision;
DELETE 9

demo=# INSERT INTO precision(precision6, precision15)
VALUES ('9e-45', '9e-324'),
('8e-45', '8e-324'),
('7e-45', '7e-324'),
('6e-45', '6e-324'),
('5e-45', '5e-324'),
('4e-45', '4e-324'),
('3e-45', '3e-324'),
('2e-45', '2e-323'),
('1e-45', '1e-323');
INSERT 0 9

demo=# SELECT precision15, to_char(precision15::double precision, '9D99999999999999EEEE') FROM precision;

 precision15 |        to_char
-------------+------------------------
 9e-324      |  9.88131291682493e-324
 8e-324      |  9.88131291682493e-324
 7e-324      |  4.94065645841247e-324
 6e-324      |  4.94065645841247e-324
 5e-324      |  4.94065645841247e-324
 4e-324      |  4.94065645841247e-324
 3e-324      |  4.94065645841247e-324
 2e-323      |  1.97626258336499e-323
 1e-323      |  9.88131291682493e-324
(9 rows)
```

</details>
<details>
<summary>Задание 4.5 (Infinity)</summary>
5. Типы данных real и double precision поддерживают специальные значения Infinity (бесконечность) и −Infinity (отрицательная бесконечность). Проверьте с помощью SQL-команды SELECT ожидаемые свойства этих значений. Например, сравните Infinity с наибольшим значением, которое допускается для типа double precision (можно использовать сокращенное написание Inf):

```sql
SELECT 'Inf'::double precision > 1E+308;
?column?
----------
t
(1 строка)
```

Выполните аналогичный запрос для наименьшего возможного значения типа double precision.

**Решение 4.5:**

```sql
demo-# SELECT 'Inf'::double precision > 1E+308;

 ?column?
----------
 t
(1 row)

demo=# SELECT '-Inf'::double precision < 1E-307;

 ?column?
----------
 t
(1 row)
```

</details>
<details>
<summary>Задание 4.6 (NaN)</summary>
6. Типы данных real и double precision поддерживают специальное значение NaN, которое означает «не число» (not a number). В математике существует такое понятие, как неопределенность. В качестве одного из ее вариантов служит результат операции умножения нуля на бесконечность. Посмотрите, что выдаст в результате PostgreSQL:
  
```sql
SELECT 0.0 * 'Inf'::real;

?column?
----------
NaN
(1 строка)
```

В документации утверждается, что значение NaN считается равным другому значению NaN, а также что значение NaN считается большим любого другого «нормального» значения, т. е. не-NaN. Проверьте эти утверждения с помощью
SQL-команды SELECT. Например, сравните значения NaN и Infinity.

**Решение 4.6:**

```sql
select 'NaN'::real > 'Inf'::real;

?column?
----------
t
(1 строка)
```

</details>
<details>
<summary>Задание 4.11 (точность)</summary>
11. Типы timestamp, time и interval позволяют задать точность ввода и вывода значений. Точность предписывает количество десятичных цифр в поле секунд. Проиллюстрируем эту возможность на примере типа time, выполнив три запроса: в первом запросе вообще не используем параметр точности, во втором назначим его равным 0, в третьем запросе сделаем его равным 3.
  
```sql
SELECT current_time;

timetz
--------------------
19:46:14.584641+03
(1 строка)

SELECT current_time::time( 0 );

time
----------
19:39:45
(1 строка)

SELECT current_time::time( 3 );

time
--------------
19:39:54.085
(1 строка)
```

Выполните подобные команды для типов timestamp и interval. Тип date такой возможности — задавать точность — не имеет. Как вы думаете, почему?

**Решение 4.11:**

```sql
demo=# SELECT current_time;

    current_time
--------------------
 21:31:46.401496+04
(1 row)

demo=# SELECT current_time::time( 0 );

 current_time
--------------
 21:33:04
(1 row)

demo=# SELECT current_time::time( 3 );
 current_time
--------------
 21:33:26.314
(1 row)

demo=# SELECT current_timestamp::timestamp;
     current_timestamp
----------------------------
 2023-04-14 22:04:28.762675
(1 row)

demo=# SELECT current_timestamp::timestamp(0);

  current_timestamp
---------------------
 2023-04-14 22:04:44
(1 row)

demo=# SELECT current_timestamp::timestamp(3);

    current_timestamp
-------------------------
 2023-04-14 22:04:57.524
(1 row)

demo=#  SELECT 'P2023-04-14T22:04:57.524333'::interval;

                 interval
-------------------------------------------
 2023 years 4 mons 14 days 22:04:57.524333
(1 row)

demo=# SELECT ('yesterday'::timestamp - 'today'::timestamp)::interval;

 interval
----------
 -1 days
(1 row)

demo=# SELECT ('2023-04-14 22:04:57.524333'::timestamp - current_timestamp)::interval(3);

   interval
---------------
 -01:02:39.223
(1 row)

demo=# -- В типе данных date не задается точность, т.к.
demo=# -- он задается в днях целым числом без долей.
```

</details>
<details>
<summary>Задание 4.15 (функции форматирования)</summary>
15. В документации в разделе 9.8 «Функции форматирования данных» представлены описания множества полезных функций, позволяющих преобразовать в строку данные других типов, например, timestamp. Одна из таких функций — to_char.
  
Приведем несколько команд, иллюстрирующих использование этой функции.Ее первым параметром является форматируемое значение, а вторым — шаблон,описывающий формат, в котором это значение будет представлено при вводе или выводе. Сначала попробуйте разобраться, не обращаясь к документации, в том, что означает второй параметр этой функции в каждой из приведенных команд, а затем проверьте свои предположения по документации.

```sql
SELECT to_char( current_timestamp, 'mi:ss' );

to_char
---------
47:43
(1 строка)

SELECT to_char( current_timestamp, 'dd' );

to_char
---------
12
(1 строка)

SELECT to_char( current_timestamp, 'yyyy-mm-dd' );

to_char
------------
2017-03-12
(1 строка)
```
Поэкспериментируйте с этой функцией, извлекая из значения типа timestamp различные поля и располагая их в нужном вам порядке.

**Решение 4.15:**

```sql
demo=# SELECT to_char(current_date, 'Сегодня: DAY, YYYY');

         to_char
--------------------------
 Сегодня: SATURDAY , 2023
(1 row)

```

</details>
<details>
<summary>Задание 4.16 (даты и время)</summary>
16. При выполнении приведения типа данных производится проверка значения на допустимость. Попробуйте ввести недопустимое значение даты, например, 29 февраля в невисокосном году.

```sql
SELECT 'Feb 29, 2015'::date;
```

Получите сообщение об ошибке.

**Решение 4.16:**

```sql
demo-# SELECT 'Feb 29, 2015'::date;
ERROR:  date/time field value out of range: "Feb 29, 2015"
LINE 2: SELECT 'Feb 29, 2015'::date;
               ^
```

</details>
<details>
<summary>Задание 4.17 (даты и время)</summary>
17. При выполнении приведения типа данных производится проверка значения на допустимость. Попробуйте ввести недопустимое значение времени, например, с нарушением формата.

```sql
SELECT '21:15:16:22'::time;

ОШИБКА: неверный синтаксис для типа time: "21:15:16:22"
СТРОКА 1: select '21:15:16:22'::time;
```

**Решение 4.17:**

```sql
demo=# SELECT '21:15:16:22'::time;
ERROR:  invalid input syntax for type time: "21:15:16:22"
LINE 2: SELECT '21:15:16:22'::time;
               ^
demo=# SELECT '21:15 16:22'::time;
ERROR:  invalid input syntax for type time: "21:15 16:22"
LINE 1: SELECT '21:15 16:22'::time;
               ^
demo=# -- Правильный формат ждя time отображается как 'HH:MM:SS' без часового пояса.
demo=# SELECT '21:16:22'::time;

   time
----------
 21:16:22
(1 row)
```

</details>
<details>
<summary>Задание 4.19 (даты и время)</summary>
19. С типами даты и времени можно выполнять различные арифметические операции. Как правило, их применение является интуитивно понятным. Выполните следующую команду и проанализируйте результат.
  
```sql
SELECT ( '20:34:35'::time - '19:44:45'::time );
```
А теперь попробуйте предположить, какой результат будет получен, если в этой
команде знак «минус» заменить на знак «плюс»? Проверьте ваши предположения с помощью утилиты psql. Подробное описание всех допустимых арифметических операций с датами и временем приведено в документации в разделе 9.9
«Операторы и функции даты/времени».

**Решение 4.19:**

```sql
demo-# SELECT ( '20:34:35'::time - '19:44:45'::time );
 ?column?
----------
 00:49:50
(1 row)

demo=# -- Результат выводится как разница в приведенном ранее формате.
demo=# -- Если выполнить сложение, то должна появиться ошибка. т.к. более 24 часов быть не может.

demo=# SELECT ( '20:34:35'::time + '19:44:45'::time );
ERROR:  operator is not unique: time without time zone + time without time zone
LINE 5: SELECT ( '20:34:35'::time + '19:44:45'::time );
                                  ^
HINT:  Could not choose a best candidate operator. You might need to add explicit type casts.
```

</details>
<details>
<summary>Задание 4.20 (даты и время)</summary>
20. Значение типа interval можно получить при вычитании одной временной отметки из другой, например:
  
```sql
SELECT ( current_timestamp - '2016-01-01'::timestamp )
AS new_date;

new_date
-------------------------
278 days 00:10:33.33236
(1 строка)
```

А что получится, если прибавить интервал к временной отметке? Сначала попробуйте дать ответ, не прибегая к помощи утилиты psql, а затем проверьте свой ответ с помощью этой утилиты. Например, прибавим интервал длительностью в 1 месяц к текущей к временной отметке:

```sql
SELECT ( current_timestamp + '1 mon'::interval ) AS new_date;
```

В этой команде с помощью ключевого слова AS мы назначили псевдоним для того столбца, который будет выведен в результате. Выполните эту же команду, убрав псевдоним, и найдите отличия.

**Решение 4.20:**

```sql
demo=# SELECT ( current_timestamp - '2016-01-01'::timestamp ) AS new_date;

         new_date
---------------------------
 2764 days 21:25:43.189133
(1 row)

demo=# -- Если прибавить интервал к временной отметке, то она передвинеися на величину интервала. 

demo=# SELECT ( current_timestamp + '1 mon'::interval ) AS new_date;

           new_date
-------------------------------
 2023-08-27 21:05:35.704142+04
(1 row)

demo=# SELECT ( current_timestamp + '1 mon'::interval );

           ?column?
------------------------------
 2023-08-27 21:05:43.26417+04
(1 row)

demo=# -- В данном случае в запросе не использовались никакое из существующих представлений
demo=# -- и в результатах запроса колонка в таблице получила наименование по-умолчанию.
demo=# -- Испрльзование псевдонима позволило отобразить желаемое название колонки.
```

</details>
<details>
<summary>Задание 4.32 (массивы)</summary>
32. Изучая приемы работы с массивами, можно, как и в других случаях, пользоваться способностью команды SELECT обходиться без создания таблиц. Покажем лишь два примера.
  
Для объединения (конкатенации) массивов служит функция array_cat

```sql
SELECT array_cat( ARRAY[ 1, 2, 3 ], ARRAY[ 3, 5 ] );

array_cat
-------------
{1,2,3,3,5}
(1 строка)
```

Удалить из массива элементы, имеющие указанное значение, можно таким образом:

```sql
SELECT array_remove( ARRAY[ 1, 2, 3 ], 3 );

array_remove
--------------
{1,2}
(1 строка)
```
Для работы с массивами предусмотрено много различных функций и операторов, представленных в разделе документации 9.18 «Функции и операторы для работы с массивами». Самостоятельно ознакомьтесь с ними, используя описанную технологию работы с командой SELECT.

**Решение 4.32:**

```sql
demo=# SELECT array_cat( ARRAY[ 1, 2, 3 ], ARRAY[ 3, 5 ] );

  array_cat
-------------
 {1,2,3,3,5}
(1 row)

demo=# SELECT array_remove( ARRAY[ 1, 2, 3 ], 3 );

 array_remove
--------------
 {1,2}
(1 row)

demo=# SELECT array_fill(55, ARRAY[2,3]);

       array_fill
-------------------------
 {{55,55,55},{55,55,55}}
(1 row)

demo=# SELECT array_fill(55, ARRAY[4,3]);

                  array_fill
-----------------------------------------------
 {{55,55,55},{55,55,55},{55,55,55},{55,55,55}}
(1 row)

demo=# SELECT array_fill(55, ARRAY[4], ARRAY[3]);

     array_fill
---------------------
 [3:6]={55,55,55,55}
(1 row)
```

</details>
<details>
<summary>Задание 4.35 (JSON)</summary>
35. Изучая приемы работы с типами JSON, можно, как и в случае с массивами, пользоваться способностью команды SELECT обходиться без создания таблиц.
  
Покажем лишь один пример. Добавить новый ключ и соответствующее ему значения в уже существующий объект можно оператором ||:

```sql
SELECT '{ "sports": "хоккей" }'::jsonb || '{ "trips": 5 }'::jsonb;

?column?
----------------------------------
{"trips": 5, "sports": "хоккей"}
(1 строка)
```
Для работы с типами JSON предусмотрено много различных функций и операторов, представленных в разделе документации 9.15 «Функции и операторы JSON». Самостоятельно ознакомьтесь с ними, используя описанную технологию работы с командой SELECT.

**Решение 4.35:**

```sql
demo=#  SELECT
('{ "sports": "хоккей" }'::jsonb || '{ "trips": 5 }'::jsonb || '{"experience":7}'::jsonb)  -> 'trips' AS trips, -- Извлекаем поле по ключу
jsonb_object ('{ "sports", "хоккей", "trips", 5, "experience", 7}') AS jsb_array, -- Формируем объект JSON из текстового массива
jsonb_path_query_array ('["sports", "хоккей", "trips", 5, "experience", 7]', '$[*] ? (@ == "sports")')  AS jsb_chk; -- Проверка равенства

 trips |                       jsb_array                       |  jsb_chk
-------+-------------------------------------------------------+------------
 5     | {"trips": "5", "sports": "хоккей", "experience": "7"} | ["sports"]
(1 row)

demo=# SELECT * FROM json_each ('{ "sports" : "хоккей", "trips" : 5, "experience" : 7}'); -- Разворачиваем JSON-объект верхнего уровня в набор пар ключ/значение (key/value)

    key     |  value
------------+----------
 sports     | "хоккей"
 trips      | 5
 experience | 7
(3 rows)

```

</details>

## Глава 5. Основы языка определения данных
<details>
<summary>Задание 5.1 (DEFAULT, current_user)</summary>
1. При использовании значений по умолчанию с ключевым словом DEFAULT возможны и ситуации, когда типичным будет не конкретное значение данных, а способ его получения. Например, если мы захотим фиксировать в каждой строке таблицы «Студенты» имя пользователя базы данных, добавившего эту строку в таблицу, тогда необходимо в определение таблицы добавить еще один столбец. Этот столбец по умолчанию будет получать значение, возвращаемое функцией current_user.

```sql
CREATE TABLE students
( record_book numeric( 5 ) NOT NULL,
name text NOT NULL,
doc_ser numeric( 4 ),
doc_num numeric( 6 ),
who_adds_row text DEFAULT current_user, -- добавленный столбец
PRIMARY KEY ( record_book )
);
```

Эта функция — current_user — будет вызываться не при создании таблицы, а при вставке каждой строки. При этом в команде INSERT не требуется указывать значение для столбца who_adds_row, поскольку функция current_user будет вызываться самой СУБД PostgreSQL:

```sql
INSERT INTO students ( record_book, name, doc_ser, doc_num )
VALUES ( 12300, 'Иванов Иван Иванович', 0402, 543281 );
```

Давайте пойдем дальше и пожелаем фиксировать не только имя пользователя базы данных, добавившего строку в таблицу, но также и момент времени, когда это было сделано. Самостоятельно внесите модификацию в определение таблицы students для решения этой задачи, а затем выполните команду INSERT для проверки полученного решения.

Если до выполнения этого упражнения вы еще не ознакомились с командой ALTER TABLE, то вместо модифицирования определения таблицы сначала удалите ее, а затем создайте заново:

```sql
DROP TABLE students;
CREATE TABLE students ...
```

**Решение 5.1:**

```sql
edu=# CREATE TABLE students
( record_book numeric( 5 ) NOT NULL,
name text NOT NULL,
doc_ser numeric( 4 ),
doc_num numeric( 6 ),
who_adds_row text DEFAULT current_user, -- добавленный столбец
PRIMARY KEY ( record_book )
);
CREATE TABLE

edu=# INSERT INTO students ( record_book, name, doc_ser, doc_num )
VALUES ( 12300, 'Иванов Иван Иванович', 0402, 543281 );
INSERT 0 1

edu=# SELECT * FROM students;

 record_book |         name         | doc_ser | doc_num | who_adds_row
-------------+----------------------+---------+---------+--------------
       12300 | Иванов Иван Иванович |     402 |  543281 | postgres
(1 row)

edu=# ALTER TABLE students
ADD add_time timestamp DEFAULT current_timestamp;
ALTER TABLE

edu=# INSERT INTO students ( record_book, name, doc_ser, doc_num )
VALUES ( 12301, 'Петров Петр Петрович', 0402, 543282 );
INSERT 0 1

edu=# SELECT * FROM students;
 record_book |         name         | doc_ser | doc_num | who_adds_row |         add_time
-------------+----------------------+---------+---------+--------------+---------------------------
       12300 | Иванов Иван Иванович |     402 |  543281 | postgres     | 2023-05-20 23:25:40.533526
       12301 | Петров Петр Петрович |     402 |  543282 | postgres     | 2023-05-20 23:27:09.616822
(2 rows)
```

</details>
<details>
<summary>Задание 5.2 (CHECK)</summary>
2. Посмотрите, какие ограничения уже наложены на атрибуты таблицы «Успеваемость» (progress). Воспользуйтесь командой \d утилиты psql. А теперь предложите для этой таблицы ограничение уровня таблицы.

В качестве примера рассмотрим такой вариант. Добавьте в таблицу progress еще один атрибут — «Форма проверки знаний» (test_form), который можетпринимать только два значения: «экзамен» или «зачет». Тогда набор допустимых значений атрибута «Оценка» (mark) будет зависеть от того, экзамен или зачет предусмотрены по данной дисциплине. Если предусмотрен экзамен, тогдадопускаются значения 3, 4, 5, если зачет — тогда 0 (не зачтено) или 1 (зачтено).

Не забудьте, что значения NULL для атрибутов test_form и mark не допускаются. Новое ограничение может быть таким:

```sql
ALTER TABLE progress
ADD CHECK (
( test_form = 'экзамен' AND mark IN ( 3, 4, 5 ) )
OR
( test_form = 'зачет' AND mark IN ( 0, 1 ) )
);
```

Проверьте, как будет работать новое ограничение в модифицированной таблице progress. Для этого выполните команды INSERT, как удовлетворяющиеограничению, так и нарушающие его.

В таблице уже было ограничение на допустимые значения атрибута mark. Как вы думаете, не будет ли оно конфликтовать с новым ограничением? Проверьте эту гипотезу. Если ограничения конфликтуют, тогда удалите старое ограничение и снова попробуйте добавить строки в таблицу.

Подумайте, какое еще ограничение уровня таблицы можно предложить для этой таблицы?

**Решение 5.2:**

```sql
edu=# \d progress
                   Table "public.progress"
   Column    |     Type     | Collation | Nullable | Default
-------------+--------------+-----------+----------+---------
 record_book | numeric(5,0) |           | not null |
 subject     | text         |           | not null |
 acad_year   | text         |           | not null |
 term        | numeric(1,0) |           | not null |
 mark        | numeric(1,0) |           | not null | 5
Check constraints:
    "progress_mark_check" CHECK (mark >= 3::numeric AND mark <= 5::numeric)
    "progress_term_check" CHECK (term = 1::numeric OR term = 2::numeric)
Foreign-key constraints:
    "progress_record_book_fkey" FOREIGN KEY (record_book) REFERENCES students(record_book) ON UPDATE CASCADE ON DELETE CASCADE

edu=# ALTER TABLE progress
ADD test_form text NOT NULL; -- Форма проверки знаний
ALTER TABLE

edu=# ALTER TABLE progress
ADD CHECK (
( test_form = 'экзамен' AND mark IN ( 3, 4, 5 ) )
OR
( test_form = 'зачет' AND mark IN ( 0, 1 ) )
);
ALTER TABLE

edu=# -- Проверяем как работает ограничение
edu=# -- Вставляем запись, удовлетворяющую ограничению
edu=# INSERT INTO progress (record_book, subject, acad_year, term, mark, test_form)
VALUES (12300, 'Термодинамика', 2023, 1, 4, 'экзамен');
INSERT 0 1
edu=# SELECT * FROM progress;

 record_book |    subject    | acad_year | term | mark | test_form
-------------+---------------+-----------+------+------+-----------
       12300 | Термодинамика | 2023      |    1 |    4 | экзамен
(1 row)

edu=# -- Вставляем запись, нарушающую ограничение
edu=# INSERT INTO progress (record_book, subject, acad_year, term, mark, test_form)
VALUES (12301, 'Термодинамика', 2023, 1, 1, 'экзамен');
ERROR:  new row for relation "progress" violates check constraint "progress_check"
DETAIL:  Failing row contains (12301, Термодинамика, 2023, 1, 1, экзамен).

edu=# -- Проверяем не конфликтует ли старое и новые ограничения
edu=#  INSERT INTO progress (record_book, subject, acad_year, term, mark, test_form)
VALUES (12301, 'Термодинамика', 2023, 1, 1, 'зачет');
ERROR:  new row for relation "progress" violates check constraint "progress_mark_check"
DETAIL:  Failing row contains (12301, Термодинамика, 2023, 1, 1, зачет).

edu=# -- Удаляем ограничение, созданное вместе с таблицей
edu=# ALTER TABLE progress
DROP CONSTRAINT progress_mark_check;
ALTER TABLE

edu=# -- Снова добавляем строку в таблицу
edu=#  INSERT INTO progress (record_book, subject, acad_year, term, mark, test_form)
VALUES (12301, 'Термодинамика', 2023, 1, 1, 'зачет');
INSERT 0 1

edu=# -- Можно установить еще одно ограничение на заполнение таблицы конкретным академгодом для избежания опечаток
ALTER TABLE progress
ADD CONSTRAINT  current_acad_year_only CHECK (acad_year = '2022/2023');
ALTER TABLE
```

</details>
<details>
<summary>Задание 5.4 (DEFAULT)</summary>
4. В определении таблицы «Успеваемость» (progress) для атрибута mark не только задано ограничение CHECK, но и установлено значение по умолчанию с помощью ключевого слова DEFAULT:

```sql
...
mark numeric( 1 ) NOT NULL
CHECK ( mark >= 3 AND mark <= 5 )
DEFAULT 5,
...
```

Как вы думаете, что будет, если в ограничении DEFAULT мы «случайно» допустим ошибку, написав DEFAULT 6? Если в команде INSERT не указать значение для атрибута mark, то на каком этапе эта ошибка будет выявлена: уже на этапе
создания таблицы или только при вставке строки в нее?

Вот эта команда может быть вам полезной для проверки гипотезы, поскольку в ней отсутствует передаваемое значение для атрибута mark:

```sql
INSERT INTO progress ( record_book, subject, acad_year, term )
VALUES ( 12300, 'Физика', '2016/2017',1 );
```

**Решение 5.4:**

```sql
edu=# -- Устанавливаем значение по-умолчанию =6 для атрибута mark
edu=# ALTER TABLE progress
ALTER COLUMN mark SET DEFAULT 6;
ALTER TABLE

edu=# -- Проверяем работу ограничения DEFAULT 6
edu=# INSERT INTO progress ( record_book, subject, acad_year, term )
VALUES ( 12303, 'Физика', '2016/2017',1 );
ERROR:  new row for relation "progress" violates check constraint "progress_mark_check"
DETAIL:  Failing row contains (12303, Физика, 2016/2017, 1, 6).
edu=# -- Ограничения проверяются на момент вставки строк в таблицу
```

</details>
<details>
<summary>Задание 5.5 (constraint и NULL)</summary>
5. В стандарте SQL сказано, что при наличии ограничения уникальности, включающего один или более столбцов, все же возможны повторяющиеся значения этих столбцов в разных строках, но лишь в том случае, если это значения NULL.
PostgreSQL придерживается такого же подхода.
  
Модифицируйте определение таблицы «Студенты» (students), добавив ограничение уникальности по двум столбцам: doc_ser и doc_num. А затем проверьте вышеприведенное утверждение, добавив в таблицу не только строки, содержащие конкретные значения этих двух столбцов, но также и по две строки, имеющие следующие свойства:

– одинаковые значения столбца doc_ser и NULL-значения столбца doc_num;
– NULL-значения столбца doc_num и столбца doc_ser.

Подобные вещи возможны, так как NULL-значения не считаются совпадающими. Это можно проверить с помощью команды

```sql
SELECT (null = null);
```

**Решение 5.5:**

```sql
edu=# ALTER TABLE students
ADD CONSTRAINT doc_unique UNIQUE (doc_ser, doc_num);
ALTER TABLE

edu=# INSERT INTO students
(record_book, name, doc_ser, doc_num)
VALUES
(12300, 'Иванов Иван Иванович', 0402, 543281),
(12301, 'Петров Иван Иванович', 0403, 543282),
(12302, 'Сидоров Иван Иванович', 0404, NULL),
(12303, 'Иванова Мария Иванович', 0404, NULL),
(12304, 'Петрова Клавдия Иванович', NULL, NULL),
(12305, 'Сидорова Ольга  Иванович', NULL, NULL);
INSERT 0 6

edu=# SELECT * FROM students;
 record_book |           name           | doc_ser | doc_num | who_adds_row |    add_time
-------------+--------------------------+---------+---------+--------------+----------------
       12300 | Иванов Иван Иванович     |     402 |  543281 | postgres     | 00:22:17.54066
       12301 | Петров Иван Иванович     |     403 |  543282 | postgres     | 00:22:17.54066
       12302 | Сидоров Иван Иванович    |     404 |         | postgres     | 00:22:17.54066
       12303 | Иванова Мария Иванович   |     404 |         | postgres     | 00:22:17.54066
       12304 | Петрова Клавдия Иванович |         |         | postgres     | 00:22:17.54066
       12305 | Сидорова Ольга  Иванович |         |         | postgres     | 00:22:17.54066
(6 rows)
```

</details>
<details>
<summary>Задание 5.6 (PRIMARY KEY, FOREIGN KEY)</summary>
6. Модифицируйте определения таблиц «Студенты» (students) и «Успеваемость»
(progress). В таблице students в качестве первичного ключа назначьте комбинацию атрибутов doc_ser и doc_num, а в таблице progress соответствующим образом измените определение внешнего ключа.

```sql
CREATE TABLE students
( record_book numeric( 5 ) NOT NULL UNIQUE,
name text NOT NULL,
doc_ser numeric( 4 ),
doc_num numeric( 6 ),
PRIMARY KEY ( doc_ser, doc_num )
);
```

Обратите внимание, что для атрибутов doc_ser и doc_num можно не указывать ограничение NOT NULL: они входят в состав первичного ключа, а в нем NULLзначения не допускаются, поэтому ограничение NOT NULL фактически подразумевается при включении атрибута в состав первичного ключа.

```sql
CREATE TABLE progress
( doc_ser numeric( 4 ),
doc_num numeric( 6 ),
subject text NOT NULL,
acad_year text NOT NULL,
term numeric( 1 ) NOT NULL CHECK ( term = 1 OR term = 2 ),
mark numeric( 1 ) NOT NULL CHECK ( mark >= 3 AND mark <= 5 )
DEFAULT 5,
FOREIGN KEY ( doc_ser, doc_num )
REFERENCES students ( doc_ser, doc_num )
ON DELETE CASCADE
ON UPDATE CASCADE
);
```

Теперь и первичный, и внешний ключи — составные. Проверьте их действие, добавив несколько строк в каждую таблицу.

**Решение 5.6:**

```sql
edu=# DROP TABLE students, progress;
DROP TABLE

edu=# CREATE TABLE students
( record_book numeric( 5 ) NOT NULL UNIQUE,
name text NOT NULL,
doc_ser numeric( 4 ),
doc_num numeric( 6 ),
PRIMARY KEY ( doc_ser, doc_num )
);
CREATE TABLE

edu=# CREATE TABLE progress
( doc_ser numeric( 4 ),
doc_num numeric( 6 ),
subject text NOT NULL,
acad_year text NOT NULL,
term numeric( 1 ) NOT NULL CHECK ( term = 1 OR term = 2 ),
mark numeric( 1 ) NOT NULL CHECK ( mark >= 3 AND mark <= 5 )
DEFAULT 5,
FOREIGN KEY ( doc_ser, doc_num )
REFERENCES students ( doc_ser, doc_num )
ON DELETE CASCADE
ON UPDATE CASCADE
);
CREATE TABLE

edu=#
edu=# INSERT INTO students
(record_book, name, doc_ser, doc_num)
VALUES
(12300, 'Иванов Иван Иванович', 0402, 543281),
(12301, 'Петров Иван Иванович', 0402, 543282);
INSERT 0 2

edu=# INSERT INTO progress
(doc_ser, doc_num, subject, acad_year, term, mark)
VALUES
(0402, 543281, 'Термодинамика', '2022/2023', 2, 4),
(0402, 543282, 'Термодинамика', '2022/2023', 2, 5);
INSERT 0 2
edu=# SELECT * FROM students;

 record_book |         name         | doc_ser | doc_num
-------------+----------------------+---------+---------
       12300 | Иванов Иван Иванович |     402 |  543281
       12301 | Петров Иван Иванович |     402 |  543282
(2 rows)

edu=# SELECT * FROM progress;

 doc_ser | doc_num |    subject    | acad_year | term | mark
---------+---------+---------------+-----------+------+------
     402 |  543281 | Термодинамика | 2022/2023 |    2 |    4
     402 |  543282 | Термодинамика | 2022/2023 |    2 |    5
(2 rows)
```

</details>
<details>
<summary>Задание 5.10 (constraint и изменение тапа данных)</summary>
В таблице «Студенты» (students) атрибут «Серия документа, удостоверяющего личность» (doc_ser) имеет числовой тип, однако в сериях таких документов могут встречаться лидирующие нули, которые в числовых столбцах не сохраняются. Например, при записи значения серии «0402» первый ноль не сохранится.
  
Модифицируйте таблицу students, заменив числовой тип данных на символьный, например, character. Как вы думаете, эта операция пройдет без затруднений или они все же возможны? Проверьте ваши предположения, выполнив
модификацию.

**Решение 5.10:**

```sql
ALTER TABLE students
ALTER COLUMN doc_ser TYPE character (4);
ERROR:  foreign key constraint "progress_doc_ser_doc_num_fkey" cannot be implemented
DETAIL:  Key columns "doc_ser" and "doc_ser" are of incompatible types: numeric and character.

edu=# ALTER TABLE progress
DROP CONSTRAINT progress_doc_ser_doc_num_fkey;
ALTER TABLE

edu=# ALTER TABLE students
DROP CONSTRAINT students_pkey;
ALTER TABLE

edu=# ALTER TABLE students
ALTER COLUMN doc_ser TYPE character (4);
ALTER TABLE

edu=# ALTER TABLE progress
ALTER COLUMN doc_ser TYPE character (4);
ALTER TABLE

edu=#  ALTER TABLE students
ADD CONSTRAINT students_pkey PRIMARY KEY (doc_ser, doc_num);
ALTER TABLE

edu=# ALTER TABLE progress
ADD CONSTRAINT students_fkey FOREIGN KEY (doc_ser, doc_num)
REFERENCES students (doc_ser, doc_num);
ALTER TABLE
```

</details>
<details>
<summary>Задание 5.13 (dependent objects)</summary>
И представление «Рейсы» (flights_v), и материализованное представление «Маршруты» (routes) построены на основе таблиц «Рейсы» (flights) и «Аэропорты» (airports). Логично предположить, что при каскадном удалении, например, таблицы «Аэропорты», представление «Рейсы» будет также удалено, поскольку при удалении базовой таблицы этому представлению просто неоткуда будет брать данные.
  
А что вы можете предположить насчет материализованного представления «Маршруты»: будет ли оно также удалено или нет? Ведь оно уже содержит данные, в отличие от обычного представления. Так ли, условно говоря, сильна его
связь с таблицами, на основе которых оно сконструировано?
  
Проведите необходимые эксперименты, начав с команды

```sql
DROP TABLE airports;
```

Если вам потребуется восстановить все объекты базы данных, то вы всегда сможете воспользоваться файлом demo_small.sql и просто повторить процедуру развертывания учебной базы данных, которая описана в главе 2. Поэтому смело экспериментируйте с таблицами и представлениями.

**Решение 5.13:**

```sql
demo=# DROP TABLE airports;
ERROR:  cannot drop table airports because other objects depend on it
DETAIL:  view flights_v depends on table airports
materialized view routes depends on table airports
constraint flights_arrival_airport_fkey on table flights depends on table airports
constraint flights_departure_airport_fkey on table flights depends on table airports
HINT:  Use DROP ... CASCADE to drop the dependent objects too.

demo=# DROP MATERIALIZED VIEW routes;
DROP MATERIALIZED VIEW
demo=# -- Материализованные представления можно удалять независимо от связей с таблицами
```

</details>
<details>
<summary>Задание 5.16 (REFRESH MATERIALIZED VIEW)</summary>
16. Как вы думаете, при изменении данных в таблицах, на основе которых сконструировано материализованное представление, содержимое этого представления тоже синхронно изменяется или нет?
  
Если содержимое материализованного представления изменяется синхронно с базовыми таблицами, то продемонстрируйте это. Если же оно остается неизменным, то покажите, как его синхронизировать с базовыми таблицами.

**Решение 5.16:**

```sql
demo=# -- В отличие от представления, материализованное представление
demo=# -- хранит "снимок" результатов запроса на момент своего созданния.
demo=# -- Чтобы получить свежие данные при изменении исходных таблиц нужно обновить м. представление.
demo=# REFRESH MATERIALIZED VIEW routes;
REFRESH MATERIALIZED VIEW
```

</details>
