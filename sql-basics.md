# Решения задач на PostgreSQL из учебного пособия [Моргунов, Е. П. PostgreSQL. Основы языка SQL: учеб. пособие / Е. П. Моргунов; под ред. Е. В. Рогова, П. В. Лузанова. — СПб.: БХВ-Петербург, 2018. — 336 с.: ил](https://postgrespro.ru/education/books/sqlprimer)  

Задания выполнены в [демонстрационной базе данных "Авиаперевозки".](https://postgrespro.ru/education/demodb)

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

## Глава 6. Запросы
<details>
<summary>Задание 6.2 (LIKE)</summary>
2. Этот запрос выбирает из таблицы «Билеты» (tickets) всех пассажиров с именами, состоящими из трех букв (в шаблоне присутствуют три символа «_»):

```sql
SELECT passenger_name
FROM tickets
WHERE passenger_name LIKE '___ %';
```
Предложите шаблон поиска в операторе LIKE для выбора из этой таблицы всех пассажиров с фамилиями, состоящими из пяти букв.

**Решение 6.2:**

```sql
demo=#  SELECT passenger_name
FROM tickets
WHERE passenger_name LIKE '% _____' LIMIT 10;

 passenger_name
----------------
 EVGENIY BELOV
 ALEKSEY ISAEV
 MATVEY POPOV
 ARTEM POPOV
 ILYA POPOV
 VLADIMIR POPOV
 PAVEL GUSEV
 LEONID ORLOV
 EVGENIY GUSEV
 NIKOLAY FOMIN
(10 rows)
```

</details>
<details>
<summary>Задание 6.3 (SIMILAR TO)</summary>
3. В разделе документации 9.7.2 «Регулярные выражения SIMILAR TO» рассматривается оператор SIMILAR TO. Он работает аналогично оператору LIKE, но использует шаблоны, соответствующие определению регулярных выражений,
приведенному в стандарте SQL. Регулярные выражения SQL представляют собой комбинацию синтаксиса LIKE с синтаксисом обычных регулярных выражений. Самостоятельно ознакомьтесь с оператором SIMILAR TO.

**Решение 6.3:**

```sql
demo=# -- Найдем пассажиров с фамилией, начинающейся на буквы A,B,V
demo=# SELECT passenger_name
FROM tickets
WHERE passenger_name SIMILAR TO '% (A|B|V)%' LIMIT 10;

   passenger_name
---------------------
 NINA BELOVA
 TATYANA ANDREEVA
 EVGENIY BELOV
 MARINA ALEKSANDROVA
 VITALIY VOLKOV
 IRINA VOROBEVA
 VALENTINA BARANOVA
 ILMIRA VOLKOVA
 YURIY VASILEV
 EKATERINA BORISOVA
(10 rows)
```

</details>
<details>
<summary>Задание 6.5 (COALESCE, NULLIF)</summary>
5. В разделе документации 9.17 «Условные выражения» представлены условные выражения, которые поддерживаются в PostgreSQL. В тексте главы была рассмотрена конструкция CASE. Самостоятельно ознакомьтесь с функциями COALESCE, NULLIF, GREATEST и LEAST.

**Решение 6.5:**

```sql
demo=# --COALESCE
demo=# -- Список пассажиров с email и без него
demo=# SELECT passenger_name, COALESCE(contact_data -> 'email') AS email
FROM tickets
LIMIT 10;

  passenger_name  |                  email
------------------+------------------------------------------
 VLADIMIR FROLOV  |
 NINA BELOVA      | "belovanina11041976@postgrespro.ru"
 KIRA SIDOROVA    | "sidorova.kira_101971@postgrespro.ru"
 GENNADIY NIKITIN | "nikitin_g_111965@postgrespro.ru"
 FEDOR SHEVCHENKO | "shevchenkofedor-1963@postgrespro.ru"
 DAMIR TIMOFEEV   |
 EVGENIY MAKAROV  | "makarov-e011968@postgrespro.ru"
 ALFIYA FROLOVA   | "alfiya-frolova1963@postgrespro.ru"
 NADEZHDA KUZMINA | "kuzmina-nadezhda.121975@postgrespro.ru"
 TATYANA ANDREEVA |
(10 rows)

demo=# -- NULLIF
demo=# -- Кол-во рейсов, которые не были отменены
demo=# SELECT count(NULLIF(status, 'Cancelled')) FROM flights;

 count
-------
 65235
(1 row)

demo=# -- GREATEST и LEAST применены в Задании 5.7
```

</details>
<details>
<summary>Задание 6.7 (GREATEST, LEAST)</summary>
Самые крупные самолеты в нашей авиакомпании — это Boeing 777-300. Выяснить, между какими парами городов они летают, поможет запрос:

```sql
SELECT DISTINCT departure_city, arrival_city
FROM routes r
JOIN aircrafts a ON r.aircraft_code = a.aircraft_code
WHERE a.model = 'Boeing 777-300'
ORDER BY 1;

departure_city | arrival_city
----------------+--------------
Екатеринбург | Москва
Москва | Екатеринбург
Москва | Новосибирск
Москва | Пермь
Москва | Сочи
Новосибирск | Москва
Пермь | Москва
Сочи | Москва
(8 строк)
```

К сожалению, в этой выборке информация дублируется. Пары городов приведены по два раза: для рейса «туда» и для рейса «обратно». Модифицируйте запрос таким образом, чтобы каждая пара городов была выведена только один раз:

```sql
departure_city | arrival_city
---------------+--------------
 Москва        | Екатеринбург
 Новосибирск   | Москва
 Пермь         | Москва
 Сочи          | Москва
(4 rows)
```

**Решение 6.1:**

```sql
demo=# SELECT DISTINCT  GREATEST(departure_city, arrival_city) , LEAST(departure_city, arrival_city)
FROM routes r
JOIN aircrafts a ON r.aircraft_code = a.aircraft_code
WHERE a.model = 'Boeing 777-300'
ORDER BY 1;

  greatest   |    least
-------------+--------------
 Москва      | Екатеринбург
 Новосибирск | Москва
 Пермь       | Москва
 Сочи        | Москва
(4 rows)
```

</details>
<details>
<summary>Задание 6.9 (GROUP BY)</summary>
9. Для ответа на вопрос, сколько рейсов выполняется из Москвы в Санкт-Петербург, можно написать совсем простой запрос:

```sql
SELECT count( * )
FROM routes
WHERE departure_city = 'Москва'
AND arrival_city = 'Санкт-Петербург';

count
-------
12
(1 строка)
```

А с помощью какого запроса можно получить результат в таком виде?

```sql
 departure_city |  arrival_city   | count
----------------+-----------------+-------
 Москва         | Санкт-Петербург |    12
(1 row)
```

**Решение 6.9:**

```sql
SELECT departure_city, arrival_city, count( * )
FROM routes
WHERE departure_city = 'Москва'
AND arrival_city = 'Санкт-Петербург'
GROUP BY 1, 2;

 departure_city |  arrival_city   | count
----------------+-----------------+-------
 Москва         | Санкт-Петербург |    12
(1 row)
```

</details>
<details>
<summary>Задание 6.11 (array_length, ORDER BY)</summary>
11. В материализованном представлении «Маршруты» (routes) имеется столбец days_of_week, который содержит списки (массивы) номеров дней недели, когда выполняется каждый рейс. Для оптимизации расписания вылетов из Москвы нужно выявить пять городов, в которые из столицы отправляется наибольшее число ежедневных рейсов (маршрутов). Строки в выборке следует расположить в убывающем порядке числа выполняемых рейсов. Указание: воспользуйтесь функцией array_length.

**Решение 6.11:**

```sql
demo=# SELECT departure_city, arrival_city, count(*)
FROM routes
WHERE departure_city = 'Москва'
AND array_length( days_of_week, 1 ) = 7
GROUP BY 1, 2
ORDER BY count DESC
LIMIT 5;

 departure_city |  arrival_city   | count
----------------+-----------------+-------
 Москва         | Санкт-Петербург |    12
 Москва         | Брянск          |     9
 Москва         | Ульяновск       |     5
 Москва         | Петрозаводск    |     4
 Москва         | Йошкар-Ола      |     4
(5 rows)
```

</details>
<details>
<summary>Задание 6.12 (unnest, WITH ORDINALITY)</summary>
12. Предположим, что служба материального снабжения нашей авиакомпании запросила информацию о числе рейсов, выполняющихся из Москвы в каждый день недели.Результат можно получить путем выполнения семи аналогичных запросов: по
одному для каждого дня недели. Начнем с понедельника:

```sql
SELECT 'Понедельник' AS day_of_week, count( * ) AS num_flights
FROM routes
WHERE departure_city = 'Москва'
AND days_of_week @> '{ 1 }'::integer[];
```

В этом запросе используется оператор @>, который проверяет, содержатся ли все элементы массива, стоящего справа от него, в том массиве, который находится слева. В правом массиве всего один элемент — номер интересующего нас дня недели.

```sql
day_of_week | num_flights
-------------+-------------
Понедельник | 131
(1 строка)
```

Запрос для вторника отличается лишь номером дня недели в массиве. Запрос для вторника отличается лишь номером дня недели в массиве. Нужно выполнить еще пять аналогичных команд, чтобы получить результаты для всех дней недели. Очевидно, что это нерациональный способ. Получить требуемый результат можно с помощью одного запроса:

```sql
SELECT unnest( days_of_week ) AS day_of_week,
count( * ) AS num_flights
FROM routes
WHERE departure_city = 'Москва'
GROUP BY day_of_week
ORDER BY day_of_week;

day_of_week | num_flights
-------------+-------------
1 | 131
2 | 134
3 | 126
4 | 136
5 | 124
6 | 133
7 | 124
(7 строк)
```

Задание 1. Самостоятельно разберитесь, как работает приведенный запрос. Выясните, что делает функция unnest. Для того чтобы найти ее описание, можно воспользоваться теми разделами документации, которые были указаны в главе 4. Однако можно воспользоваться и предметным указателем (Index), ссылка на который находится в самом низу оглавления документации. В качестве вспомогательного запроса, проясняющего работу функции unnest, можно выполнить следующий:

```sql
SELECT flight_no, unnest( days_of_week ) AS day_of_week
FROM routes
WHERE departure_city = 'Москва'
ORDER BY flight_no;
```

Задание 2. Использование номеров дней недели в предыдущей выборке не должно вызывать затруднений. Но все-таки предположим, что нас попросили модифицировать запрос, чтобы результат выводился в таком виде: 

```sql
name_of_day | num_flights
-------------+-------------
Пн. | 131
Вт. | 134
Ср. | 126
Чт. | 136
Пт. | 124
Сб. | 133
Вс. | 124
(7 строк)
```

Покажем одно из возможных решений задачи. Оно основано на использовании специальной табличной функции unnest в предложении FROM. Подробно об этом написано в документации в разделе 7.2.1.4 «Табличные функции». Функция может принимать любое число параметров-массивов, а возвращает набор строк, которые могут использоваться в запросах как обычные таблицы. В этих наборах строк столбцы формируются из значений, содержащихся в массивах.

```sql
SELECT dw.name_of_day, count( * ) AS num_flights
FROM (
SELECT unnest( days_of_week ) AS num_of_day
FROM routes
WHERE departure_city = 'Москва'
) AS r,
unnest( '{ 1, 2, 3, 4, 5, 6, 7 }'::integer[],
'{ "Пн.", "Вт.", "Ср.", "Чт.", "Пт.", "Сб.", "Вс."}'::text[]
) AS dw( num_of_day, name_of_day )
WHERE r.num_of_day = dw.num_of_day
GROUP BY r.num_of_day, dw.name_of_day
ORDER BY r.num_of_day;
```

Этот запрос можно упростить. Предложение WITH ORDINALITY позволяет в нашем примере избавиться от массива целых чисел, обозначающих дни недели, поскольку автоматически формируется столбец целых чисел, нумерующих строки результирующего набора. По умолчанию этот столбец называется ordinality. Это имя можно использовать в запросе. Самостоятельно модифицируйте запрос с применением предложения WITH ORDINALITY.

**Решение 6.12:**

```sql
demo=# SELECT dw.name_of_day, count( * ) AS num_flights
FROM (
SELECT unnest( days_of_week ) AS num_of_day
FROM routes
WHERE departure_city = 'Москва'
) AS r,
unnest('{ "Пн.", "Вт.", "Ср.", "Чт.", "Пт.", "Сб.", "Вс."}'::text[]) WITH ORDINALITY
AS dw(name_of_day, num_of_day )
WHERE r.num_of_day = dw.num_of_day
GROUP BY r.num_of_day, dw.name_of_day
ORDER BY r.num_of_day;

 name_of_day | num_flights
-------------+-------------
 Пн.         |         131
 Вт.         |         134
 Ср.         |         127
 Чт.         |         135
 Пт.         |         124
 Сб.         |         133
 Вс.         |         124
(7 rows)
```

</details>
<details>
<summary>Задание 6.14 (left, strpos, substr)</summary>
14. Предположим, что маркетологи нашей авиакомпании хотят знать, как часто встречаются различные имена среди пассажиров? Получить распределение частот имен пассажиров в таблице «Билеты» (tickets) поможет такой запрос:

```sql
SELECT left( passenger_name, strpos( passenger_name, ' ' ) - 1 )
AS firstname, count( * )
FROM tickets
GROUP BY 1
ORDER BY 2 DESC;

firstname | count
-----------+-------
ALEKSANDR | 20328
SERGEY | 15133
VLADIMIR | 12806
TATYANA | 12058
ELENA | 11291
OLGA | 9998
...
MAGOMED | 14
ASKAR | 13
RASUL | 11
(363 строки)
```

Напишите запрос для ответа на аналогичный вопрос насчет распределения частот фамилий пассажиров. Подробные сведения о других функциях для работы со строковыми данными приведены в документации в разделе 9.4 «Строковые функции и операторы».

**Решение 6.14:**

```sql
SELECT substr(passenger_name, strpos( passenger_name, ' ' )+1)
AS firstname, count( * )
FROM tickets
GROUP BY 1
ORDER BY 2 DESC
LIMIT 10;

 firstname | count
-----------+-------
 IVANOV    | 17081
 KUZNECOV  | 15480
 IVANOVA   | 14610
 KUZNECOVA | 13275
 POPOV     |  9693
 PETROV    |  8595
 POPOVA    |  8243
 MAKAROV   |  7687
 VASILEV   |  7626
 PETROVA   |  7383
(10 rows)
```

</details>
<details>
<summary>Задание 6.19 (WITH RECURSIVE, UNION)</summary>
29. В разделе 6.4 мы использовали рекурсивный алгоритм в общем табличном выражении. Изучите этот пример, чтобы лучше понять работу рекурсивного алгоритма:

```sql
WITH RECURSIVE ranges ( min_sum, max_sum )
AS (
VALUES( 0, 100000 ),
( 100000, 200000 ),
( 200000, 300000 )
UNION ALL
SELECT min_sum + 100000, max_sum + 100000
FROM ranges
WHERE max_sum < ( SELECT max( total_amount ) FROM bookings )
)
SELECT * FROM ranges;

min_sum | max_sum
---------+---------
0      | 100000 исходные строки
100000 | 200000
200000 | 300000
100000 | 200000 результат первой итерации
200000 | 300000
300000 | 400000
200000 | 300000 результат второй итерации
300000 | 400000
400000 | 500000
300000 | 400000
400000 | 500000
500000 | 600000
...
1000000 | 1100000 результат (n-3)-й итерации
1100000 | 1200000
1200000 | 1300000
1100000 | 1200000 результат (n-2)-й итерации
1200000 | 1300000
1200000 | 1300000 результат (n-1)-й итерации (предпоследней)
(36 строк)
```

Здесь мы с помощью предложения VALUES специально создали виртуальную таблицу из трех строк, хотя для получения требуемого результата достаточно только одной строки (0, 100000). Еще важно то, что предложение UNION ALL не
удаляет строки-дубликаты, поэтому мы можем видеть весь рекурсивный процесс порождения новых строк.

При рекурсивном выполнении запроса

```sql
SELECT min_sum + 100000, max_sum + 100000
FROM ranges
WHERE max_sum < ( SELECT max( total_amount ) FROM bookings )
```

каждый раз выполняется проверка в условии WHERE. И на (n−2)-й итерации это условие отсеивает одну строку, т. к. после (n − 3)-й итерации значение атрибута max_sum в третьей строке было равно 1 300 000.

Ведь запрос выдаст значение

```sql
SELECT max( total_amount ) FROM bookings;

max
------------
1204500.00
(1 строка)
```

Таким образом, после (n − 2)-й итерации во временной области остается всего две строки, после (n−1)-й итерации во временной области остается только одна строка.

Заключительная итерация уже не добавляет строк в результирующую таблицу,поскольку единственная строка, поданная на вход команде SELECT, будет отклонена условием WHERE. Работа алгоритма завершается. 

Задание 1. Модифицируйте запрос, добавив в него столбец level (можно назвать его и iteration). Этот столбец должен содержать номер текущей итерации, поэтому нужно увеличивать его значение на единицу на каждом шаге. Незабудьте задать начальное значение для добавленного столбца в предложении VALUES.

Задание 2. Для завершения экспериментов замените UNION ALL на UNION ивыполните запрос. Сравните этот результат с предыдущим, когда мы использовали UNION ALL.

**Решение 6.19:**

```sql
demo=# WITH RECURSIVE ranges ( min_sum, max_sum, iteration ) AS
( VALUES( 0, 100000, 1 )
UNION ALL
SELECT min_sum + 100000, max_sum + 100000, iteration + 1
FROM ranges
WHERE max_sum <
( SELECT max( total_amount ) FROM bookings )
)
SELECT r.min_sum, r.max_sum, count( b.* ), r.iteration
FROM bookings b
RIGHT OUTER JOIN ranges r
ON b.total_amount >= r.min_sum
AND b.total_amount < r.max_sum
GROUP BY r.min_sum, r.max_sum, r.iteration
ORDER BY r.min_sum;

  min_sum | max_sum | count  | iteration
---------+---------+--------+-----------
       0 |  100000 | 447043 |         1
  100000 |  200000 | 106632 |         2
  200000 |  300000 |  27009 |         3
  300000 |  400000 |   7485 |         4
  400000 |  500000 |   3025 |         5
  500000 |  600000 |   1482 |         6
  600000 |  700000 |    541 |         7
  700000 |  800000 |    124 |         8
  800000 |  900000 |     56 |         9
  900000 | 1000000 |     26 |        10
 1000000 | 1100000 |      8 |        11
 1100000 | 1200000 |      1 |        12
 1200000 | 1300000 |      1 |        13
(13 rows)

demo=# -- Меняем UNION ALL на UNION
demo=# -- Ничего не изменилось, т.к. в рекурсивном алгоритме
demo=# -- участвует только одна строка и строк-дубликатов не возникает,
demo=# -- и нет целосообразности в использовании UNION

demo=# WITH RECURSIVE ranges ( min_sum, max_sum, iteration ) AS
( VALUES( 0, 100000, 1 )
UNION
SELECT min_sum + 100000, max_sum + 100000, iteration + 1
FROM ranges
WHERE max_sum <
( SELECT max( total_amount ) FROM bookings )
)
SELECT r.min_sum, r.max_sum, count( b.* ), r.iteration
FROM bookings b
RIGHT OUTER JOIN ranges r
ON b.total_amount >= r.min_sum
AND b.total_amount < r.max_sum
GROUP BY r.min_sum, r.max_sum, r.iteration
ORDER BY r.min_sum;

 min_sum | max_sum | count  | iteration
---------+---------+--------+-----------
       0 |  100000 | 447043 |         1
  100000 |  200000 | 106632 |         2
  200000 |  300000 |  27009 |         3
  300000 |  400000 |   7485 |         4
  400000 |  500000 |   3025 |         5
  500000 |  600000 |   1482 |         6
  600000 |  700000 |    541 |         7
  700000 |  800000 |    124 |         8
  800000 |  900000 |     56 |         9
  900000 | 1000000 |     26 |        10
 1000000 | 1100000 |      8 |        11
 1100000 | 1200000 |      1 |        12
 1200000 | 1300000 |      1 |        13
(13 rows)
```

</details>
<details>
<summary>Задание 6.21 (UNION, INTERSECT, EXCEPT)</summary>
21. В тексте главы был приведен запрос, выводящий список городов, в которые нет рейсов из Москвы.

```sql
SELECT DISTINCT a.city
FROM airports a
WHERE NOT EXISTS (
SELECT * FROM routes r
WHERE r.departure_city = 'Москва'
AND r.arrival_city = a.city
)
AND a.city <> 'Москва'
ORDER BY city;
```

Можно предложить другой вариант, в котором используется одна из операций над множествами строк: объединение, пересечение или разность. Вместо знака «?» поставьте в приведенном ниже запросе нужное ключевое слово — UNION, INTERSECT или EXCEPT — и обоснуйте ваше решение.

```sql
SELECT city
FROM airports
WHERE city <> 'Москва'
?
SELECT arrival_city
FROM routes
WHERE departure_city = 'Москва'
ORDER BY city;
```

**Решение 6.21:**

```sql
demo=# -- Сначала строим множество городов, исключающее Москву
demo=# -- Затем объединяем его с множеством городов прибытия за минусом Москвы
demo=# -- Дубликатов не будет, т.к. нет рейсов внутри одного города
demo=# SELECT city
FROM airports
WHERE city <> 'Москва'
EXCEPT
SELECT arrival_city
FROM routes
WHERE departure_city = 'Москва'
ORDER BY city;

         city
----------------------
 Благовещенск
 Иваново
 Иркутск
 Калуга
 Когалым
 Комсомольск-на-Амуре
 Кызыл
 Магадан
 Нижнекамск
 Новокузнецк
 Стрежевой
 Сургут
 Удачный
 Усть-Илимск
 Усть-Кут
 Ухта
 Череповец
 Чита
 Якутск
 Ярославль
(20 rows)
```

</details>
<details>
<summary>Задание 6.23 (CTE - общее табличное выражение)</summary>
Предположим, что департамент развития нашей авиакомпании задался вопросом: каким будет общее число различных маршрутов, которые теоретически можно проложить между всеми городами?
  
Если в каком-то городе имеется более одного аэропорта, то это учитывать не будем, т. е. маршрутом будем считать путь между городами, а не между аэропортами. Здесь мы используем соединение таблицы с самой собой на основе неравенства значений атрибутов.

```sql
SELECT count( * )
FROM ( SELECT DISTINCT city FROM airports ) AS a1
JOIN ( SELECT DISTINCT city FROM airports ) AS a2
ON a1.city <> a2.city;

count
-------
10100
(1 строка)
```

Задание. Перепишите этот запрос с общим табличным выражением.

**Решение 6.23:**

```sql
demo=# WITH cts AS
(SELECT DISTINCT city FROM airports)
SELECT count( * )
FROM  cts AS a1
JOIN cts  AS a2
ON  a1.city <> a2.city;

 count
-------
 10100
(1 row)
```

</details>
<details>
<summary>Задание 6.24 (IN, ANY)</summary>
В тексте главы мы рассмотрели использование подзапросов в предикатах EXISTS и IN. Существуют также предикаты многократного сравнения ANY и ALL. Они представлены в документации в разделе 9.22 «Выражения подзапросов». Самостоятельно ознакомьтесь с этими предикатами и напишите несколько запросов с их применением.
  
Предикаты ANY и ALL имеют некоторую связь с предикатом IN. В частности, использование IN эквивалентно использованию конструкции = ANY, а использование NOT IN эквивалентно использованию конструкции <> ALL. Пример двух эквивалентных запросов, выбирающих аэропорты в часовых поясах Asia/Novokuznetsk и Asia/Krasnoyarsk:

```sql
SELECT * FROM airports
WHERE timezone IN ( 'Asia/Novokuznetsk', 'Asia/Krasnoyarsk' );
SELECT * FROM airports
WHERE timezone = ANY (
VALUES ( 'Asia/Novokuznetsk' ), ( 'Asia/Krasnoyarsk' )
);
```

Еще один пример. В тексте главы мы рассматривали запрос, подсчитывающий
количество маршрутов, проложенных из самых восточных аэропортов.

```sql
SELECT departure_city, count( * )
FROM routes
GROUP BY departure_city
HAVING departure_city IN (
SELECT city
FROM airports
WHERE longitude > 150
)
ORDER BY count DESC;
```

В этом запросе можно заменить IN на ANY таким образом:

```sql
HAVING departure_city = ANY ( ... )
```

**Решение 6.24:**

```sql
demo=# -- Расчет кол-ва городов для полетов в зоне Europe/Moscow
demo=# SELECT count(city)
FROM airports
WHERE timezone = ANY (VALUES ( 'Europe/Moscow'));

 count
-------
    44
(1 row)

demo=# -- Расчет кол-ва городов для полетов вне зоеы Europe/Moscow
demo=# SELECT count(city)
FROM airports
WHERE timezone != ALL (VALUES ( 'Europe/Moscow'));

 count
-------
    60
(1 row)
```

</details>
<details>
<summary>Задание 6.26 (CTE - общее табличное выражение)</summary>
Предположим, что некая контролирующая организация потребовала информацию о размещении пассажиров одного из рейсов Кемерово — Москва в салоне самолета. Для определенности выберем конкретный рейс из тех рейсов, которые уже прибыли на момент времени, соответствующий текущему моменту. Текущий момент времени в базе данных «Авиаперевозки» определяется с помощью функции bookings.now.
Выполним запрос:

```sql
SELECT *
FROM flights_v
WHERE departure_city = 'Кемерово'
AND arrival_city = 'Москва'
AND actual_arrival < bookings.now();
```

Выберем для дальнейшей работы рейс, у которого значения атрибутов flight_id — 27584, aircraft_code — SU9. Получим список пассажиров этого рейса с местами, которые им были назначены в салоне самолета.

```sql
SELECT t.passenger_name, b.seat_no
FROM (
ticket_flights tf
JOIN tickets t ON tf.ticket_no = t.ticket_no
)
JOIN boarding_passes b
ON tf.ticket_no = b.ticket_no
AND tf.flight_id = b.flight_id
WHERE tf.flight_id = 27584
ORDER BY t.passenger_name;

passenger_name | seat_no
---------------------+---------
ALEKSANDR ABRAMOV | 1A
ALEKSANDR GRIGOREV | 5C
ALEKSANDR SERGEEV | 6F
ALEKSEY FEDOROV | 11D
ALEKSEY MELNIKOV | 18A
...
VLADIMIR POPOV | 11A
YAROSLAV KUZMIN | 18F
YURIY ZAKHAROV | 10F
(44 строки)
```

Отсортируем строки по фамилиям пассажиров:

```sql
SELECT t.passenger_name,
substr( t.passenger_name,
strpos( t.passenger_name, ' ' ) + 1
) AS lastname,
left( t.passenger_name,
strpos( t.passenger_name, ' ' ) - 1
) AS firstname,
b.seat_no
FROM (
ticket_flights tf
JOIN tickets t ON tf.ticket_no = t.ticket_no
)
JOIN boarding_passes b
ON tf.ticket_no = b.ticket_no
AND tf.flight_id = b.flight_id
WHERE tf.flight_id = 27584
ORDER BY 2, 3;

passenger_name | lastname | firstname | seat_no
---------------------+-----------+-----------+---------
ALEKSANDR ABRAMOV | ABRAMOV | ALEKSANDR | 1A
NIKITA ANDREEV | ANDREEV | NIKITA | 6D
ANTONINA ANISIMOVA | ANISIMOVA | ANTONINA | 11F
...
YURIY ZAKHAROV | ZAKHAROV | YURIY | 10F
ELENA ZOTOVA | ZOTOVA | ELENA | 20E
(44 строки)
```

Получим список мест в салоне самолета и пассажиров, которые сидели на этих местах. При этом незанятые места также должны быть выведены (поэтому используем левое внешнее соединение LEFT OUTER JOIN)

```sql
SELECT s.seat_no, p.passenger_name
FROM seats s
LEFT OUTER JOIN (
SELECT t.passenger_name, b.seat_no
FROM (
ticket_flights tf
JOIN tickets t ON tf.ticket_no = t.ticket_no
)
JOIN boarding_passes b
ON tf.ticket_no = b.ticket_no
AND tf.flight_id = b.flight_id
WHERE tf.flight_id = 27584
) AS p
ON s.seat_no = p.seat_no
WHERE s.aircraft_code = 'SU9'
ORDER BY s.seat_no;

seat_no | passenger_name
---------+---------------------
10A |
10C |
10D | NATALYA POPOVA
10E |
10F | YURIY ZAKHAROV
11A | VLADIMIR POPOV
11C | ANNA KUZMINA
...
8F |
9A | MAKSIM CHERNOV
9C |
9D | LYUDMILA IVANOVA
9E |
9F | SOFIYA KULIKOVA
(97 строк)
```

Предположим, что нас попросили отсортировать места в порядке их расположения в салоне самолета и вывести также адреса электронной почты пассажиров (у кого они были указаны при бронировании). Для выполнения второго требования воспользуемся столбцом contact_data. В нем содержатся JSON-объекты, содержащие контактные данные пассажиров. Ряд из них имеет ключ email. Модифицированный запрос будет таким:

```sql
SELECT s.seat_no, p.passenger_name, p.email
FROM seats s
LEFT OUTER JOIN (
SELECT t.passenger_name, b.seat_no,
t.contact_data->'email' AS email
FROM (
ticket_flights tf
JOIN tickets t ON tf.ticket_no = t.ticket_no
)
JOIN boarding_passes b
ON tf.ticket_no = b.ticket_no
AND tf.flight_id = b.flight_id
WHERE tf.flight_id = 27584
) AS p
ON s.seat_no = p.seat_no
WHERE s.aircraft_code = 'SU9'
ORDER BY
left( s.seat_no, length( s.seat_no ) - 1 )::integer,
right( s.seat_no, 1 );

seat_no | passenger_name | email
---------+-------------------+------------------------------------
1A | ALEKSANDR ABRAMOV |
1C | |
1D | DENIS PETROV |
1F | LEONID BARANOV | "baranov.l.1967@postgrespro.ru"
2A | |
2C | |
...
9F | SOFIYA KULIKOVA | "sofiya.kulikova_041963@postgre..."
10A | |
10C | |
10D | NATALYA POPOVA | "popova.n_13031976@postgrespro.ru"
...
20E | ELENA ZOTOVA |
20F | LILIYA OSIPOVA |
(97 строк)
```

Задание. Перепишите последний запрос с использованием общего табличного выражения и добавьте столбец «Класс обслуживания» (fare_conditions).

**Решение 6.26:**

```sql
demo=# -- В посадочных талонах моей версии БД нет рейса 27584
demo=# -- Вместо него взят рейс 27589	
WITH p AS
(
SELECT t.passenger_name, b.seat_no, t.contact_data->'email' AS email
FROM (
ticket_flights tf
JOIN tickets t ON tf.ticket_no = t.ticket_no
)
JOIN boarding_passes b
ON tf.ticket_no = b.ticket_no
AND tf.flight_id = b.flight_id
WHERE tf.flight_id = 27589
)
SELECT s.seat_no, s.fare_conditions, p.passenger_name, p.email
FROM seats s
LEFT OUTER JOIN p 
ON s.seat_no = p.seat_no
WHERE s.aircraft_code = 'SU9'
ORDER BY
left( s.seat_no, length( s.seat_no ) - 1 )::integer,
right( s.seat_no, 1 );

 seat_no | fare_conditions | passenger_name  |                email
---------+-----------------+-----------------+-------------------------------------
 1A      | Business        | IRINA ISAEVA    | "i_isaeva.1981@postgrespro.ru"
 1C      | Business        |                 |
 1D      | Business        |                 |
 1F      | Business        |                 |
 2A      | Business        | YULIYA CHERNOVA |
 2C      | Business        |                 |
 2D      | Business        |                 |
 2F      | Business        |                 |
 3A      | Business        | IVAN DENISOV    | "ivan-denisov1964@postgrespro.ru"
 3C      | Business        |                 |
 3D      | Business        |                 |
 3F      | Business        |                 |
 4A      | Economy         | IRINA MALYSHEVA |
 4C      | Economy         |                 |
 4D      | Economy         |                 |
 4E      | Economy         |                 |
 4F      | Economy         |                 |
:
```

</details>

## Глава 7. Изменение данных
<details>
<summary>Задание 7.1 (INSERT)</summary>
1. Добавьте в определение таблицы aircrafts_log значение по умолчанию current_timestamp и соответствующим образом измените команды INSERT, приведенные в тексте главы.

**Решение 7.1:**

```sql
demo=# -- Чтобы сработала вставка строки 'INSERT', в operation изменен порядок столбцов в aircrafts_log

demo=# ALTER TABLE aircrafts_log
ALTER COLUMN when_add SET DEFAULT current_timestamp;
ALTER TABLE

demo=# WITH add_row AS
( INSERT INTO aircrafts_tmp
SELECT * FROM aircrafts
RETURNING *
)
INSERT INTO aircrafts_log
SELECT add_row.aircraft_code, add_row.model, add_row.range, 'INSERT'
FROM add_row;
INSERT 0 9
demo=# SELECT * FROM aircrafts_log;

 aircraft_code |        model        | range | operation |          when_add
---------------+---------------------+-------+-----------+----------------------------
 773           | Boeing 777-300      | 11100 | INSERT    | 2023-05-23 11:24:52.804142
 763           | Boeing 767-300      |  7900 | INSERT    | 2023-05-23 11:24:52.804142
 SU9           | Sukhoi SuperJet-100 |  3000 | INSERT    | 2023-05-23 11:24:52.804142
 320           | Airbus A320-200     |  5700 | INSERT    | 2023-05-23 11:24:52.804142
 321           | Airbus A321-200     |  5600 | INSERT    | 2023-05-23 11:24:52.804142
 319           | Airbus A319-100     |  6700 | INSERT    | 2023-05-23 11:24:52.804142
 733           | Boeing 737-300      |  4200 | INSERT    | 2023-05-23 11:24:52.804142
 CN1           | Cessna 208 Caravan  |  1200 | INSERT    | 2023-05-23 11:24:52.804142
 CR2           | Bombardier CRJ-200  |  2700 | INSERT    | 2023-05-23 11:24:52.804142
(9 rows)
```

</details>
<details>
<summary>Задание 7.2 (RETURNING)</summary>
2. В предложении RETURNING можно указывать не только символ «∗», означающий выбор всех столбцов таблицы, но и более сложные выражения, сформированные на основе этих столбцов. В тексте главы мы копировали содержимое таблицы «Самолеты» в таблицу aircrafts_tmp, используя в предложении RETURNING именно «∗». Однако возможен и другой вариант запроса:

```sql
WITH add_row AS
( INSERT INTO aircrafts_tmp
SELECT * FROM aircrafts
RETURNING aircraft_code, model, range,
current_timestamp, 'INSERT'
)
INSERT INTO aircrafts_log
SELECT ? FROM add_row;
```

Что нужно написать в этом запросе вместо вопросительного знака?

**Решение 7.2:**

```sql
WITH add_row AS
( INSERT INTO aircrafts_tmp
SELECT * FROM aircrafts
RETURNING aircraft_code, model, range,
current_timestamp, 'INSERT'
)
INSERT INTO aircrafts_log
SELECT * FROM add_row;
```

</details>
<details>
<summary>Задание 7.3 (RETURNING)</summary>
3 Если бы мы для копирования данных в таблицу aircrafts_tmp использовали команду INSERT без общего табличного выражения

```sql
INSERT INTO aircrafts_tmp SELECT * FROM aircrafts;
```

то в качестве выходного результата мы увидели бы сообщение
INSERT 0 9

Как вы думаете, что будет выведено, если дополнить команду предложением RETURNING *?

```sql
INSERT INTO aircrafts_tmp SELECT * FROM aircrafts RETURNING *;
```

Проверьте ваши предположения на практике. Подумайте, каким образом можно использовать выведенный результат?

**Решение 7.3:**

```sql
demo=# -- Ничего, кроме "INSERT 0 9" выведено не будет.
demo=# -- Строки хранятся в памяти и доступны для обращения,
demo=# -- но на них никакие запросы не ссылаются.
demo=# -- Возвращенные через RETURNING * строки в данном случае можно использовать, например,
demo=# -- для расширения информативности результатов вставки в рамках одного запроса.
demo=# -- Например, сразу при вставке выведем количество самолетов Boeing

demo=# WITH a_tmp AS
(INSERT INTO aircrafts_tmp SELECT * FROM aircrafts RETURNING *
)
SELECT count(a_tmp.model) AS boeing
FROM a_tmp
WHERE model ~ '^Boeing';

 boeing
--------
      3
(1 row)
```

</details>
<details>
<summary>Задание 7.4 (INSERT, ON CONFLICT)</summary>

4. В тексте главы в предложениях ON CONFLICT команды INSERT мы использовали только выражения, состоящие из имени одного столбца. Однако в таблице «Места» (seats) первичный ключ является составным и включает два столбца.

Напишите команду INSERT для вставки новой строки в эту таблицу и предусмотрите возможный конфликт добавляемой строки со строкой, уже имеющейся в таблице. Сделайте два варианта предложения ON CONFLICT: первый — с использованием перечисления имен столбцов для проверки наличия дублирования, второй — с использованием предложения ON CONSTRAINT.

Для того чтобы не изменить содержимое таблицы «Места», создайте ее копию и выполняйте все эти эксперименты с таблицей-копией.

**Решение 7.4:**

```sql
demo=# CREATE TEMP TABLE seats_tmp
( LIKE seats INCLUDING CONSTRAINTS INCLUDING INDEXES );
CREATE TABLE

demo=# INSERT INTO seats_tmp
SELECT * FROM seats;
INSERT 0 1339
demo=# SELECT * FROM seats_tmp LIMIT 10;

 aircraft_code | seat_no | fare_conditions
---------------+---------+-----------------
 319           | 2A      | Business
 319           | 2C      | Business
 319           | 2D      | Business
 319           | 2F      | Business
 319           | 3A      | Business
 319           | 3C      | Business
 319           | 3D      | Business
 319           | 3F      | Business
 319           | 4A      | Business
 319           | 4C      | Business
(10 rows)

demo=# -- Вставка строки с игнорированием операции в случае конфликта
demo=# INSERT INTO seats_tmp
VALUES ( '319', '2A', 'Business' )
ON CONFLICT DO NOTHING
RETURNING *;

 aircraft_code | seat_no | fare_conditions
---------------+---------+-----------------
(0 rows)

INSERT 0 0

demo=# -- Вставка строки c перезаписью
demo=# INSERT INTO seats_tmp
VALUES ( '319', '2A', 'Business' )
ON CONFLICT ON CONSTRAINT seats_tmp_pkey
DO UPDATE SET aircraft_code = excluded.aircraft_code,
seat_no = excluded.seat_no
RETURNING *;

 aircraft_code | seat_no | fare_conditions
---------------+---------+-----------------
 319           | 2A      | Business
(1 row)

INSERT 0 1
```

</details>
<details>
<summary>Задание 7.5 (INSERT, DO UPDATE, WHERE)</summary>
В предложении DO UPDATE команды INSERT может использоваться и условие WHERE. Самостоятельно ознакомьтесь с этой возможностью с помощью документации и напишите такую команду INSERT.

**Решение 7.5:**

```sql
demo=# INSERT INTO seats_tmp
VALUES ( '319', '2A', 'Business' ),
('CN1', '2A', 'Economy')
ON CONFLICT ON CONSTRAINT seats_tmp_pkey
DO UPDATE SET aircraft_code = excluded.aircraft_code,
seat_no = excluded.seat_no
WHERE seats_tmp.fare_conditions = 'Business'
RETURNING *;

 aircraft_code | seat_no | fare_conditions
---------------+---------+-----------------
 319           | 2A      | Business
(1 row)

INSERT 0 1
```

</details>
<details>
<summary>Задание 7.8 (модификация запроса)</summary>
8. В тексте главы был приведен запрос, предназначенный для учета числа билетов, проданных по всем направлениям на текущую дату. Однако тот запрос был рассчитан на одновременное добавление только одной записи в таблицу «Перелеты» (ticket_flights_tmp). Ниже мы предложим более универсальный запрос, который предусматривает возможность единовременного ввода нескольких записей о перелетах, выполняемых на различных рейсах.

Для проверки работоспособности предлагаемого запроса выберем несколько рейсов по маршрутам: Красноярск — Москва, Москва — Сочи, Сочи — Москва, Сочи — Красноярск. Для определения идентификаторов рейсов сформируем вспомогательный запрос, в котором даты начала и конца рассматриваемого периода времени зададим с помощью функции bookings.now. Использование этой функции необходимо, поскольку в будущих версиях базы данных могут быть представлены другие диапазоны дат.

```sql
SELECT flight_no, flight_id, departure_city,
arrival_city, scheduled_departure
FROM flights_v
WHERE scheduled_departure
BETWEEN bookings.now() AND bookings.now() + INTERVAL '15 days'
AND ( departure_city, arrival_city ) IN
( ( 'Красноярск', 'Москва' ),
( 'Москва', 'Сочи'),
( 'Сочи', 'Москва' ),
( 'Сочи', 'Красноярск' )
)
ORDER BY departure_city, arrival_city, scheduled_departure;
```

Обратите внимание на предикат IN: в нем используются не индивидуальные значения, а пары значений.

Предположим, что в течение указанного интервала времени пассажир планирует совершить перелеты по маршруту: Красноярск — Москва, Москва — Сочи, Сочи — Москва, Москва — Сочи, Сочи — Красноярск. Выполнив вспомогательный запрос, выберем следующие идентификаторы рейсов (в этом же порядке): 13829, 4728, 30523, 7757, 30829.

```sql
WITH sell_tickets AS
( INSERT INTO ticket_flights_tmp
( ticket_no, flight_id, fare_conditions, amount )
VALUES ( '1234567890123', 13829, 'Economy', 10500 ),
( '1234567890123', 4728, 'Economy', 3400 ),
( '1234567890123', 30523, 'Economy', 3400 ),
( '1234567890123', 7757, 'Economy', 3400 ),
( '1234567890123', 30829, 'Economy', 12800 )
RETURNING *
)
UPDATE tickets_directions td
SET last_ticket_time = current_timestamp,
tickets_num = tickets_num +
( SELECT count( * )
FROM sell_tickets st, flights_v f
WHERE st.flight_id = f.flight_id
AND f.departure_city = td.departure_city
AND f.arrival_city = td.arrival_city
)
WHERE ( td.departure_city, td.arrival_city ) IN
( SELECT departure_city, arrival_city
FROM flights_v
WHERE flight_id IN ( SELECT flight_id FROM sell_tickets )
);
UPDATE 4
```

В этой версии запроса предусмотрен единовременный ввод нескольких строк в таблицу ticket_flights_tmp, причем перелеты могут выполняться на различных рейсах. Поэтому необходимо преобразовать список идентификаторов этих рейсов в множество пар «город отправления — город прибытия», поскольку именно для таких пар и ведется подсчет числа забронированных перелетов. Эта задача решается в предложении WHERE, где вложенный подзапрос формирует список идентификаторов рейсов, а внешний подзапрос преобразует этот список в множество пар «город отправления — город прибытия». Затем с помощью предиката IN производится отбор строк таблицы tickets_directions для обновления.

Теперь обратимся к предложению SET. Подзапрос с функцией count вычисляет количество перелетов по каждому направлению. Это коррелированный подзапрос: он выполняется для каждой строки, отобранной в предложении WHERE. В нем используется соединение временной таблицы sell_tickets с представлением flights_v. Это нужно для того, чтобы подсчитать все перелеты, соответствующие паре атрибутов «город отправления — город прибытия», взятых из текущей обновляемой строки таблицы tickets_directions. Этот подзапрос позволяет учесть такой факт: рейсы могут иметь различные идентификаторы flight_id, но при этом соответствовать одному и тому же направлению, а в таблице tickets_directions учитываются именно направления.

В случае попытки повторного бронирования одного и того же перелета для данного пассажира, т. е. ввода строки с дубликатом первичного ключа, такая строка будет отвергнута, и будет сгенерировано сообщение об ошибке. В таком случае и таблица tickets_directions не будет обновлена.

Давайте посмотрим, что изменилось в таблице tickets_directions.

```sql
SELECT departure_city AS dep_city,
arrival_city AS arr_city,
last_ticket_time,
tickets_num AS num
FROM tickets_directions
WHERE tickets_num > 0
ORDER BY departure_city, arrival_city;

По маршруту Москва — Сочи наш пассажир приобретал два билета, что и отражено в выборке.
dep_city | arr_city | last_ticket_time | num
------------+------------+----------------------------+-----
Красноярск | Москва | 2017-02-04 14:02:23.769443 | 1
Москва | Сочи | 2017-02-04 14:02:23.769443 | 2
Сочи | Красноярск | 2017-02-04 14:02:23.769443 | 1
Сочи | Москва | 2017-02-04 14:02:23.769443 | 1
(4 строки)
```

А это информация о каждом перелете, забронированном нашим пассажиром:

```sql
SELECT * FROM ticket_flights_tmp;

ticket_no | flight_id | fare_conditions | amount
---------------+-----------+-----------------+----------
1234567890123 | 13829 | Economy | 10500.00
1234567890123 | 4728 | Economy | 3400.00
1234567890123 | 30523 | Economy | 3400.00
1234567890123 | 7757 | Economy | 3400.00
1234567890123 | 30829 | Economy | 12800.00
(5 строк)
```

Задание. Модифицируйте запрос и таблицу tickets_directions так, чтобы учет числа забронированных перелетов по различным маршрутам выполнялся для каждого класса обслуживания: Economy, Business и Comfort.

**Решение 7.8:**

```sql
demo=# WITH sell_tickets AS
( INSERT INTO ticket_flights_tmp
( ticket_no, flight_id, fare_conditions, amount )
VALUES
('1234567890123', 27477, 'Economy', 33300),
('1234567890123', 9355, 'Business', 40900),
('1234567890123', 60552, 'Business', 40900),
('1234567890123', 9339, 'Business', 40900),
('1234567890123', 61099, 'Economy', 39100)
RETURNING *
)
UPDATE tickets_directions td
SET last_ticket_time = current_timestamp,
tickets_num = tickets_num +
( SELECT count( * )
FROM sell_tickets st, flights_v f
WHERE st.flight_id = f.flight_id
AND f.departure_city = td.departure_city
AND f.arrival_city = td.arrival_city),
economy = (SELECT count( * )
FROM sell_tickets st, flights_v f
WHERE st.flight_id = f.flight_id
AND f.departure_city = td.departure_city
AND f.arrival_city = td.arrival_city
AND fare_conditions = 'Economy'),
comfort = (SELECT count( * )
FROM sell_tickets st, flights_v f
WHERE st.flight_id = f.flight_id
AND f.departure_city = td.departure_city
AND f.arrival_city = td.arrival_city
AND fare_conditions = 'Comfort'),
business = (SELECT count( * )
FROM sell_tickets st, flights_v f
WHERE st.flight_id = f.flight_id
AND f.departure_city = td.departure_city
AND f.arrival_city = td.arrival_city
AND fare_conditions = 'Business')
WHERE ( td.departure_city, td.arrival_city ) IN
( SELECT departure_city, arrival_city
FROM flights_v
WHERE flight_id IN ( SELECT flight_id FROM sell_tickets )
);
UPDATE 4

demo=# SELECT departure_city AS dep_city,
arrival_city AS arr_city,
last_ticket_time,
tickets_num AS num,
economy,
comfort,
business
FROM tickets_directions
WHERE tickets_num > 0
ORDER BY departure_city, arrival_city;

  dep_city  |  arr_city  |      last_ticket_time      | num | economy | comfort | business
------------+------------+----------------------------+-----+---------+---------+----------
 Красноярск | Москва     | 2023-05-29 21:11:42.705345 |   1 |       1 |       0 |        0
 Москва     | Сочи       | 2023-05-29 21:11:42.705345 |   2 |       0 |       0 |        2
 Сочи       | Красноярск | 2023-05-29 21:11:42.705345 |   1 |       1 |       0 |        0
 Сочи       | Москва     | 2023-05-29 21:11:42.705345 |   1 |       0 |       0 |        1
(4 rows)
```

</details>
<details>
<summary>Задание 7.9 (подзапросы, оконные функции)</summary>
Предположим, что руководство нашей авиакомпании решило отказаться от использования самолетов компаний Boeing и Airbus, имеющих наименьшее количество пассажирских мест в салонах. Мы должны соответствующим образом откорректировать таблицу «Самолеты» (aircrafts_tmp). Мы предлагаем такой алгоритм.

Шаг 1. Для каждой модели вычислить общее число мест в салоне.

Шаг 2. Используя оконную функцию rank, присвоить моделям ранги на основе числа мест (упорядочив их по возрастанию числа мест). Ранжирование выполняется в пределах каждой компании-производителя, т. е. для Boeing и для Airbus — отдельно. Ранг, равный 1, соответствует наименьшему числу мест.

Шаг 3. Выполнить удаление тех строк из таблицы aircrafts_tmp, которые удовлетворяют следующим требованиям: модель — Boeing или Airbus, а число мест в салоне — минимальное из всех моделей данной компании-производителя, т. е. модель имеет ранг, равный 1.

```sql
WITH aicrafts_seats AS
( SELECT aircraft_code, model, seats_num,
rank() OVER (
PARTITION BY left( model, strpos( model, ' ' ) - 1 )
ORDER BY seats_num
)
FROM
( SELECT a.aircraft_code, a.model, count( * ) AS seats_num
FROM aircrafts_tmp a, seats s
WHERE a.aircraft_code = s.aircraft_code
GROUP BY 1, 2
) AS seats_numbers
)
DELETE FROM aircrafts_tmp a
USING aicrafts_seats a_s
WHERE a.aircraft_code = a_s.aircraft_code
AND left( a.model, strpos( a.model, ' ' ) - 1 )
IN ( 'Boeing', 'Airbus' )
AND a_s.rank = 1
RETURNING *;
```

Шаг 1 выполняется в подзапросе в предложении WITH. Шаг 2 — в главном запросе в предложении WITH. Шаг 3 реализуется командой DELETE. 

Обратите внимание, что название компании-производителя мы определяем путем взятия подстроки от значения атрибута model: от начала строки до пробельного символа (используем функции left и strpos). Мы включили предложение RETURNING *, чтобы увидеть, какие именно модели были удалены. Предложение WITH выдает такой результат:

```sql
aircraft_code | model | seats_num | rank
---------------+---------------------+-----------+------
319 | Airbus A319-100 | 116 | 1
320 | Airbus A320-200 | 140 | 2
321 | Airbus A321-200 | 170 | 3
733 | Boeing 737-300 | 130 | 1
763 | Boeing 767-300 | 222 | 2
773 | Boeing 777-300 | 402 | 3
CR2 | Bombardier CRJ-200 | 50 | 1
CN1 | Cessna 208 Caravan | 12 | 1
SU9 | Sukhoi SuperJet-100 | 97 | 1
(9 строк)
```

Очевидно, что должны быть удалены модели с кодами 319 и 733. После выполнения запроса получим (это работает предложение RETURNING *):

```sql
-[ RECORD 1 ]--+----------------
aircraft_code | 319
model | Airbus A319-100
range | 6700
aircraft_code | 319
model | Airbus A319-100
seats_num | 116
rank | 1
-[ RECORD 2 ]--+----------------
aircraft_code | 733
model | Boeing 737-300
range | 4200
aircraft_code | 733
model | Boeing 737-300
seats_num | 130
rank | 1
DELETE 2
```

Обратите внимание, что в результате были выведены комбинированные строки, полученные при соединении таблицы aircrafts_tmp с временной таблицей aicrafts_seats, указанной в предложении USING. Но удалены были, конечно, строки из таблицы aircrafts_tmp.

Задание. Предложите другой вариант решения этой задачи. Например, можно поступить так: оставить предложение WITH без изменений, из команды DELETE
убрать предложение USING, а в предложении WHERE вместо соединения таблиц использовать подзапрос с предикатом IN для получения списка кодов удаляемых моделей самолетов.

Еще один вариант решения задачи связан с использованием представлений, которые мы рассматривали в главе 5. Можно создать представление на основе
таблиц «Самолеты» (aircrafts) и «Места» (seats) и перенести конструкцию с функциями left и strpos в представление. В нем будут вычисляемые столбцы: company — «Компания-производитель самолетов» и seats_num — «Число мест».

```sql
CREATE VIEW aircrafts_seats AS
( SELECT a.aircraft_code,
a.model,
left( a.model,
strpos( a.model, ' ' ) - 1 ) AS company,
count( * ) AS seats_num
FROM aircrafts a, seats s
WHERE a.aircraft_code = s.aircraft_code
GROUP BY 1, 2, 3
);
```

Имея это представление, можно использовать его в конструкции WITH. При этом вызов функции rank может упроститься:

```sql
rank() OVER ( PARTITION BY company ORDER BY seats_num )
```

Для выбора удаляемых строк в команде DELETE можно использовать, например, подзапрос в предикате IN. При этом не забывайте, что значение столбца rank для них будет равно 1.

Еще одна идея: для выбора минимальных значений числа мест в самолетах можно попытаться в качестве замены оконной функции rank использовать предложения LIMIT 1 и ORDER BY. В таком случае не потребуется также и функция min.

**Решение 7.1:**

```sql
demo=# -- Первый вариант решения

demo=# WITH aircrafts_seats AS
( SELECT aircraft_code, model, seats_num,
rank() OVER (
PARTITION BY left( model, strpos( model, ' ' ) - 1 )
ORDER BY seats_num
)
FROM
( SELECT a.aircraft_code, a.model, count( * ) AS seats_num
FROM aircrafts_tmp a, seats s
WHERE a.aircraft_code = s.aircraft_code
GROUP BY 1, 2
) AS seats_numbers
)
DELETE FROM aircrafts_tmp a
WHERE aircraft_code IN
(SELECT aircraft_code
FROM aircrafts_seats
WHERE left( a.model, strpos( a.model, ' ' ) - 1 )
IN ( 'Boeing', 'Airbus' )
AND rank = 1)
RETURNING *;

 aircraft_code |      model      | range
---------------+-----------------+-------
 319           | Airbus A319-100 |  6700
 733           | Boeing 737-300  |  4200
(2 rows)

DELETE 2

demo=# -- Второй вариант решения

demo=# INSERT INTO aircrafts_tmp
VALUES ( '319', 'Airbus A319-100', 6700 ),
( '733', 'Boeing 737-300', 4200 );

demo=# CREATE VIEW aircrafts_seats AS
( SELECT a.aircraft_code,
a.model,
left( a.model,
strpos( a.model, ' ' ) - 1 ) AS company,
count( * ) AS seats_num
FROM aircrafts a, seats s
WHERE a.aircraft_code = s.aircraft_code
GROUP BY 1, 2, 3
);

CREATE VIEW
demo=# SELECT * FROM aircrafts_seats;

 aircraft_code |        model        |  company   | seats_num
---------------+---------------------+------------+-----------
 CN1           | Cessna 208 Caravan  | Cessna     |        12
 319           | Airbus A319-100     | Airbus     |       116
 SU9           | Sukhoi SuperJet-100 | Sukhoi     |        97
 321           | Airbus A321-200     | Airbus     |       170
 320           | Airbus A320-200     | Airbus     |       140
 CR2           | Bombardier CRJ-200  | Bombardier |        50
 773           | Boeing 777-300      | Boeing     |       402
 763           | Boeing 767-300      | Boeing     |       222
 733           | Boeing 737-300      | Boeing     |       130
(9 rows)

demo=# WITH aircrafts_rank AS
(SELECT aircraft_code, company, seats_num,
rank() OVER ( PARTITION BY company ORDER BY seats_num )
FROM aircrafts_seats
)
DELETE FROM aircrafts_tmp
WHERE aircraft_code IN
(SELECT aircraft_code
FROM aircrafts_rank
WHERE company IN ( 'Boeing', 'Airbus' )
AND rank = 1)
RETURNING *;

 aircraft_code |      model      | range
---------------+-----------------+-------
 319           | Airbus A319-100 |  6700
 733           | Boeing 737-300  |  4200
(2 rows)

DELETE 2
```

</details>

## Глава 8. Индексы
<details>
<summary>Задание 8.1 (NULL в индексе)</summary>
1. Предположим, что для какой-то таблицы создан уникальный индекс по двум столбцам: column1 и column2. В таблице есть строка, у которой значение атрибута column1 равно ABC, а значение атрибута column2 — NULL. Мы решили добавить в таблицу еще одну строку с такими же значениями ключевых атрибутов, т. е. column1 — ABC, а column2 — NULL. Как вы думаете, будет ли операция вставки новой строки успешной или завершится с ошибкой? Объясните ваше решение.

**Решение 8.1:**

Операции вставки новой строки будет успешной.  Значения NULL в индексе являются уникальными. Таким образом атрибуты одной строки не будут полностью совпадать с атрибутами другой.
</details>
<details>
<summary>Задание 8.2 (ускорение запроса c помощью Index'a)</summary>
2. В тексте главы шла речь о выполнении одной и той же выборки из таблицы «Билеты» (tickets) при наличии индекса по столбцу passenger_name и при его отсутствии. Вы видели, что наличие индекса ускоряет выполнение запроса почти на порядок. Если секундомер в утилите psql выключен, то включите его с помощью команды

```sql
\timing on
```

Проведите следующий эксперимент: выполните этот запрос несколько раз подряд при отсутствии индекса, а затем создайте индекс и опять выполните этот запрос несколько раз подряд.

```sql
SELECT count( * )
FROM tickets
WHERE passenger_name = 'IVAN IVANOV';
```

Вы увидите, что время выполнения повторных запросов к таблице сокращается, причем, когда создан индекс, оно сокращается на порядок. Как вы думаете, почему?

**Решение 8.2:**

```sql
demo=# SELECT count( * )
FROM tickets
WHERE passenger_name = 'IVAN IVANOV';
 count
-------
   470
(1 row)

Time: 322.540 ms

demo=# SELECT count( * )
FROM tickets
WHERE passenger_name = 'IVAN IVANOV';
 count
-------
   470
(1 row)

Time: 72.526 ms

demo=# SELECT count( * )
FROM tickets
WHERE passenger_name = 'IVAN IVANOV';
 count
-------
   470
(1 row)

Time: 70.195 ms

demo=# CREATE INDEX passenger_name_index
ON tickets ( passenger_name );

CREATE INDEX
Time: 1328.400 ms (00:01.328)
demo=# SELECT count( * )
FROM tickets
WHERE passenger_name = 'IVAN IVANOV';

 count
-------
   470
(1 row)

Time: 0.980 ms

demo=# SELECT count( * )
FROM tickets
WHERE passenger_name = 'IVAN IVANOV';

 count
-------
   470
(1 row)

Time: 0.521 ms
demo=# SELECT count( * )
FROM tickets
WHERE passenger_name = 'IVAN IVANOV';

 count
-------
   470
(1 row)

Time: 0.632 ms

demo=# -- Записи в индексе хранятся в отсортированном виде.
demo=# -- Это позволяет сократить время выполнения запроса при операциях выборки
demo=# -- по индексированным полям. Полный перебор неиндексированных строк
demo=# -- заменяется более быстрым поиском по уже отсортированному индексу.
demo=# -- Что касается уменьшения времени одинаковых повторяющихся запросов, то
demo=# -- после первого запроса используются кешированные данные.
```

</details>
<details>
<summary>Задание 8.3 (ускорение запроса c помощью Index'a)</summary>
3. Известно, что индекс значительно ускоряет работу, если при выполнении запроса из таблицы отбирается лишь небольшая часть строк. Если же эта доля велика, скажем, половина строк или более, то большого положительного эффекта от наличия индекса уже не будет, а возможно даже, что не будет практически никакого эффекта. Наша задача — проверить это утверждение на практике.
  
Обратимся к таблице «Перелеты» (ticket_flights). В ней имеется столбец «Класс обслуживания» (fare_conditions), который отличается от остальных
тем, что в нем могут присутствовать лишь три различных значения: Comfort, Business и Economy. Если секундомер в утилите psql выключен, то включите его.

Выполните запросы, подсчитывающие количество строк, в которых атрибут fare_conditions принимает одно из трех возможных значений. Каждый из запросов выполните три-четыре раза, поскольку время может немного изменяться, и подсчитайте среднее время. Обратите внимание на число строк, которые возвращает функция count для каждого значения атрибута. При этом среднее время выполнения запросов для трех различных значений атрибута fare_conditions будет различаться незначительно, поскольку в каждом случае СУБД просматривает все строки таблицы.

```sql
SELECT count( * )
FROM ticket_flights
WHERE fare_conditions = 'Comfort';

SELECT count( * )
FROM ticket_flights
WHERE fare_conditions = 'Business';

SELECT count( * )
FROM ticket_flights
WHERE fare_conditions = 'Economy';
```

Создайте индекс по столбцу fare_conditions. Конечно, в реальной ситуации такой индекс вряд ли целесообразен, но нам он нужен для экспериментов.
Проделайте те же эксперименты с таблицей ticket_flights. Будет ли различаться среднее время выполнения запросов для различных значений атрибута
fare_conditions? Почему это имеет место? В завершение этого упражнения отметим, что в случае ошибки планировщика при использовании индекса возможно не только отсутствие положительного эффекта, но и значительный отрицательный эффект.

**Решение 8.3:**

```sql
-- Выполняем запросы без индекса
SELECT count( * )
FROM ticket_flights
WHERE fare_conditions = 'Comfort';
 count
-------
 39154
(1 row)

Time: 181.010 ms
demo=# SELECT count( * )
FROM ticket_flights
WHERE fare_conditions = 'Comfort';
 count
-------
 39154
(1 row)

Time: 170.194 ms
demo=# SELECT count( * )
FROM ticket_flights
WHERE fare_conditions = 'Comfort';
 count
-------
 39154
(1 row)

Time: 168.074 ms
demo=# SELECT count( * )
FROM ticket_flights
WHERE fare_conditions = 'Business';
 count
--------
 242204
(1 row)

Time: 164.852 ms
demo=# SELECT count( * )
FROM ticket_flights
WHERE fare_conditions = 'Business';
 count
--------
 242204
(1 row)

Time: 163.771 ms
demo=# SELECT count( * )
FROM ticket_flights
WHERE fare_conditions = 'Business';
 count
--------
 242204
(1 row)

Time: 161.623 ms
demo=# SELECT count( * )
FROM ticket_flights
WHERE fare_conditions = 'Economy';
  count
---------
 2078977
(1 row)

Time: 201.452 ms
demo=# SELECT count( * )
FROM ticket_flights
WHERE fare_conditions = 'Economy';
  count
---------
 2078977
(1 row)

Time: 199.548 ms
demo=# SELECT count( * )
FROM ticket_flights
WHERE fare_conditions = 'Economy';
  count
---------
 2078977
(1 row)

Time: 199.638 ms
demo=# CREATE INDEX
ON ticket_flights (fare_conditions);
CREATE INDEX
Time: 1759.991 ms (00:01.760)

demo=# -- запускаем запросы при наличии индекса
demo=# SELECT count( * )
FROM ticket_flights
WHERE fare_conditions = 'Comfort';
 count
-------
 39154
(1 row)

Time: 4.129 ms
demo=# SELECT count( * )
FROM ticket_flights
WHERE fare_conditions = 'Comfort';
 count
-------
 39154
(1 row)

Time: 3.494 ms
demo=# SELECT count( * )
FROM ticket_flights
WHERE fare_conditions = 'Comfort';
 count
-------
 39154
(1 row)

Time: 3.502 ms
demo=# SELECT count( * )
FROM ticket_flights
WHERE fare_conditions = 'Business';
 count
--------
 242204
(1 row)

Time: 27.749 ms
demo=# SELECT count( * )
FROM ticket_flights
WHERE fare_conditions = 'Business';
 count
--------
 242204
(1 row)

Time: 19.032 ms
demo=# SELECT count( * )
FROM ticket_flights
WHERE fare_conditions = 'Business';
 count
--------
 242204
(1 row)

Time: 17.273 ms
demo=# SELECT count( * )
FROM ticket_flights
WHERE fare_conditions = 'Economy';
  count
---------
 2078977
(1 row)

Time: 134.874 ms
demo=# SELECT count( * )
FROM ticket_flights
WHERE fare_conditions = 'Economy';
  count
---------
 2078977
(1 row)

Time: 105.163 ms
demo=# SELECT count( * )
FROM ticket_flights
WHERE fare_conditions = 'Economy';
  count
---------
 2078977
(1 row)

Time: 101.869 ms

demo=# -- Статистика запросов

demo=# CREATE TABLE index_stat
(fare_conditions  character varying(10) NOT NULL,
fc_count integer NOT NULL,
index character varying(3) NOT NULL,
query_time numeric(10,3) NOT NULL
);
CREATE TABLE
Time: 7.489 ms

demo=# INSERT INTO index_stat (fare_conditions,fc_count, index, query_time)
VALUES ('Comfort', 39154, 'NO', 181.010),
('Comfort', 39154, 'NO', 170.194),
('Comfort', 39154, 'NO', 168.074),
('Business', 242204, 'NO', 164.852),
('Business', 242204, 'NO', 163.771),
('Business', 242204, 'NO', 161.623),
('Economy', 2078977, 'NO', 201.452),
('Economy', 2078977, 'NO', 199.548),
('Economy', 2078977, 'NO', 199.638),
('Comfort', 39154, 'YES',  4.129),
('Comfort', 39154, 'YES', 3.494),
('Comfort', 39154, 'YES', 3.502),
('Business', 242204, 'YES', 27.749),
('Business', 242204, 'YES', 19.032),
('Business', 242204, 'YES', 17.273),
('Economy', 2078977, 'YES', 134.874),
('Economy', 2078977, 'YES', 105.163),
('Economy', 2078977, 'YES', 101.869);
INSERT 0 18
Time: 11.062 ms

demo=#
SELECT  fare_conditions, fc_count, index, avg(query_time)
FROM index_stat
GROUP BY 1, 2, 3
ORDER BY 1,3;
 fare_conditions | fc_count | index |         avg
-----------------+----------+-------+----------------------
 Business        |   242204 | NO    | 163.4153333333333333
 Business        |   242204 | YES   |  21.3513333333333333
 Comfort         |    39154 | NO    | 173.0926666666666667
 Comfort         |    39154 | YES   |   3.7083333333333333
 Economy         |  2078977 | NO    | 200.2126666666666667
 Economy         |  2078977 | YES   | 113.9686666666666667
(6 rows)

demo=# -- Среднее время выполнения запросов по разным выборкам с индексом и без сильно различается в зависимости от селективности.
demo=# -- У Economy самая большая доля строк, т.е. самая маленькая селективность.
demo=# -- Поэтому разница во времени выполнения запросов с индексом и без в этом случае минимальная.
demo=# -- Индекс в запросе с выборкой по Economy наименее эффективен.
```

</details>
<details>
<summary>Задание 8.4 (index по двум столбцам, QUERY PLAN)</summary>
4. Для одной из таблиц создайте индекс по двум столбцам, причем по одному из них укажите убывающий порядок значений столбца, а по другому — возрастающий. Значения NULL у первого столбца должны располагаться в начале, а у второго — в конце. Посмотрите полученный индекс с помощью команд psql

```sql
\d имя_таблицы
\di+ имя_индекса
```

Обратите внимание, что первая команда выведет не только имя индекса, но также и имена столбцов, по которым он создан, а вторая команда выведет размер индекса.

Подберите запросы, в которых созданный индекс предположительно должен использоваться, а также запросы, в которых он использоваться, по вашему мнению, не будет. Проверьте ваши гипотезы, выполнив запросы. Объясните полученные результаты.

**Решение 8.4:**

```sql
emo=# CREATE INDEX
ON tickets (passenger_name DESC NULLS FIRST, book_ref ASC NULLS LAST);
CREATE INDEX

demo=# \d tickets
                        Table "bookings.tickets"
     Column     |         Type          | Collation | Nullable | Default
----------------+-----------------------+-----------+----------+---------
 ticket_no      | character(13)         |           | not null |
 book_ref       | character(6)          |           | not null |
 passenger_id   | character varying(20) |           | not null |
 passenger_name | text                  |           | not null |
 contact_data   | jsonb                 |           |          |
Indexes:
    "tickets_pkey" PRIMARY KEY, btree (ticket_no)
    "tickets_passenger_name_book_ref_idx" btree (passenger_name DESC, book_ref)
Foreign-key constraints:
    "tickets_book_ref_fkey" FOREIGN KEY (book_ref) REFERENCES bookings(book_ref)
Referenced by:
    TABLE "ticket_flights" CONSTRAINT "ticket_flights_ticket_no_fkey" FOREIGN KEY (ticket_no) REFERENCES tickets(ticket_no)

demo=# -- Создался индекс с атонаименованием "tickets_passenger_name_book_ref_idx" btree (passenger_name DESC, book_ref)
demo=# -- DESC автоматически предполагает NULLS FIRST.
demo=# -- Если ничего не указано, то по-умолчанию ASC NULLS LAST.


demo=# \di+ tickets_passenger_name_book_ref_idx
                                                        List of relations
  Schema  |                Name                 | Type  |  Owner   |  Table  | Persistence | Access method | Size  | Description
----------+-------------------------------------+-------+----------+---------+-------------+---------------+-------+-------------
 bookings | tickets_passenger_name_book_ref_idx | index | postgres | tickets | permanent   | btree         | 34 MB |
(1 row)

demo=# -- Выполняем запрос с сортировкой по индексированному столбцу.
demo=# -- Планировщик использует индекс, т.к. он уже отсортирован.
demo=# EXPLAIN SELECT * 
FROM tickets 
ORDER BY passenger_name;
                                                      QUERY PLAN
-----------------------------------------------------------------------------------------------------------------------
 Index Scan Backward using tickets_passenger_name_book_ref_idx on tickets  (cost=0.42..85376.48 rows=829071 width=104)
(1 row)

demo=# -- Делаем запрос с фильтрацией по неиндексированному столбцу, но с выборкой по индексированному текстовому.
demo=# -- Если в запросе не используются индексированные столбцы, то, естественно, не используется и индекс,
demo=# --  но в данном случае планировщик не использует индекс в индексированном столбце passenger_name.
demo=# -- Причина предположительно в том, что шаблон начинается с алфавитных символов, 
demo=# -- которые подвержены преобразованию регистра, что приводит к решению планировщика не использовать индекс b-дерева.
demo=# -- Остальные условия сравнения по шаблону соблюдены: 
demo=# -- шаблон привязан к началу строки, локаль С, высокая селективность, столбец в индексе ведущий.

demo=# EXPLAIN SELECT ticket_no, passenger_id, passenger_name
FROM tickets
WHERE passenger_name ~ '^YAK'
ORDER BY passenger_id;
                                    QUERY PLAN
----------------------------------------------------------------------------------
 Gather Merge  (cost=19208.90..19216.37 rows=64 width=42)
   Workers Planned: 2
   ->  Sort  (cost=18208.88..18208.96 rows=32 width=42)
         Sort Key: passenger_id
         ->  Parallel Seq Scan on tickets  (cost=0.00..18208.08 rows=32 width=42)
               Filter: (passenger_name ~ '^YAK'::text)
(6 rows)

demo=# -- Возвращаем таблицу к исходному состоянию, удаляем созданный индекс.
demo=# DROP INDEX tickets_passenger_name_book_ref_idx;
DROP INDEX
```

</details>
<details>
<summary>Задание 8.5 (комбинации Index'ов, QUERY PLAN)</summary>
В сложных базах данных целесообразно использование комбинаций индексов. Иногда бывают более полезны комбинированные индексы по нескольким столбцам, чем отдельные индексы по единичным столбцам. В реальных ситуациях часто приходится делать выбор, т. е. находить компромисс, между, например, созданием двух индексов по каждому из двух столбцов таблицы либо созданием одного индекса по двум столбцам этой таблицы, либо созданием всех трех индексов. Выбор зависит от того, запросы какого вида будут выполняться чаще всего. Предложите какую-нибудь таблицу в базе данных «Авиаперевозки» и смоделируйте ситуации, в которых вы приняли бы одно из этих трех возможных решений. Воспользуйтесь документацией на PostgreSQL.

**Решение 8.5:**

```sql
-- Предположим, что от отдела маркетинга и продаж пришел запрос
-- на список забронированных билетов по тарифу Comfort
-- за последний месяц с указанием ФИО и телефона пассажира.

demo=# \timing on
Timing is on.
demo=# WITH recent_b AS
(SELECT book_ref, book_date,
EXTRACT(YEAR FROM book_date) AS year,
EXTRACT(MONTH FROM book_date) AS month,
rank() OVER (
PARTITION BY EXTRACT(YEAR FROM book_date)
ORDER BY EXTRACT(MONTH FROM book_date) DESC
) AS rank
FROM bookings
)
SELECT rb.book_date, tf.fare_conditions, t.passenger_name, 
t.contact_data->'phone' AS phone
FROM tickets AS t
JOIN ticket_flights AS tf ON t.ticket_no = tf.ticket_no
JOIN recent_b AS rb ON t.book_ref = rb.book_ref
WHERE tf.fare_conditions = 'Comfort' AND rb.rank = 1;
Time: 5959.698 ms (00:05.960)

       book_date        | fare_conditions |     passenger_name      |     phone
------------------------+-----------------+-------------------------+----------------
 2016-10-12 03:49:00+04 | Comfort         | NINA NIKOLAEVA          | "+70356001154"
 2016-10-10 14:13:00+04 | Comfort         | MARIYA IVANOVA          | "+70476510302"
 2016-10-09 01:21:00+04 | Comfort         | ELENA BOGDANOVA         | "+70264878099"
...

-- Посмотрим на плане запроса через EXPLAIN ANALYZE почему он так долго выполняется. В оконной функции гоняются циклы и используется диск. Здесь приведем краткую форму плана запроса без оценок стоимости EXPLAIN (COSTS FALSE).

Если в SQL-запросе есть предложение ORDER BY, то индекс может позволить избежать этапа сортировки выбранных строк. 
Индексы более полезны, когда из таблицы выбирается лишь небольшая доля строк, т. е. при высокой селективности выборки.



                                                            QUERY PLAN
----------------------------------------------------------------------------------------------------------------------------------
 Nested Loop
   ->  Hash Join
         Hash Cond: (t.book_ref = rb.book_ref)
         ->  Seq Scan on tickets t
         ->  Hash
               ->  Subquery Scan on rb
                     Filter: (rb.rank = 1)
                     ->  WindowAgg
                           ->  Sort
                                 Sort Key: (EXTRACT(year FROM bookings.book_date)), (EXTRACT(month FROM bookings.book_date)) DESC
                                 ->  Seq Scan on bookings
   ->  Index Scan using ticket_flights_pkey on ticket_flights tf
         Index Cond: (ticket_no = t.ticket_no)
         Filter: ((fare_conditions)::text = 'Comfort'::text)
(14 rows)

-- Посмотрим уже существующие индексы для представлений, участвующих в запросе

demo=# \di
                                     List of relations
  Schema  |                   Name                    | Type  |  Owner   |      Table
----------+-------------------------------------------+-------+----------+-----------------
 bookings | bookings_pkey                             | index | postgres | bookings
 bookings | ticket_flights_pkey                       | index | postgres | ticket_flights
 bookings | tickets_pkey                              | index | postgres | tickets
...

-- Из описаний трех таблиц скопируем индексы
Indexes:
"bookings_pkey" PRIMARY KEY, btree (book_ref)
Indexes:
    "tickets_pkey" PRIMARY KEY, btree (ticket_no)
Indexes:
    "ticket_flights_pkey" PRIMARY KEY, btree (ticket_no, flight_id)

-- Все индексы являются первичными ключами. Индексов, созданных под запросы, нет.
-- В оконной функции используется ORDER BY для book_date. 
-- Проверим сможет ли индекс для  book_date ускорить запрос.

demo=# CREATE INDEX ON bookings (book_date);
CREATE INDEX

demo=# WITH recent_b AS
(SELECT book_ref, book_date,
EXTRACT(YEAR FROM book_date) AS year,
EXTRACT(MONTH FROM book_date) AS month,
rank() OVER (
PARTITION BY EXTRACT(YEAR FROM book_date)
ORDER BY EXTRACT(MONTH FROM book_date) DESC
) AS rank
FROM bookings
)
SELECT rb.book_date, tf.fare_conditions, t.passenger_name,
t.contact_data->'phone' AS phone
FROM tickets AS t
JOIN ticket_flights AS tf ON t.ticket_no = tf.ticket_no
JOIN recent_b AS rb ON t.book_ref = rb.book_ref
WHERE tf.fare_conditions = 'Comfort' AND rb.rank = 1;
Time: 5909.491 ms (00:05.909)

-- Индекс никак не улучшил время выполнения запроса.
demo=# DROP INDEX bookings_book_date_idx;
DROP INDEX

-- В запросе после оконной функции в общем табличном выражении есть JOIN таблиц с условием WHERE tf.fare_conditions = 'Comfort' AND rb.rank = 1.
-- Проверим сможет ли индекс по fare_conditions ускорить запрос. 
--  fare_conditions = 'Comfort' является относительно селективным.

demo=# SELECT fare_conditions, count(fare_conditions)
FROM ticket_flights
GROUP BY 1;
 fare_conditions |  count
-----------------+---------
 Business        |  242204
 Comfort         |   39154
 Economy         | 2078977
(3 rows)

demo=# CREATE INDEX ON ticket_flights (fare_conditions);
CREATE INDEX

demo=# WITH recent_b AS
(SELECT book_ref, book_date,
EXTRACT(YEAR FROM book_date) AS year,
EXTRACT(MONTH FROM book_date) AS month,
rank() OVER (
PARTITION BY EXTRACT(YEAR FROM book_date)
ORDER BY EXTRACT(MONTH FROM book_date) DESC
) AS rank
FROM bookings
)
SELECT rb.book_date, tf.fare_conditions, t.passenger_name,
t.contact_data->'phone' AS phone
FROM tickets AS t
JOIN ticket_flights AS tf ON t.ticket_no = tf.ticket_no
JOIN recent_b AS rb ON t.book_ref = rb.book_ref
WHERE tf.fare_conditions = 'Comfort' AND rb.rank = 1;
Time: 6069.038 ms (00:06.069)

-- Индекс ticket_flights_fare_conditions_idx запрос не ускорил.

                                                            QUERY PLAN
----------------------------------------------------------------------------------------------------------------------------------
 Nested Loop
   ->  Hash Join
         Hash Cond: (t.book_ref = rb.book_ref)
         ->  Seq Scan on tickets t
         ->  Hash
               ->  Subquery Scan on rb
                     Filter: (rb.rank = 1)
                     ->  WindowAgg
                           ->  Sort
                                 Sort Key: (EXTRACT(year FROM bookings.book_date)), (EXTRACT(month FROM bookings.book_date)) DESC
                                 ->  Seq Scan on bookings
   ->  Index Scan using ticket_flights_pkey on ticket_flights tf
         Index Cond: (ticket_no = t.ticket_no)
         Filter: ((fare_conditions)::text = 'Comfort'::text)
(14 rows)

demo=# DROP INDEX ticket_flights_fare_conditions_idx;
DROP INDEX

-- Попробуем создать составной индекс.
demo=# CREATE INDEX ON ticket_flights (ticket_no, fare_conditions);
CREATE INDEX

demo=# WITH recent_b AS
(SELECT book_ref, book_date,
EXTRACT(YEAR FROM book_date) AS year,
EXTRACT(MONTH FROM book_date) AS month,
rank() OVER (
PARTITION BY EXTRACT(YEAR FROM book_date)
ORDER BY EXTRACT(MONTH FROM book_date) DESC
) AS rank
FROM bookings
)
SELECT rb.book_date, tf.fare_conditions, t.passenger_name,
t.contact_data->'phone' AS phone
FROM tickets AS t
JOIN ticket_flights AS tf ON t.ticket_no = tf.ticket_no
JOIN recent_b AS rb ON t.book_ref = rb.book_ref
WHERE tf.fare_conditions = 'Comfort' AND rb.rank = 1;
Time: 5368.584 ms (00:05.369)

-- Посмотрим план запроса.

                                                            QUERY PLAN
----------------------------------------------------------------------------------------------------------------------------------
 Nested Loop
   ->  Hash Join
         Hash Cond: (t.book_ref = rb.book_ref)
         ->  Seq Scan on tickets t
         ->  Hash
               ->  Subquery Scan on rb
                     Filter: (rb.rank = 1)
                     ->  WindowAgg
                           ->  Sort
                                 Sort Key: (EXTRACT(year FROM bookings.book_date)), (EXTRACT(month FROM bookings.book_date)) DESC
                                 ->  Seq Scan on bookings
   ->  Index Only Scan using ticket_flights_ticket_no_fare_conditions_idx on ticket_flights tf
         Index Cond: ((ticket_no = t.ticket_no) AND (fare_conditions = 'Comfort'::text))
(13 rows)

-- Индекс "ticket_flights_ticket_no_fare_conditions_idx" btree (ticket_no, fare_conditions)
-- немного ускорил запрос, но незначительно, всего на примерно на 10%.
-- При объединении таблиц по условию fare_conditions = 'Comfort' 
-- стал использоваться только составной индекс без сканирования таблицы (Index Only Scan).
-- Вероятно, это и привело к небольшому ускорению запроса.
-- Решение оставить ли этот составной индекс ticket_flights_ticket_no_fare_conditions_idx
-- будет зависеть от того, насколько часто нужно выполнять запрос при том, что в индексе участвует
-- большая и постоянно обновляемая таблица и его поддержание потребует ресурсов.
-- Судя по месячному интервалу в формулировке задачи от отдела маркетинга - не часто.
-- Поэтому удаляем и этот индекс как слишком затратный.

demo=# DROP INDEX ticket_flights_ticket_no_fare_conditions_idx;
DROP INDEX

-- В итоге ни один из рассматриваемых вновь созданных индексов использовать не будем.
-- Вместо этого нужно подумать как оптимизировать очень тяжелой код 
-- с оконной функцией в общем табличном выражении.
```

</details>
<details>
<summary>Задание 8.6 (Index по двум и более столбцам)</summary>
Предложите какую-нибудь таблицу в базе данных «Авиаперевозки» и смоделируйте ситуацию, в которой было бы целесообразно использование индекса на основе функции или скалярного выражения от двух или более столбцов.

**Решение 8.6:**
Абстрактный запрос строит таблицу: статус рейса, его номер и аэропорты вылета - прилета, сконкотинированые в одну строку. Для чего объединять в одну строку? Просто нужен индекс по выражению по условию задачи. Все предикаты индекса участвуют в сортировке и индекс задействован. Планировщик выбрал построение битовых карт.

```sql
demo=# CREATE INDEX ON flights (status, (departure_airport || ' - ' || arrival_airport), flight_no);
CREATE INDEX

demo=# SELECT departure_airport || ' - ' || arrival_airport AS rout, flight_no, status
FROM  flights
WHERE status = 'Scheduled' AND flight_no LIKE 'PG03%' AND departure_airport || ' - ' || arrival_airport LIKE 'DME%'
ORDER BY 1;

   rout    | flight_no |  status
-----------+-----------+-----------
 DME - ARH | PG0337    | Scheduled
 DME - ARH | PG0337    | Scheduled
 DME - ARH | PG0337    | Scheduled
 DME - ARH | PG0337    | Scheduled
 DME - ARH | PG0337    | Scheduled
 DME - ARH | PG0337    | Scheduled
 DME - ARH | PG0337    | Scheduled
...
Time: 5.670 ms

                                                                  QUERY PLAN

----------------------------------------------------------------------------------------------------------------------------------------------
 Sort  (cost=1395.82..1395.84 rows=10 width=47)
   Sort Key: ((((departure_airport)::text || ' - '::text) || (arrival_airport)::text))
   ->  Bitmap Heap Scan on flights  (cost=182.99..1395.65 rows=10 width=47)
         Recheck Cond: ((status)::text = 'Scheduled'::text)
         Filter: ((flight_no ~~ 'PG03%'::text) AND ((((departure_airport)::text || ' - '::text) || (arrival_airport)::text) ~~ 'DME%'::text))
         ->  Bitmap Index Scan on flights_status_expr_flight_no_idx  (cost=0.00..182.99 rows=15293 width=0)
               Index Cond: ((status)::text = 'Scheduled'::text)
(7 rows)
```

</details>

## Глава 9. Транзакции
<details>
<summary>Задание 9.2 (Read Committed)</summary>
2. Транзакции, работающие на уровне изоляции Read Committed, видят только свои собственные обновления и обновления, зафиксированные параллельными транзакциями. При этом нужно учитывать, что иногда могут возникать ситуации, которые на первый взгляд кажутся парадоксальными, но на самом деле все происходит в строгом соответствии с этим принципом.

Воспользуемся таблицей «Самолеты» (aircrafts) или ее копией. Предположим, что мы решили удалить из таблицы те модели, дальность полета которых менее 2 000 км. В таблице представлена одна такая модель — Cessna 208 Caravan, имеющая дальность полета 1 200 км. Для выполнения удаления мы организовали транзакцию. Однако параллельная транзакция, которая, причем, началась раньше, успела обновить таблицу таким образом, что дальность полета самолета Cessna 208 Caravan стала составлять 2 100 км, а вот для самолета Bombardier CRJ-200 она, напротив, уменьшилась до 1 900 км. Таким образом, в результате выполнения операций обновления в таблице по-прежнему присутствует строка, удовлетворяющая первоначальному условию, т. е. значение атрибута range у которой меньше 2000.

Наша задача: проверить, будет ли в результате выполнения двух транзакций удалена какая-либо строка из таблицы.

На первом терминале начнем транзакцию, при этом уровень изоляции Read Committed в команде указывать не будем, т. к. он принят по умолчанию:

```sql
BEGIN;
BEGIN

SELECT *
FROM aircrafts_tmp
WHERE range < 2000;
aircraft_code | model | range
---------------+--------------------+-------
CN1 | Cessna 208 Caravan | 1200
(1 строка)
UPDATE aircrafts_tmp
SET range = 2100
WHERE aircraft_code = 'CN1';
UPDATE 1
UPDATE aircrafts_tmp
SET range = 1900
WHERE aircraft_code = 'CR2';
UPDATE 1
```

На втором терминале начнем вторую транзакцию, которая и будет пытаться удалить строки, у которых значение атрибута range меньше 2000.

```sql
BEGIN;
BEGIN
SELECT * FROM aircrafts_tmp
WHERE range < 2000;

aircraft_code | model | range
---------------+--------------------+-------
CN1 | Cessna 208 Caravan | 1200
(1 строка)

DELETE FROM aircrafts_tmp WHERE range < 2000;
```

Введя команду DELETE, мы видим, что она не завершается, а ожидает, когда со строки, подлежащей удалению, будет снята блокировка. Блокировка, установленная командой UPDATE в первой транзакции, снимается только при завершении транзакции, а завершение может иметь два исхода: фиксацию изменений с помощью команды COMMIT (или END) или отмену изменений с помощью команды ROLLBACK.

Давайте зафиксируем изменения, выполненные первой транзакцией. На первом терминале сделаем так:

```sql
COMMIT;
COMMIT
```

Тогда на втором терминале мы получим такой результат от команды DELETE:

```sql
DELETE 0
```

Чем объясняется такой результат? Он кажется нелогичным: ведь команда SELECT, выполненная в этой же второй транзакции, показывала наличие строки, удовлетворяющей условию удаления. Объяснение таково: поскольку вторая транзакция пока еще не видит изменений, произведенных в первой транзакции, то команда DELETE выбирает для
удаления строку, описывающую модель Cessna 208 Caravan, однако эта строка была заблокирована в первой транзакции командой UPDATE. Эта команда изменила значение атрибута range в этой строке.

При завершении первой транзакции блокировка с этой строки снимается (со второй строки — тоже), и команда DELETE во второй транзакции получает возможность заблокировать эту строку. При этом команда DELETE данную строку перечитывает и вновь вычисляет условие WHERE применительно к ней. Однако теперь условие WHERE для данной строки уже не выполняется, следовательно, эту строку удалять нельзя. Конечно, в таблице есть теперь другая строка, для самолета Bombardier CRJ-200, удовлетворяющая условию удаления, однако повторный поиск строк, удовлетворяющих условию WHERE в команде DELETE, не производится.

В результате не удаляется ни одна строка. Таким образом, к сожалению, имеет место нарушение согласованности, которое можно объяснить деталями реализации СУБД. Завершим вторую транзакцию:

```sql
END;
COMMIT
```

Вот что получилось в результате:

```sql
SELECT * FROM aircrafts_tmp;

aircraft_code | model | range
---------------+---------------------+-------
773 | Boeing 777-300 | 11100
763 | Boeing 767-300 | 7900
SU9 | Sukhoi SuperJet-100 | 3000
320 | Airbus A320-200 | 5700
321 | Airbus A321-200 | 5600
319 | Airbus A319-100 | 6700
733 | Boeing 737-300 | 4200
CN1 | Cessna 208 Caravan | 2100
CR2 | Bombardier CRJ-200 | 1900
(9 строк)
```

Задание. Модифицируйте сценарий выполнения транзакций: в первой транзакции вместо фиксации изменений выполните их отмену с помощью команды ROLLBACK и посмотрите, будет ли удалена строка и какая конкретно.

**Решение 9.2:**

```sql
demo=# CREATE TABLE aircrafts_tmp AS
SELECT * FROM aircrafts;
SELECT 9
 ALTER TABLE aircrafts_tmp
ADD PRIMARY KEY ( aircraft_code );
ALTER TABLE

demo=# -- Открываем второй терминал и запускаем второй сеанс psql.

demo=# BEGIN; -- Транзакция в первом терминале.
BEGIN
demo=*# SELECT *
FROM aircrafts_tmp
WHERE range < 2000;
 aircraft_code |       model        | range
---------------+--------------------+-------
 CN1           | Cessna 208 Caravan |  1200
(1 row)

demo=*# UPDATE aircrafts_tmp
SET range = 2100
WHERE aircraft_code = 'CN1';
UPDATE 1
demo=*# UPDATE aircrafts_tmp
SET range = 1900
WHERE aircraft_code = 'CR2';
UPDATE 1
demo=*#

demo=# BEGIN; -- Транзакция во втором терминале.
BEGIN
demo=*# SELECT *
FROM aircrafts_tmp
WHERE range < 2000;
 aircraft_code |       model        | range
---------------+--------------------+-------
 CN1           | Cessna 208 Caravan |  1200
(1 row)

demo=*# DELETE FROM aircrafts_tmp WHERE range < 2000; -- Транзакция не завершилась.

demo=*# COMMIT; -- -- Завершаем транзакцию в первом терминале.
COMMIT

DELETE 0 -- После завершения транзакции в первом терминале выполнена команда  DELETE во втором, но ничего не удалено.

demo=*# END; -- Завершаем транзакцию во втором терминале.
COMMIT
demo=# SELECT * FROM aircrafts_tmp;
 aircraft_code |        model        | range
---------------+---------------------+-------
 773           | Boeing 777-300      | 11100
 763           | Boeing 767-300      |  7900
 SU9           | Sukhoi SuperJet-100 |  3000
 320           | Airbus A320-200     |  5700
 321           | Airbus A321-200     |  5600
 319           | Airbus A319-100     |  6700
 733           | Boeing 737-300      |  4200
 CN1           | Cessna 208 Caravan  |  2100
 CR2           | Bombardier CRJ-200  |  1900
(9 rows)

demo=# -- Bombardier CRJ-200 не удален из-за нарушения согласованности.

demo=# -- Возвращаем дальности полета в исходное состояние.
demo=# UPDATE aircrafts_tmp
SET range = 1200
WHERE aircraft_code = 'CN1';
UPDATE 1
demo=# UPDATE aircrafts_tmp
SET range = 2700
WHERE aircraft_code = 'CR2';
UPDATE 1
demo=# SELECT *
FROM aircrafts_tmp;
 aircraft_code |        model        | range
---------------+---------------------+-------
 773           | Boeing 777-300      | 11100
 763           | Boeing 767-300      |  7900
 SU9           | Sukhoi SuperJet-100 |  3000
 320           | Airbus A320-200     |  5700
 321           | Airbus A321-200     |  5600
 319           | Airbus A319-100     |  6700
 733           | Boeing 737-300      |  4200
 CN1           | Cessna 208 Caravan  |  1200
 CR2           | Bombardier CRJ-200  |  2700
(9 rows)

demo=# -- Модифицируем сценарий выполнения транзакций.

demo=# BEGIN; -- Первый терминал.
BEGIN
demo=*# UPDATE aircrafts_tmp
SET range = 2100
WHERE aircraft_code = 'CN1'
demo-*# ;
UPDATE 1
demo=*# UPDATE aircrafts_tmp
SET range = 1900
WHERE aircraft_code = 'CR2';
UPDATE 1

demo=# BEGIN; -- Второй терминал.
BEGIN
demo=*# DELETE FROM aircrafts_tmp WHERE range < 2000;

demo=*# ROLLBACK; -- Первый терминал.
ROLLBACK

DELETE 1 -- Второй терминал
demo=*# END;
COMMIT

demo=# SELECT * FROM aircrafts_tmp;
 aircraft_code |        model        | range
---------------+---------------------+-------
 773           | Boeing 777-300      | 11100
 763           | Boeing 767-300      |  7900
 SU9           | Sukhoi SuperJet-100 |  3000
 320           | Airbus A320-200     |  5700
 321           | Airbus A321-200     |  5600
 319           | Airbus A319-100     |  6700
 733           | Boeing 737-300      |  4200
 CR2           | Bombardier CRJ-200  |  2700
(8 rows)

demo=# -- Первая транзакция откатилась без выполнения апдейтов.
demo=# -- Когда разблокировались строки, выполнилась DELETE во второй транзакции
demo=# -- для строки Cessna 208 Caravan, единственной подпадающей под условие WHERE range < 2000.

demo=# INSERT INTO aircrafts_tmp
VALUES ( 'CN1', 'Cessna 208 Caravan', 1200 );
INSERT 0 1
```

</details>
<details>
<summary>Задание 9.4 (Read Committed, чтение фантомных строк)</summary>
4. На уровне изоляции транзакций Read Committed имеет место такой феномен, как чтение фантомных строк. Такие строки могут появляться в выборке как в результате добавления новых строк параллельной транзакцией, так и вследствие изменения ею значений атрибутов, участвующих в формировании условия выборки. Рассмотрим пример, иллюстрирующий вторую из указанных причин.
  
На первом терминале организуем транзакцию. Она будет иметь уровень изоляции Read Committed:

```sql
BEGIN;
BEGIN
SELECT *
FROM aircrafts_tmp
WHERE range > 6000;

aircraft_code | model | range
---------------+-----------------+-------
773 | Boeing 777-300 | 11100
763 | Boeing 767-300 | 7900
319 | Airbus A319-100 | 6700
(3 строки)
```

На втором терминале организуем транзакцию и обновим одну из строк таблицы таким образом, чтобы эта строка стала удовлетворять условию отбора строк, заданному в первой транзакции.

```sql
BEGIN;
BEGIN

UPDATE aircrafts_tmp
SET range = 6100
WHERE aircraft_code = '320';
UPDATE 1
```

Сразу завершим вторую транзакцию, чтобы первая транзакция увидела эти изменения.

```sql
END;
COMMIT
```

На первом терминале повторим ту же самую выборку:

```sql
SELECT *
FROM aircrafts_tmp
WHERE range > 6000;

aircraft_code | model | range
---------------+-----------------+-------
773 | Boeing 777-300 | 11100
763 | Boeing 767-300 | 7900
319 | Airbus A319-100 | 6700
320 | Airbus A320-200 | 6100
(4 строки)
```

Транзакция еще не завершилась, но она уже увидела новую строку, обновленную зафиксированной параллельной транзакцией. Теперь эта строка стала соответствовать условию выборки. Таким образом, не изменяя критерий выборки, мы получили другое множество строк.

Завершим теперь и первую транзакцию:

```sql
END;
COMMIT
```

Задание. Модифицируйте этот эксперимент: вместо операции UPDATE используйте операцию INSERT.

**Решение 9.4:**

```sql
demo=# BEGIN; -- Первый терминал.
BEGIN
demo=*# SELECT *
FROM aircrafts_tmp
WHERE range > 6000;
 aircraft_code |      model      | range
---------------+-----------------+-------
 773           | Boeing 777-300  | 11100
 763           | Boeing 767-300  |  7900
 319           | Airbus A319-100 |  6700
(3 rows)

demo=# BEGIN; -- Второй терминал.
BEGIN
demo=*# INSERT INTO aircrafts_tmp
VALUES ( '001', 'Airbus 000-000', 6100 );
INSERT 0 1
demo=*# END;
COMMIT

demo=*# SELECT * FROM aircrafts_tmp WHERE range > 6000; -- Первый терминал.
 aircraft_code |      model      | range
---------------+-----------------+-------
 773           | Boeing 777-300  | 11100
 763           | Boeing 767-300  |  7900
 319           | Airbus A319-100 |  6700
 001           | Airbus 000-000  |  6100
(4 rows)

demo=*# END;
COMMIT

-- Эффект фантомных строк при использовании INSERT.
-- Первая незавершенная транзакция видит вставленную строку во второй завершенной.
```

</details>
<details>
<summary>Задание 9.5 (SELECT... FOR UPDATE)</summary>
В тексте главы была рассмотрена команда SELECT ... FOR UPDATE, выполняющая блокировку на уровне отдельных строк. Организуйте две параллельные транзакции с уровнем изоляции Read Committed и выполните с ними ряд экспериментов. В первой транзакции заблокируйте некоторое множество строк, отбираемых с помощью условия WHERE. А во второй транзакции изменяйте условие выборки таким образом, чтобы выбираемое множество строк:
  
– являлось подмножеством множества строк, выбираемых в первой транзакции;
– являлось надмножеством множества строк, выбираемых в первой транзакции;
– пересекалось с множеством строк, выбираемых в первой транзакции;
– не пересекалось с множеством строк, выбираемых в первой транзакции.

Наблюдайте за поведением команд выборки в каждой транзакции. Попробуйте обобщить ваши наблюдения.

**Решение 9.5:**

```sql
demo=# SELECT * FROM aircrafts_tmp ORDER BY range DESC;

 aircraft_code |        model        | range
---------------+---------------------+-------
 773           | Boeing 777-300      | 11100
 763           | Boeing 767-300      |  7900
 319           | Airbus A319-100     |  6700
 001           | Airbus 000-000      |  6100
 320           | Airbus A320-200     |  5700
 321           | Airbus A321-200     |  5600
 733           | Boeing 737-300      |  4200
 SU9           | Sukhoi SuperJet-100 |  3000
 CR2           | Bombardier CRJ-200  |  2700
 CN1           | Cessna 208 Caravan  |  1200
(10 rows)

demo=# -- Отбор подмножества множества строк.

demo=# BEGIN; -- Первый терминал.
BEGIN
demo=*# SELECT *
FROM aircrafts_tmp
WHERE range BETWEEN 4000 AND 7000
ORDER BY range DESC
FOR UPDATE;

 aircraft_code |      model      | range
---------------+-----------------+-------
 319           | Airbus A319-100 |  6700
 001           | Airbus 000-000  |  6100
 320           | Airbus A320-200 |  5700
 321           | Airbus A321-200 |  5600
 733           | Boeing 737-300  |  4200
(5 rows)


demo=# BEGIN; -- Второй терминал.
BEGIN

demo=*# SELECT *
FROM aircrafts_tmp
WHERE range BETWEEN 5000 AND 6000
ORDER BY range DESC
FOR UPDATE;

demo=*# END; -- Первый терминал
COMMIT

demo=*# -- Вторая транзакция выполнилась после разблокировки строк первой.
 aircraft_code |      model      | range
---------------+-----------------+-------
 320           | Airbus A320-200 |  5700
 321           | Airbus A321-200 |  5600
(2 rows)
demo=*# END; -- Второй терминал.
COMMIT

demo=# -- Отбор надмножества множества строк.

demo=# BEGIN; -- Первый терминал.
BEGIN
demo=*# SELECT *
FROM aircrafts_tmp
WHERE range BETWEEN 4000 AND 7000
ORDER BY range DESC
FOR UPDATE;

 aircraft_code |      model      | range
---------------+-----------------+-------
 319           | Airbus A319-100 |  6700
 001           | Airbus 000-000  |  6100
 320           | Airbus A320-200 |  5700
 321           | Airbus A321-200 |  5600
 733           | Boeing 737-300  |  4200
(5 rows)

demo=# BEGIN; -- Второй терминал.
BEGIN
demo=*# SELECT *
FROM aircrafts_tmp
WHERE range > 3000
ORDER BY range DESC
FOR UPDATE;

demo=*# END; -- Первый терминал.
COMMIT

-- Второй терминал
 aircraft_code |      model      | range
---------------+-----------------+-------
 773           | Boeing 777-300  | 11100
 763           | Boeing 767-300  |  7900
 319           | Airbus A319-100 |  6700
 001           | Airbus 000-000  |  6100
 320           | Airbus A320-200 |  5700
 321           | Airbus A321-200 |  5600
 733           | Boeing 737-300  |  4200
(7 rows)

demo=*# END; -- Второй терминал.
COMMIT

demo=# -- Пересечение с множеством строк.

demo=# BEGIN; -- Первый терминал.
BEGIN
demo=*# SELECT *
FROM aircrafts_tmp
WHERE range BETWEEN 4000 AND 7000
ORDER BY range DESC
FOR UPDATE;

 aircraft_code |      model      | range
---------------+-----------------+-------
 319           | Airbus A319-100 |  6700
 001           | Airbus 000-000  |  6100
 320           | Airbus A320-200 |  5700
 321           | Airbus A321-200 |  5600
 733           | Boeing 737-300  |  4200
(5 rows)

demo=# BEGIN; -- Второй терминал.
BEGIN
demo=*# SELECT *
FROM aircrafts_tmp
WHERE range > 5000
ORDER BY range DESC
FOR UPDATE;

demo=*# END; -- Первый терминал.
COMMIT

-- Второй терминал
 aircraft_code |      model      | range
---------------+-----------------+-------
 773           | Boeing 777-300  | 11100
 763           | Boeing 767-300  |  7900
 319           | Airbus A319-100 |  6700
 001           | Airbus 000-000  |  6100
 320           | Airbus A320-200 |  5700
 321           | Airbus A321-200 |  5600
(6 rows)
demo=*# END; -- Второй терминал.
COMMIT

demo=# -- Не пересекается с множеством строк.

demo=# BEGIN; -- Первый терминал.
BEGIN
demo=*# SELECT *
FROM aircrafts_tmp
WHERE range BETWEEN 4000 AND 7000
ORDER BY range DESC
FOR UPDATE;

 aircraft_code |      model      | range
---------------+-----------------+-------
 319           | Airbus A319-100 |  6700
 001           | Airbus 000-000  |  6100
 320           | Airbus A320-200 |  5700
 321           | Airbus A321-200 |  5600
 733           | Boeing 737-300  |  4200
(5 rows)

demo=# BEGIN; -- Второй терминал.
BEGIN
demo=*# SELECT *
FROM aircrafts_tmp
WHERE range < 2000
ORDER BY range DESC
FOR UPDATE;
 aircraft_code |       model        | range
---------------+--------------------+-------
 CN1           | Cessna 208 Caravan |  1200
(1 row)
demo=*# END; -- Второй терминал.
COMMIT

demo=*# END; -- Первый терминал.
COMMIT

demo=# -- При блокировании строк в режиме FOR UPDATE в текущей транзакции, другая транзакция SELECT FOR UPDATE 
demo=# -- блокируется до завершения текущей. но только в рамках работы с заблокированными строками пересекающихся множеств.
demo=# -- Если Другая транзакция SELECT FOR UPDATE оперирует со строками за пределами заблокированных и никак не пересекается с ними, то она выполняется.
```

</details>
<details>
<summary>Задание 9.9 (ISOLATION LEVEL SERIALIZABLE)</summary>
9. В разделе документации 13.2.3 «Уровень изоляции Serializable» сказано, что если поиск в таблице осуществляется последовательно, без использования индекса, тогда на всю таблицу накладывается так называемая предикатная блокировка. Такой подход приводит к увеличению числа сбоев сериализации. В качестве контрмеры можно попытаться использовать индексы. Конечно, если таблица совсем небольшая, то может и не получиться заставить PostgreSQL использовать поиск по индексу. Тем не менее давайте выполним следующий эксперимент.

Для его проведения создадим специальную таблицу, в которой будет всего два столбца: один — числовой, а второй — текстовый. Значения во втором столбце будут иметь вид: LOW1, LOW2, ..., HIGH1, HIGH2, ... Назовем эту таблицу modes.

Добавим в нее такое число строк, которое сделает очень вероятным использование индекса при выполнении операций обновления строк и, соответственно, отсутствие предикатной блокировки всей таблицы. О том, как узнать, используется ли индекс при выполнении тех или иных операций, написано в главе 10.

```sql
CREATE TABLE modes AS
SELECT num::integer, 'LOW' || num::text AS mode
FROM generate_series( 1, 100000 ) AS gen_ser( num )
UNION ALL
SELECT num::integer, 'HIGH' || ( num - 100000 )::text AS mode
FROM generate_series( 100001, 200000 ) AS gen_ser( num );
SELECT 200000
```

Проиндексируем таблицу по числовому столбцу.

```sql
CREATE INDEX modes_ind
ON modes ( num );
CREATE INDEX
```

Из всего множества строк нас будут интересовать только две:

```sql
SELECT *
FROM modes
WHERE mode IN ( 'LOW1', 'HIGH1' );

num | mode
--------+-------
1 | LOW1
100001 | HIGH1
(2 строки)
```

На первом терминале начнем транзакцию и обновим одну строку из тех двух строк, которые были показаны в предыдущем запросе.

```sql
BEGIN TRANSACTION ISOLATION LEVEL SERIALIZABLE;
BEGIN
UPDATE modes
SET mode = 'HIGH1'
WHERE num = 1;
UPDATE 1
```

На втором терминале тоже начнем транзакцию и обновим другую строку из тех двух строк, которые были показаны выше.

```sql
BEGIN TRANSACTION ISOLATION LEVEL SERIALIZABLE;
BEGIN

UPDATE modes
SET mode = 'LOW1'
WHERE num = 100001;
UPDATE 1
```

Обратите внимание, что обе команды UPDATE были выполнены, ни одна из них не ожидает завершения другой транзакции.
Попробуем завершить транзакции. Сначала — на первом терминале:

```sql
COMMIT;
COMMIT
```

А потом на втором терминале:

```sql
COMMIT;
COMMIT
```

Посмотрим, что получилось:

```sql
SELECT *
FROM modes
WHERE mode IN ( 'LOW1', 'HIGH1' );

num | mode
--------+-------
1 | HIGH1
100001 | LOW1
(2 строки)
```

Теперь система смогла сериализовать параллельные транзакции и зафиксировать их обе. Как вы думаете, почему это удалось? Обосновывая ваш ответ, примите во внимание тот результат, который был бы получен при последовательном выполнении транзакций.

**Решение 9.9:**

```sql
demo=# CREATE TABLE modes AS
SELECT num::integer, 'LOW' || num::text AS mode
FROM generate_series( 1, 100000 ) AS gen_ser( num )
UNION ALL
SELECT num::integer, 'HIGH' || ( num - 100000 )::text AS mode
FROM generate_series( 100001, 200000 ) AS gen_ser( num );
SELECT 200000
demo=# CREATE INDEX modes_ind
ON modes ( num );
CREATE INDEX
demo=# SELECT *
FROM modes
WHERE mode IN ( 'LOW1', 'HIGH1' );
  num   | mode
--------+-------
      1 | LOW1
 100001 | HIGH1
(2 rows)

demo=# BEGIN TRANSACTION ISOLATION LEVEL SERIALIZABLE; -- Первый теминал.
BEGIN
demo=*# UPDATE modes
SET mode = 'HIGH1'
WHERE num = 1;
UPDATE 1

demo=# BEGIN TRANSACTION ISOLATION LEVEL SERIALIZABLE; -- Второй терминал.
BEGIN
demo=*# UPDATE modes
SET mode = 'LOW1'
WHERE num = 100001;
UPDATE 1

demo=*# END; -- Первый терминал.
COMMIT

demo=*# END; - Второй терминал.
COMMIT

demo=# SELECT *
FROM modes
WHERE mode IN ( 'LOW1', 'HIGH1' );
  num   | mode
--------+-------
      1 | HIGH1
 100001 | LOW1
(2 rows)

demo=# -- Параллельные транзакции сериализованы, т.к. другой порядок выполнения транзакций дает тот же результат.
demo=# -- Не смотря на то, что каждая транзакция работает со своим снимком данных, сканирование индекса в данных транзакциях дает гарантию,
demo=# -- что при обновлении в параллельных транзакциях читаются и обновляются разные строки и предикатная блокировка
demo=# -- не отменяет следующую после первой по очередности транзакцию. */

demo=*# EXPLAIN UPDATE modes
SET mode = 'HIGH1'
WHERE num = 1;
                                  QUERY PLAN
------------------------------------------------------------------------------
 Update on modes  (cost=0.42..8.44 rows=0 width=0)
   ->  Index Scan using modes_ind on modes  (cost=0.42..8.44 rows=1 width=38)
         Index Cond: (num = 1)
(3 rows)
```

</details>
<details>
<summary>Задание 9.10 (ISOLATION LEVEL SERIALIZABLE)</summary>
10. В тексте главы был рассмотрен пример транзакции над таблицами базы данных «Авиаперевозки». Давайте теперь создадим две параллельные транзакции и выполним их с уровнем изоляции Serializable. Отправим также двоих пассажиров теми же самыми рейсами, что и ранее, но операции распределим между двумя транзакциями. Отличие заключается в том, что в начале транзакции будут выполняться выборки из таблицы ticket_flights. Для упрощения ситуации не будем предварительно проверять наличие свободных мест, т. к. сейчас для нас важно не это. Итак, первая транзакция:

```sql
BEGIN TRANSACTION ISOLATION LEVEL SERIALIZABLE;
BEGIN
SELECT *
FROM ticket_flights
WHERE flight_id = 13881;

ticket_no | flight_id | fare_conditions | amount
---------------+-----------+-----------------+----------
0005433848165 | 13881 | Business | 99800.00
...
0005433848007 | 13881 | Economy | 33300.00
(82 строки)

INSERT INTO bookings ( book_ref, book_date, total_amount )
VALUES ( 'ABC123', bookings.now(), 0 );
INSERT 0 1

INSERT INTO tickets
( ticket_no, book_ref, passenger_id, passenger_name )
VALUES ( '9991234567890', 'ABC123', '1234 123456', 'IVAN PETROV' );
INSERT 0 1

INSERT INTO ticket_flights
( ticket_no, flight_id, fare_conditions, amount )
VALUES ( '9991234567890', 13881, 'Business', 12500 );
INSERT 0 1

UPDATE bookings
SET total_amount = 12500
WHERE book_ref = 'ABC123';
UPDATE 1

COMMIT;
COMMIT
```

Вторая транзакция:

```sql
BEGIN TRANSACTION ISOLATION LEVEL SERIALIZABLE;
BEGIN
SELECT *
FROM ticket_flights
WHERE flight_id = 5572;

ticket_no | flight_id | fare_conditions | amount
---------------+-----------+-----------------+----------
0005433847924 | 5572 | Business | 99800.00
...
0005433847890 | 5572 | Economy | 33300.00
(100 строк)

INSERT INTO bookings ( book_ref, book_date, total_amount )
VALUES ( 'ABC456', bookings.now(), 0 );
INSERT 0 1

INSERT INTO tickets
( ticket_no, book_ref, passenger_id, passenger_name )
VALUES ( '9991234567891', 'ABC456', '4321 654321', 'PETR IVANOV' );
INSERT 0 1

INSERT INTO ticket_flights
( ticket_no, flight_id, fare_conditions, amount )
VALUES ( '9991234567891', 5572, 'Business', 12500 );
INSERT 0 1

UPDATE bookings
SET total_amount = 12500
WHERE book_ref = 'ABC456';
UPDATE 1

COMMIT;
ОШИБКА: не удалось сериализовать доступ из-за зависимостей
чтения/записи между транзакциями
ПОДРОБНОСТИ: Reason code: Canceled on identification as a pivot,
during commit attempt.
ПОДСКАЗКА: Транзакция может завершиться успешно при следующей
попытке.
```

Задание 1. Попытайтесь объяснить, почему транзакции не удалось сериализовать. Что можно сделать, чтобы удалось зафиксировать обе транзакции? Одно из возможных решений — понизить уровень изоляции. Другим решением может быть создание индекса по столбцу flight_id для таблицы ticket_flights. Почему создание индекса может помочь? Обратитесь за разъяснениями к разделу документации 13.2.3 «Уровень изоляции Serializable».

Задание 2. В первой транзакции условие в команде SELECT такое: ... WHERE flight_id = 13881. В команде вставки в таблицу ticket_flights значение поля flight_id также равно 13881. Во второй транзакции в этих же командах используется значение 5572. Поменяйте местами значения в командах SELECT и повторите эксперименты, выполнив транзакции параллельно с уровнем изоляции Serializable. Почему сейчас наличие индекса не помогает зафиксировать обе транзакции? Вспомните, что аномалия сериализации — это ситуация, когда параллельное выполнение транзакций приводит к результату, невозможному ни при каком из вариантов упорядочения этих же транзакций при их последовательном выполнении.

**Решение 9.10:**

```sql
demo=# BEGIN TRANSACTION ISOLATION LEVEL SERIALIZABLE; -- Первый терминал.
BEGIN
demo=*# SELECT *
FROM ticket_flights
WHERE flight_id = 13881;

   ticket_no   | flight_id | fare_conditions | amount
---------------+-----------+-----------------+---------
 0005435082189 |     13881 | Economy         | 7400.00
 0005435082191 |     13881 | Economy         | 8200.00
 0005435082192 |     13881 | Economy         | 7400.00
 0005435082194 |     13881 | Economy         | 7400.00
 0005435082188 |     13881 | Economy         | 7400.00
 0005435790093 |     13881 | Economy         | 7400.00
 0005435082190 |     13881 | Economy         | 7400.00
 0005435790094 |     13881 | Economy         | 7400.00
 0005435082195 |     13881 | Economy         | 7400.00
 0005435082193 |     13881 | Economy         | 7400.00
 0005435790092 |     13881 | Economy         | 7400.00
(11 rows)

demo=*# INSERT INTO bookings ( book_ref, book_date, total_amount )
VALUES ( 'ABC123', bookings.now(), 0 );
INSERT 0 1

demo=*# INSERT INTO tickets
( ticket_no, book_ref, passenger_id, passenger_name )
VALUES ( '9991234567890', 'ABC123', '1234 123456', 'IVAN PETROV' );
INSERT 0 1

demo=*# INSERT INTO ticket_flights
( ticket_no, flight_id, fare_conditions, amount )
VALUES ( '9991234567890', 13881, 'Business', 12500 );
INSERT 0 1

demo=*# UPDATE bookings
SET total_amount = 12500
WHERE book_ref = 'ABC123';
UPDATE 1

demo=# BEGIN TRANSACTION ISOLATION LEVEL SERIALIZABLE; -- Второй терминал.
BEGIN
demo=*# SELECT *
FROM ticket_flights
WHERE flight_id = 5572;
   ticket_no   | flight_id | fare_conditions |  amount
---------------+-----------+-----------------+----------
 0005432222921 |      5572 | Business        | 33000.00
 0005433681813 |      5572 | Business        | 33000.00
 0005433681806 |      5572 | Business        | 33000.00
 0005432967371 |      5572 | Business        | 33000.00
...
demo=*# INSERT INTO bookings ( book_ref, book_date, total_amount )
VALUES ( 'ABC456', bookings.now(), 0 );
INSERT 0 1

demo=*# INSERT INTO tickets
( ticket_no, book_ref, passenger_id, passenger_name )
VALUES ( '9991234567891', 'ABC456', '4321 654321', 'PETR IVANOV' );
INSERT 0 1

demo=*# INSERT INTO ticket_flights
( ticket_no, flight_id, fare_conditions, amount )
VALUES ( '9991234567891', 5572, 'Business', 12500 );
INSERT 0 1

demo=*# UPDATE bookings
SET total_amount = 12500
WHERE book_ref = 'ABC456';
UPDATE 1

demo=*# END;  -- Первый терминал.
COMMIT

demo=*# END;  -- Второй терминал.
ERROR:  could not serialize access due to read/write dependencies among transactions
DETAIL:  Reason code: Canceled on identification as a pivot, during commit attempt.
HINT:  The transaction might succeed if retried.

demo=# -- Задание 1
demo=# -- Последовательное сканирование flight_id в таблице ticket_flights ведет 
demo=# -- к предикатной блокировке на уровне изоляции транзакций Serializable. 
demo=# -- Чтобы его избежать, оставаясь на уровне сериализации, нужно заменить последовательное
demo=# -- сканирование на сканирование индекса. Создадим индекс и повторим транзакции снова. 
demo=# -- Таблица ticket_flights теперь сканируется по индексу 
demo=# -- и параллельные транзакции удается сериализовать.

demo=# CREATE INDEX ON ticket_flights (flight_id);
CREATE INDEX

demo=# EXPLAIN SELECT *
FROM ticket_flights
WHERE flight_id = 5572;
                                              QUERY PLAN
-------------------------------------------------------------------------------------------------------
 Index Scan using ticket_flights_flight_id_idx on ticket_flights  (cost=0.43..322.92 rows=83 width=32)
   Index Cond: (flight_id = 5572)
(2 rows)

demo=# EXPLAIN SELECT *
FROM ticket_flights
WHERE flight_id = 13881;
                                              QUERY PLAN
-------------------------------------------------------------------------------------------------------
 Index Scan using ticket_flights_flight_id_idx on ticket_flights  (cost=0.43..322.92 rows=83 width=32)
   Index Cond: (flight_id = 13881)
(2 rows)

demo=# BEGIN TRANSACTION ISOLATION LEVEL SERIALIZABLE; -- Первый терминал.
BEGIN
demo=*# SELECT *
FROM ticket_flights
WHERE flight_id = 13881;

   ticket_no   | flight_id | fare_conditions | amount
---------------+-----------+-----------------+---------
 0005435082191 |     13881 | Economy         | 8200.00
 0005435082194 |     13881 | Economy         | 7400.00
 0005435082188 |     13881 | Economy         | 7400.00
 0005435790093 |     13881 | Economy         | 7400.00
 0005435082192 |     13881 | Economy         | 7400.00
 0005435082189 |     13881 | Economy         | 7400.00
 0005435082190 |     13881 | Economy         | 7400.00
 0005435082195 |     13881 | Economy         | 7400.00
 0005435790094 |     13881 | Economy         | 7400.00
 0005435082193 |     13881 | Economy         | 7400.00
 0005435790092 |     13881 | Economy         | 7400.00
(11 rows)

demo=*# INSERT INTO bookings ( book_ref, book_date, total_amount )
VALUES ( 'ABC123', bookings.now(), 0 );
INSERT 0 1

demo=*# INSERT INTO tickets
( ticket_no, book_ref, passenger_id, passenger_name )
VALUES ( '9991234567890', 'ABC123', '1234 123456', 'IVAN PETROV' );
INSERT 0 1

demo=*# INSERT INTO ticket_flights
( ticket_no, flight_id, fare_conditions, amount )
VALUES ( '9991234567890', 13881, 'Business', 12500 );
INSERT 0 1

demo=*# UPDATE bookings
SET total_amount = 12500
WHERE book_ref = 'ABC123';
UPDATE 1

demo=# BEGIN TRANSACTION ISOLATION LEVEL SERIALIZABLE; -- Второй терминал.
BEGIN
demo=*# SELECT *
FROM ticket_flights
WHERE flight_id = 5572;

   ticket_no   | flight_id | fare_conditions |  amount
---------------+-----------+-----------------+----------
 0005432222921 |      5572 | Business        | 33000.00
 0005433681806 |      5572 | Business        | 33000.00
...
 0005432967475 |      5572 | Economy         | 11000.00
 0005433681831 |      5572 | Economy         | 11000.00
(69 rows)

demo=*# INSERT INTO bookings ( book_ref, book_date, total_amount )
VALUES ( 'ABC456', bookings.now(), 0 );
INSERT 0 1

demo=*# INSERT INTO tickets
( ticket_no, book_ref, passenger_id, passenger_name )
VALUES ( '9991234567891', 'ABC456', '4321 654321', 'PETR IVANOV' );
INSERT 0 1

demo=*# INSERT INTO ticket_flights
( ticket_no, flight_id, fare_conditions, amount )
VALUES ( '9991234567891', 5572, 'Business', 12500 );
INSERT 0 1

demo=*# UPDATE bookings
SET total_amount = 12500
WHERE book_ref = 'ABC456';
UPDATE 1

demo=*# END; -- Первый терминал.
COMMIT

demo=*# END; -- Второй терминал.
COMMIT

demo=# -- Задание 2
demo=# -- Поменяем местами значения flight_id в командах SELECT.  
demo=# -- Транзакции не удается сериализовать. Наличие сканирования по индексу, 
demo=# -- как условия избегания предикатной блокировки и сбоя сериализации все так же выполняется, 
demo=# -- но нарушается другое условие успешной сериализации - равенство результатов транзакции
demo=# -- при одновременном и при разных вариантах последовательного их выполнения. 
demo=# -- Получается так, что при одновременной фиксации транзакций в одной из них в таблицу
demo=# -- ticket_flights записывается одно значение flight_id и таблица меняется, 
demo=# -- а в другой транзакции строится отношение из выборки по этому значению, 
demo=# -- но из снимка данных без изменений. И запрос SELECT во второй транзакции дает результат,
demo=# -- отличный от того, в котором вставка данных была бы зафиксирована и запрос строился бы 
demo=# -- с учетом этих изменений. Та же самая ситуация получается и при обратном последовательном
demo=# -- выполнении транзакции. Результат запроса SELECT будет отличаться от первого варианта
demo=# -- последовательности выполнения транзакций.

demo=# BEGIN TRANSACTION ISOLATION LEVEL SERIALIZABLE; -- Первый терминал.
BEGIN
demo=*# SELECT *
FROM ticket_flights
WHERE flight_id = 5572;

   ticket_no   | flight_id | fare_conditions |  amount
---------------+-----------+-----------------+----------
 0005432222921 |      5572 | Business        | 33000.00
 0005433681806 |      5572 | Business        | 33000.00
...
 0005432967475 |      5572 | Economy         | 11000.00
 0005433681831 |      5572 | Economy         | 11000.00
(69 rows)

demo=*# INSERT INTO bookings ( book_ref, book_date, total_amount )
VALUES ( 'ABC123', bookings.now(), 0 );
INSERT 0 1

demo=*# INSERT INTO tickets
( ticket_no, book_ref, passenger_id, passenger_name )
VALUES ( '9991234567890', 'ABC123', '1234 123456', 'IVAN PETROV' );
INSERT 0 1

demo=*# INSERT INTO ticket_flights
( ticket_no, flight_id, fare_conditions, amount )
VALUES ( '9991234567890', 13881, 'Business', 12500 );
INSERT 0 1

demo=*# UPDATE bookings
SET total_amount = 12500
WHERE book_ref = 'ABC123';
UPDATE 1

demo=# BEGIN TRANSACTION ISOLATION LEVEL SERIALIZABLE;  -- Второй терминал.
BEGIN
demo=*# SELECT *
FROM ticket_flights
WHERE flight_id = 13881;

   ticket_no   | flight_id | fare_conditions | amount
---------------+-----------+-----------------+---------
 0005435082191 |     13881 | Economy         | 8200.00
 0005435082194 |     13881 | Economy         | 7400.00
 0005435082188 |     13881 | Economy         | 7400.00
 0005435790093 |     13881 | Economy         | 7400.00
 0005435082192 |     13881 | Economy         | 7400.00
 0005435082189 |     13881 | Economy         | 7400.00
 0005435082190 |     13881 | Economy         | 7400.00
 0005435082195 |     13881 | Economy         | 7400.00
 0005435790094 |     13881 | Economy         | 7400.00
 0005435082193 |     13881 | Economy         | 7400.00
 0005435790092 |     13881 | Economy         | 7400.00
(11 rows)

demo=*# INSERT INTO bookings ( book_ref, book_date, total_amount )
VALUES ( 'ABC456', bookings.now(), 0 );
INSERT 0 1

demo=*# INSERT INTO tickets
( ticket_no, book_ref, passenger_id, passenger_name )
VALUES ( '9991234567891', 'ABC456', '4321 654321', 'PETR IVANOV' );
INSERT 0 1

demo=*# INSERT INTO ticket_flights
( ticket_no, flight_id, fare_conditions, amount )
VALUES ( '9991234567891', 5572, 'Business', 12500 );
INSERT 0 1

demo=*# UPDATE bookings
SET total_amount = 12500
WHERE book_ref = 'ABC456';
UPDATE 1

demo=*# END;
ERROR:  could not serialize access due to read/write dependencies among transactions
DETAIL:  Reason code: Canceled on identification as a pivot, during commit attempt.
HINT:  The transaction might succeed if retried.

demo=*# END;  -- Первый терминал.
COMMIT
demo=*# END;  -- Второй терминал.
ERROR:  could not serialize access due to read/write dependencies among transactions
DETAIL:  Reason code: Canceled on identification as a pivot, during commit attempt.
HINT:  The transaction might succeed if retried.
```

</details>

## Глава 10. Повышение производительности
<details>
<summary>Задание 10.3 (EXPLAIN, CTE)</summary>
3. Самостоятельно выполните команду EXPLAIN для запроса, содержащего общее табличное выражение (CTE). Посмотрите, на каком уровне находится узел плана, отвечающий за это выражение, как он оформляется. Учтите, что общие табличные выражения всегда материализуются, т. е. вычисляются однократно и результат их вычисления сохраняется в памяти, а затем все последующие обращения в рамках запроса направляются уже к этому материализованному результату.

**Решение 10.3:**

```sql
EXPLAIN WITH recent_b AS
(SELECT book_ref, book_date,
EXTRACT(YEAR FROM book_date) AS year,
EXTRACT(MONTH FROM book_date) AS month,
rank() OVER (
PARTITION BY EXTRACT(YEAR FROM book_date)
ORDER BY EXTRACT(MONTH FROM book_date) DESC
) AS rank
FROM bookings
)
SELECT rb.book_date, tf.fare_conditions, t.passenger_name, 
t.contact_data->'phone' AS phone
FROM tickets AS t
JOIN ticket_flights AS tf ON t.ticket_no = tf.ticket_no
JOIN recent_b AS rb ON t.book_ref = rb.book_ref
WHERE tf.fare_conditions = 'Comfort' AND rb.rank = 1;

                                                            QUERY PLAN
----------------------------------------------------------------------------------------------------------------------------------
 Nested Loop  (cost=118248.19..146279.20 rows=183 width=64)
   ->  Hash Join  (cost=118247.76..143578.94 rows=4145 width=93)
         Hash Cond: (t.book_ref = rb.book_ref)
         ->  Seq Scan on tickets t  (cost=0.00..22180.71 rows=829071 width=92)
         ->  Hash  (cost=118210.68..118210.68 rows=2967 width=15)
               ->  Subquery Scan on rb  (cost=95956.94..118210.68 rows=2967 width=15)
                     Filter: (rb.rank = 1)
                     ->  WindowAgg  (cost=95956.94..110792.76 rows=593433 width=87)
                           ->  Sort  (cost=95956.94..97440.52 rows=593433 width=79)
                                 Sort Key: (EXTRACT(year FROM bookings.book_date)), (EXTRACT(month FROM bookings.book_date)) DESC
                                 ->  Seq Scan on bookings  (cost=0.00..12681.49 rows=593433 width=79)
   ->  Index Scan using ticket_flights_pkey on ticket_flights tf  (cost=0.43..0.64 rows=1 width=22)
         Index Cond: (ticket_no = t.ticket_no)
         Filter: ((fare_conditions)::text = 'Comfort'::text)
 JIT:
   Functions: 23
   Options: Inlining false, Optimization false, Expressions true, Deforming true
(17 rows)

-- Сначала в нижнем узле сканируется по индексу таблица ticket_flights и отбираются строки с Comfort. 
-- Затем в другом крупном узле  Hash Join, где CTE и объединения двух таблиц : в подузле WindowAgg
-- вычисляется оконная функция с сортировками по разделам. На следующем подузле Subquery Scan
-- возвращаются отфильтрованные по условию результаты оконной функции.
-- Затем таблица tickets и временная материализованная recent_b из CTE объединяются 
-- через хеширование Hash Join, а потом финально с fare_conditions вложенным циклом Nested Loop.
```

</details>
<details>
<summary>Задание 10.4 (QUERY PLAN)</summary>
4. Прокомментируйте следующий план, попробуйте объяснить значения всех его узлов и параметров.

```sql
EXPLAIN
SELECT total_amount
FROM bookings
ORDER BY total_amount DESC
LIMIT 5;

QUERY PLAN
----------------------------------------------------------------
Limit (cost=8666.69..8666.71 rows=5 width=6)
-> Sort (cost=8666.69..9323.66 rows=262788 width=6)
Sort Key: total_amount DESC
-> Seq Scan on bookings (cost=0.00..4301.88 rows=262788
width=6)
(4 строки)
```

**Решение 10.4:**

```sql
EXPLAIN
SELECT total_amount
FROM bookings
ORDER BY total_amount DESC
LIMIT 5;

QUERY PLAN
----------------------------------------------------------------
Limit (cost=8666.69..8666.71 rows=5 width=6)
-> Sort (cost=8666.69..9323.66 rows=262788 width=6)
Sort Key: total_amount DESC
-> Seq Scan on bookings (cost=0.00..4301.88 rows=262788
width=6)
(4 строки)

-- Сначала на нижнем подузле последовательно сканируется таблица booking. 
-- Затем эта таблица сортируется по колонке total_amount по убыванию. 
-- Когда сортировка заканчивается, LIMIT возвращает первые пять строк из 262788.
```

</details>
<details>
<summary>Задание 10.6 (EXPLAIN, WindowAgg)</summary>
6. Выполните команду EXPLAIN для запроса, в котором использована какаянибудь из оконных функций. Найдите в плане выполнения запроса узел с именем WindowAgg. Попробуйте объяснить, почему он занимает именно этот уровень в плане.

**Решение 10.6:**

```sql
EXPLAIN WITH recent_b AS
(SELECT book_ref, book_date,
EXTRACT(YEAR FROM book_date) AS year,
EXTRACT(MONTH FROM book_date) AS month,
rank() OVER (
PARTITION BY EXTRACT(YEAR FROM book_date)
ORDER BY EXTRACT(MONTH FROM book_date) DESC
) AS rank
FROM bookings
)
SELECT rb.book_date, tf.fare_conditions, t.passenger_name, 
t.contact_data->'phone' AS phone
FROM tickets AS t
JOIN ticket_flights AS tf ON t.ticket_no = tf.ticket_no
JOIN recent_b AS rb ON t.book_ref = rb.book_ref
WHERE tf.fare_conditions = 'Comfort' AND rb.rank = 1;

                                                            QUERY PLAN
----------------------------------------------------------------------------------------------------------------------------------
 Nested Loop  (cost=118248.19..146279.20 rows=183 width=64)
   ->  Hash Join  (cost=118247.76..143578.94 rows=4145 width=93)
         Hash Cond: (t.book_ref = rb.book_ref)
         ->  Seq Scan on tickets t  (cost=0.00..22180.71 rows=829071 width=92)
         ->  Hash  (cost=118210.68..118210.68 rows=2967 width=15)
               ->  Subquery Scan on rb  (cost=95956.94..118210.68 rows=2967 width=15)
                     Filter: (rb.rank = 1)
                     ->  WindowAgg  (cost=95956.94..110792.76 rows=593433 width=87)
                           ->  Sort  (cost=95956.94..97440.52 rows=593433 width=79)
                                 Sort Key: (EXTRACT(year FROM bookings.book_date)), (EXTRACT(month FROM bookings.book_date)) DESC
                                 ->  Seq Scan on bookings  (cost=0.00..12681.49 rows=593433 width=79)
   ->  Index Scan using ticket_flights_pkey on ticket_flights tf  (cost=0.43..0.64 rows=1 width=22)
         Index Cond: (ticket_no = t.ticket_no)
         Filter: ((fare_conditions)::text = 'Comfort'::text)
 JIT:
   Functions: 23
   Options: Inlining false, Optimization false, Expressions true, Deforming true
(17 rows)

-- Запрос разделен на два больших узла: узел Hash Join как внешний для вложенного цикла  
-- Nested Loop и внутренний узел Index Scan.  Это два набора строк, которые объединяются 
-- на последнем этапе запроса. Во внутреннем узле последовательно сканируется 
-- таблица ticket_flights для отбора строк по условию = 'Comfort'. 

-- Оконная функция участвует для формирования другого набора строк во внешнем узле 
-- для отбора строк по условию rank = 1. Планировщик сначала должен сформировать материализованную 
-- таблицу оконной функции и получить колонку со значениями для фильтрации и затем 
-- после отсечения ненужных строк далее выполнять объединения. 
-- Поэтому подузел с оконной функцией находится внизу узла Hash Join.

-- В узле WindowAgg последовательно сканируется таблица bookings, 
-- из которой формируются окна, строки сортируются и далее сканируются на следующем уровне 
-- Subquery Scan для отбора по фильтру rank = 1. На верхних уровнях формируется хеш-таблица 
-- с ключами которой является значения атрибута book_ref и по нему объединяются таблицы tickets и
-- материализованная оконная функция.И на последнем этапе идет объединении вложенным циклом 
-- результатов Hash Join и выборки из ticket_flights.
```

</details>
<details>
<summary>Задание 10.7 (EXPLAIN, INSERT, DELETE)</summary>
7. Проанализируйте план выполнения операций вставки и удаления строк. Причем сделайте это таким образом, чтобы данные в таблицах фактически изменены не были.

**Решение 10.7:**

```sql
demo=# BEGIN;
BEGIN
demo=*# EXPLAIN ANALYZE
INSERT INTO nulls
VALUES ( 200001, 'TEXT200001' ),
( 200002, 'TEXT200002' ),
( 200003, 'TEXT200003' );
                                                  QUERY PLAN
--------------------------------------------------------------------------------------------------------------
 Insert on nulls  (cost=0.00..0.04 rows=0 width=0) (actual time=3.384..3.385 rows=0 loops=1)
   ->  Values Scan on "*VALUES*"  (cost=0.00..0.04 rows=3 width=36) (actual time=0.245..0.249 rows=3 loops=1)
 Planning Time: 3.116 ms
 Execution Time: 4.210 ms
(4 rows)
demo=!# ROLLBACK;
ROLLBACK

-- При вставке в данном случае в плане запроса один узел, в котором выполняется оператор VALUES,
-- стоимостью 0.04. Вставляются 3 строки по 36 бит каждая за один цикл. 
-- Оценочное время выполнения запроса - 4.210 ms

-- Вставим строки в таблицу.

demo=# INSERT INTO nulls
VALUES ( 200001, 'TEXT200001' ),
( 200002, 'TEXT200002' ),
( 200003, 'TEXT200003' );
INSERT 0 3

-- Начинать и откатывать блок транзакции не будем, чтобы строки удалились, 
-- хотя можно было начать и закоммитить транзакцию.

demo=# EXPLAIN ANALYZE
DELETE FROM nulls
WHERE num BETWEEN 200001 AND 200003;
                                                        QUERY PLAN
---------------------------------------------------------------------------------------------------------------------------
 Delete on nulls  (cost=0.42..8.48 rows=0 width=0) (actual time=0.175..0.176 rows=0 loops=1)
   ->  Index Scan using nulls_num_idx on nulls  (cost=0.42..8.48 rows=3 width=6) (actual time=0.008..0.012 rows=3 loops=1)
         Index Cond: ((num >= 200001) AND (num <= 200003))
 Planning Time: 0.126 ms
 Execution Time: 0.193 ms
(5 rows)

-- Операция удаления более сложная, поскольку нужно выполнить условие в WHERE. 
-- В таблице есть индекс для num и он используется.  
-- Для поиска первой строки в индексе использовано 0.42 единицы ресурсов. 
-- Удаление трех строк по 6 байт за один цикл оценено планировщиком в 8.48 единиц. 
-- Оценочное время выполнения запроса - 0.193 ms
```

</details>
<details>
<summary>Задание 10.8 (корр. подзапрос, JOIN)</summary>
8. Замена коррелированного подзапроса соединением таблиц является одним из способов повышения производительности.

Предположим, что мы задались вопросом: сколько маршрутов обслуживают самолеты каждого типа? При этом нужно учитывать, что может иметь место такая ситуация, когда самолеты какого-либо типа не обслуживают ни одного маршрута. Поэтому необходимо использовать не только представление «Маршруты» (routes), но и таблицу «Самолеты» (aircrafts).

Это первый вариант запроса, в нем используется коррелированный подзапрос.

```sql
EXPLAIN ANALYZE
SELECT a.aircraft_code AS a_code,
a.model,
( SELECT count( r.aircraft_code )
FROM routes r
WHERE r.aircraft_code = a.aircraft_code
) AS num_routes
FROM aircrafts a
GROUP BY 1, 2
ORDER BY 3 DESC;
```

А в этом варианте коррелированный подзапрос раскрыт и заменен внешним
соединением:

```sql
EXPLAIN ANALYZE
SELECT a.aircraft_code AS a_code,
a.model,
count( r.aircraft_code ) AS num_routes
FROM aircrafts a
LEFT OUTER JOIN routes r
ON r.aircraft_code = a.aircraft_code
GROUP BY 1, 2
ORDER BY 3 DESC;
```

Причина использования внешнего соединения в том, что может найтись модель самолета, не обслуживающая ни одного маршрута, и если не использовать внешнее соединение, она вообще не попадет в результирующую выборку.

Исследуйте планы выполнения обоих запросов. Попытайтесь найти объяснение различиям в эффективности их выполнения. Чтобы получить усредненную картину, выполните каждый запрос несколько раз. Поскольку таблицы, участвующие в запросах, небольшие, то различие по абсолютным затратам времени выполнения будет незначительным. Но если бы число строк в таблицах было большим, то экономия ресурсов сервера могла оказаться заметной.

Предложите аналогичную пару запросов к базе данных «Авиаперевозки». Проведите необходимые эксперименты с вашими запросами.

**Решение 10.8:**

```sql
demo=# EXPLAIN ANALYZE
SELECT a.aircraft_code AS a_code,
a.model,
( SELECT count( r.aircraft_code )
FROM routes r
WHERE r.aircraft_code = a.aircraft_code
) AS num_routes
FROM aircrafts a
GROUP BY 1, 2
ORDER BY 3 DESC;
                                                       QUERY PLAN
-------------------------------------------------------------------------------------------------------------------------
 Sort  (cost=236.31..236.34 rows=9 width=28) (actual time=1.095..1.097 rows=9 loops=1)
   Sort Key: ((SubPlan 1)) DESC
   Sort Method: quicksort  Memory: 25kB
   ->  HashAggregate  (cost=1.11..236.17 rows=9 width=28) (actual time=0.220..1.062 rows=9 loops=1)
         Group Key: a.aircraft_code
         Batches: 1  Memory Usage: 24kB
         ->  Seq Scan on aircrafts a  (cost=0.00..1.09 rows=9 width=20) (actual time=0.007..0.008 rows=9 loops=1)
         SubPlan 1
           ->  Aggregate  (cost=26.10..26.11 rows=1 width=8) (actual time=0.115..0.115 rows=1 loops=9)
                 ->  Seq Scan on routes r  (cost=0.00..25.88 rows=89 width=4) (actual time=0.022..0.105 rows=79 loops=9)
                       Filter: (aircraft_code = a.aircraft_code)
                       Rows Removed by Filter: 631
 Planning Time: 0.169 ms
 Execution Time: 1.243 ms
(14 rows)

demo=# EXPLAIN ANALYZE
SELECT a.aircraft_code AS a_code,
a.model,
count( r.aircraft_code ) AS num_routes
FROM aircrafts a
LEFT OUTER JOIN routes r
ON r.aircraft_code = a.aircraft_code
GROUP BY 1, 2
ORDER BY 3 DESC;
                                                          QUERY PLAN
------------------------------------------------------------------------------------------------------------------------------
 Sort  (cost=31.83..31.85 rows=9 width=28) (actual time=0.622..0.624 rows=9 loops=1)
   Sort Key: (count(r.aircraft_code)) DESC
   Sort Method: quicksort  Memory: 25kB
   ->  HashAggregate  (cost=31.60..31.69 rows=9 width=28) (actual time=0.613..0.616 rows=9 loops=1)
         Group Key: a.aircraft_code
         Batches: 1  Memory Usage: 24kB
         ->  Hash Right Join  (cost=1.20..28.05 rows=710 width=24) (actual time=0.049..0.453 rows=711 loops=1)
               Hash Cond: (r.aircraft_code = a.aircraft_code)
               ->  Seq Scan on routes r  (cost=0.00..24.10 rows=710 width=4) (actual time=0.003..0.114 rows=710 loops=1)
               ->  Hash  (cost=1.09..1.09 rows=9 width=20) (actual time=0.013..0.013 rows=9 loops=1)
                     Buckets: 1024  Batches: 1  Memory Usage: 9kB
                     ->  Seq Scan on aircrafts a  (cost=0.00..1.09 rows=9 width=20) (actual time=0.006..0.008 rows=9 loops=1)
 Planning Time: 0.151 ms
 Execution Time: 0.716 ms
(14 rows)

-- Запросы сильно различаются ресурсами, которые тратятся на операцию HashAggregate. 
-- Она складывает в хэш все строки и возвращает их для каждого значения ключа Group Key: 
-- a.aircraft_code, расcчитывая агрегат count. Получается, что различие в затраченных ресурсах 
-- на HashAggregate зависит от того, насколько затратно HashAggregate получила строки для включения 
-- их в хэш. Для плана с коррелированной агрегатной функции это 9 циклов выполнения SubPlan 1, 
-- где 9 раз выполняется последовательное сканирование с отфильтровыванием и расчет агрегатной функции
-- для 79*9=711 строк . Для JOIN это 711 строк, полученные один раз через слияние Hash Right Join.


-- Задача: сколько пассажиромест содержат модели самолетов?

-- Коррелированный подзапрос.

demo=# EXPLAIN ANALYZE
SELECT model,
(SELECT count(seat_no)
FROM seats AS s
WHERE s.aircraft_code=a.aircraft_code
) AS seats
FROM aircrafts AS a
ORDER BY 2 DESC;
                                                                   QUERY PLAN
-------------------------------------------------------------------------------------------------------------------------------------------------
 Sort  (cost=66.64..66.66 rows=9 width=24) (actual time=0.349..0.351 rows=9 loops=1)
   Sort Key: ((SubPlan 1)) DESC
   Sort Method: quicksort  Memory: 25kB
   ->  Seq Scan on aircrafts a  (cost=0.00..66.50 rows=9 width=24) (actual time=0.116..0.343 rows=9 loops=1)
         SubPlan 1
           ->  Aggregate  (cost=7.26..7.27 rows=1 width=8) (actual time=0.036..0.036 rows=1 loops=9)
                 ->  Index Only Scan using seats_pkey on seats s  (cost=0.28..6.88 rows=149 width=3) (actual time=0.010..0.026 rows=149 loops=9)
                       Index Cond: (aircraft_code = a.aircraft_code)
                       Heap Fetches: 0
 Planning Time: 0.099 ms
 Execution Time: 0.379 ms
(11 rows)

-- Запрос с JOIN

demo=# EXPLAIN ANALYZE
SELECT model, count(seat_no) AS seats
FROM seats AS s
JOIN aircrafts AS a
ON s.aircraft_code = a.aircraft_code
GROUP BY 1
ORDER BY 2 DESC;
                                                          QUERY PLAN
------------------------------------------------------------------------------------------------------------------------------
 Sort  (cost=34.69..34.72 rows=9 width=24) (actual time=0.759..0.761 rows=9 loops=1)
   Sort Key: (count(s.seat_no)) DESC
   Sort Method: quicksort  Memory: 25kB
   ->  HashAggregate  (cost=34.46..34.55 rows=9 width=24) (actual time=0.750..0.753 rows=9 loops=1)
         Group Key: a.model
         Batches: 1  Memory Usage: 24kB
         ->  Hash Join  (cost=1.20..27.77 rows=1339 width=19) (actual time=0.023..0.482 rows=1339 loops=1)
               Hash Cond: (s.aircraft_code = a.aircraft_code)
               ->  Seq Scan on seats s  (cost=0.00..21.39 rows=1339 width=7) (actual time=0.006..0.105 rows=1339 loops=1)
               ->  Hash  (cost=1.09..1.09 rows=9 width=20) (actual time=0.010..0.011 rows=9 loops=1)
                     Buckets: 1024  Batches: 1  Memory Usage: 9kB
                     ->  Seq Scan on aircrafts a  (cost=0.00..1.09 rows=9 width=20) (actual time=0.004..0.006 rows=9 loops=1)
 Planning Time: 0.201 ms
 Execution Time: 0.849 ms
(14 rows)

-- В данном случае коррелированный подзапрос  выполнился быстрее JOIN.
-- В корр. подзапросе  SubPlan 1 сканировался только индекс без обращения к таблице 
-- и он выполнялся хоть и 9 раз, но быстро. В JOIN-запросе последовательно сканировались обе таблицы, <details>
-- в соединении хэшированием шла работа с хэш-таблицей, 
-- оператор  GROUP BY тоже потребовал работы с хэшем.
```

</details>
<details>
<summary>Задание 10.9 (EXPLAINE, MATERIALIZED VIEW)</summary>
9. Одним из способов повышения производительности является изменение схемы данных, связанное с денормализацией, а именно: создание материализованных представлений. В главе 5 было описано такое материализованное представление — «Маршруты» (routes). Команда для его создания была приведена в главе 6.

Проведите эксперимент: сначала выполните выборку из готового представления, а затем ту выборку, которая это представление формирует.

```sql
EXPLAIN ANALYZE
SELECT * FROM routes;

EXPLAIN ANALYZE
WITH f3 AS ( SELECT f2.flight_no, ...
```

Поскольку второй запрос очень громоздкий, то можно поступить таким образом: сначала сохраните его в текстовом файле, а затем выполните с помощью команды \i утилиты psql.

Вы увидите, что затраты времени отличаются практически на два порядка. Конечно, нужно помнить, что материализованные представления необходимо периодически обновлять, чтобы их содержимое было актуальным.

**Решение 10.9:**

```sql
demo=# EXPLAIN ANALYZE
SELECT * FROM routes;
                                              QUERY PLAN
-------------------------------------------------------------------------------------------------------
 Seq Scan on routes  (cost=0.00..24.10 rows=710 width=147) (actual time=0.447..2.244 rows=710 loops=1)
 Planning Time: 16.348 ms
 Execution Time: 3.798 ms
(3 rows)

demo=# \i '~/14/main/exercise9_9.sql'
                                                                                                                        QUERY PLAN

-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
 Hash Join  (cost=4746.80..5294.46 rows=491 width=135) (actual time=126.967..129.447 rows=710 loops=1)
   Hash Cond: (flights.arrival_airport = arr.airport_code)
   ->  Hash Join  (cost=4742.46..5287.59 rows=944 width=101) (actual time=126.168..128.412 rows=710 loops=1)
         Hash Cond: (flights.departure_airport = dep.airport_code)
         ->  GroupAggregate  (cost=4738.12..5260.22 rows=1816 width=67) (actual time=126.096..128.073 rows=710 loops=1)
               Group Key: flights.flight_no, flights.departure_airport, flights.arrival_airport, flights.aircraft_code, ((flights.scheduled_arrival - flights.scheduled_departure))
               ->  Sort  (cost=4738.12..4783.52 rows=18160 width=39) (actual time=125.948..126.275 rows=3798 loops=1)
                     Sort Key: flights.flight_no, flights.departure_airport, flights.arrival_airport, flights.aircraft_code, ((flights.scheduled_arrival - flights.scheduled_departure)), ((to_char(flights.scheduled_departure, 'ID'::text))::integer)
                     Sort Method: quicksort  Memory: 393kB
                     ->  HashAggregate  (cost=3090.24..3453.44 rows=18160 width=39) (actual time=116.891..117.822 rows=3798 loops=1)
                           Group Key: flights.flight_no, flights.departure_airport, flights.arrival_airport, flights.aircraft_code, (flights.scheduled_arrival - flights.scheduled_departure), (to_char(flights.scheduled_departure, 'ID'::text))::integer
                           Batches: 1  Memory Usage: 1305kB
                           ->  Seq Scan on flights  (cost=0.00..2105.28 rows=65664 width=39) (actual time=1.444..81.107 rows=65664 loops=1)
         ->  Hash  (cost=3.04..3.04 rows=104 width=38) (actual time=0.054..0.059 rows=104 loops=1)
               Buckets: 1024  Batches: 1  Memory Usage: 16kB
               ->  Seq Scan on airports dep  (cost=0.00..3.04 rows=104 width=38) (actual time=0.007..0.020 rows=104 loops=1)
   ->  Hash  (cost=3.04..3.04 rows=104 width=38) (actual time=0.785..0.785 rows=104 loops=1)
         Buckets: 1024  Batches: 1  Memory Usage: 16kB
         ->  Seq Scan on airports arr  (cost=0.00..3.04 rows=104 width=38) (actual time=0.639..0.674 rows=104 loops=1)
 Planning Time: 14.614 ms
 Execution Time: 186.814 ms
(21 rows)

demo=# DROP MATERIALIZED VIEW routes_tmp;
DROP MATERIALIZED VIEW
```

</details>
<details>
<summary>Задание 10.12 (EXPLAIN, CREATE INDEX)</summary>
12. Одним из способов повышения производительности является изменение схемы данных, связанное с денормализацией, а именно: создание индексов. Выполните следующий простой запрос к таблице «Билеты»:

```sql
EXPLAIN ANALYZE
SELECT count( * )
FROM tickets
WHERE passenger_name = 'IVAN IVANOV';
```

Создайте индекс по столбцу passenger_name:

```sql
CREATE INDEX passenger_name_key
ON tickets ( passenger_name );
```

Теперь повторите запрос и сравните полученные планы и фактические результаты.

Предложите какой-нибудь запрос к базе данных «Авиаперевозки», для выполнения которого было бы целесообразно создать индекс. Создайте индекс и повторите запрос. Изучите полученный план, посмотрите, используется ли индекс планировщиком.

**Решение 10.12:**

```sql
demo=# EXPLAIN ANALYZE
SELECT count( * )
FROM tickets
WHERE passenger_name = 'IVAN IVANOV';
                                                             QUERY PLAN
-------------------------------------------------------------------------------------------------------------------------------------
 Finalize Aggregate  (cost=19208.90..19208.91 rows=1 width=8) (actual time=92.182..99.644 rows=1 loops=1)
   ->  Gather  (cost=19208.68..19208.89 rows=2 width=8) (actual time=92.000..99.631 rows=3 loops=1)
         Workers Planned: 2
         Workers Launched: 2
         ->  Partial Aggregate  (cost=18208.68..18208.69 rows=1 width=8) (actual time=70.600..70.601 rows=1 loops=3)
               ->  Parallel Seq Scan on tickets  (cost=0.00..18208.08 rows=242 width=0) (actual time=0.363..70.490 rows=157 loops=3)
                     Filter: (passenger_name = 'IVAN IVANOV'::text)
                     Rows Removed by Filter: 276200
 Planning Time: 0.091 ms
 Execution Time: 99.677 ms
(10 rows)

Time: 100.323 ms

demo=# CREATE INDEX passenger_name_key
ON tickets ( passenger_name );
CREATE INDEX
                                                                 QUERY PLAN

--------------------------------------------------------------------------------------------------------------------------------------------
 Aggregate  (cost=20.02..20.04 rows=1 width=8) (actual time=0.094..0.095 rows=1 loops=1)
   ->  Index Only Scan using passenger_name_key on tickets  (cost=0.42..18.57 rows=580 width=0) (actual time=0.021..0.062 rows=470 loops=1)
         Index Cond: (passenger_name = 'IVAN IVANOV'::text)
         Heap Fetches: 0
 Planning Time: 0.105 ms
 Execution Time: 0.119 ms
(6 rows)

-- Первый план выполняется в параллельном режиме. Последовательное сканирование осуществляется 
-- параллельно  с расчетом агрегатной функции. После создания индекса планировщик использует 
-- только индекс и после отфильтровывания строк их суммирует.

-Подсчитаем количество клиентов с чеком свыше полутора миллионов.

demo=# SELECT count(total_amount) AS over_500k_value
FROM bookings
WHERE total_amount >= 500000;
 over_500k_value
-----------------
            2239
(1 row)

demo=# EXPLAIN ANALYZE
SELECT count(total_amount) AS over_500k_value
FROM bookings
WHERE total_amount >= 500000;
                                                              QUERY PLAN
--------------------------------------------------------------------------------------------------------------------------------------
 Finalize Aggregate  (cost=7874.30..7874.31 rows=1 width=8) (actual time=49.655..54.504 rows=1 loops=1)
   ->  Gather  (cost=7874.09..7874.30 rows=2 width=8) (actual time=49.475..54.473 rows=3 loops=1)
         Workers Planned: 2
         Workers Launched: 2
         ->  Partial Aggregate  (cost=6874.09..6874.10 rows=1 width=8) (actual time=46.559..46.565 rows=1 loops=3)
               ->  Parallel Seq Scan on bookings  (cost=0.00..6870.80 rows=1317 width=6) (actual time=0.060..46.430 rows=746 loops=3)
                     Filter: (total_amount >= '500000'::numeric)
                     Rows Removed by Filter: 197065
 Planning Time: 0.113 ms
 Execution Time: 54.564 ms
(10 rows)

-- Планировщик использовал параллельное сканирование  bookings и параллельно подсчитывалась сумма строк 
-- по условию >= '500000'. Узел Gather находится вверху плана и вся часть ниже выполняется 
-- в параллельном режиме. Сборка данных осуществляется в произвольном порядке без сортировки.  
-- Создадим индекс для total_amount.

demo=# CREATE INDEX ON bookings (total_amount);
CREATE INDEX

demo=# EXPLAIN ANALYZE
SELECT count(total_amount) AS over_500k_value
FROM bookings
WHERE total_amount >= 500000;

                                                                      QUERY PLAN

-------------------------------------------------------------------------------------------------------------------------------------------------------
 Aggregate  (cost=111.13..111.14 rows=1 width=8) (actual time=0.507..0.507 rows=1 loops=1)
   ->  Index Only Scan using bookings_total_amount_idx on bookings  (cost=0.42..102.79 rows=3335 width=6) (actual time=0.027..0.363 rows=2239 loops=1)
         Index Cond: (total_amount >= '500000'::numeric)
         Heap Fetches: 0
 Planning Time: 0.152 ms
 Execution Time: 0.530 ms
(6 rows)

-- Теперь планировщик использует сканирование исключительно индекса и выполняет агрегатную функцию 
-- на самом верхнем уровне запроса. Время выполнения запроса существенно снизилось за счет индекса.
```

</details>
<details>
<summary>Задание 10.14 (EXPLAIN, NULL)</summary>
14. В столбцах таблиц могут содержаться значения NULL. При сортировке строк по значениям таких столбцов СУБД по умолчанию ведет себя так, как будто значение NULL превосходит по величине любые другие значения. В результате получается, что если задан возрастающий порядок сортировки, то значения NULL будут идти последними, если же порядок сортировки убывающий, тогда они будут первыми. Принимая решение о создании индексов, нужно учитывать требуемый порядок сортировки и желаемое расположение строк со значениями NULL в выборке (в ее начале или в конце).

Давайте создадим таблицу, содержащую такое число строк, что использование индекса планировщиком становится очень вероятным, и выполним с этой таблицей ряд экспериментов.

```sql
CREATE TABLE nulls AS
SELECT num::integer, 'TEXT' || num::text AS txt
FROM generate_series( 1, 200000 ) AS gen_ser( num );
SELECT 200000
```

Проиндексируем таблицу по числовому столбцу. Поскольку мы не указываем порядок сортировки явным образом, то по умолчанию он будет возрастающим, как если бы мы использовали ключевое слово ASC.

```sql
CREATE INDEX nulls_ind
ON nulls ( num );
CREATE INDEX
```

Добавим в таблицу одну строку, содержащую значение NULL в индексируемом столбце:

```sql
INSERT INTO nulls
VALUES ( NULL, 'TEXT' );
INSERT 0 1
```

Теперь посмотрим, будет ли использоваться индекс в следующем запросе:

```sql
EXPLAIN
SELECT *
FROM nulls
ORDER BY num;
```

Да, индекс используется.

```sql
QUERY PLAN
---------------------------------------------------------------------
Index Scan using nulls_ind on nulls (cost=0.42..5852.42 rows=200000
width=14)
(1 строка)
```

Вы можете убедиться, что строка со значением NULL окажется в выводе самой последней. Поскольку в нашей таблице очень много строк, воспользуемся предложением OFFSET:

```sql
SELECT *
FROM nulls
ORDER BY num
OFFSET 199995;

num | txt
--------+------------
199996 | TEXT199996
199997 | TEXT199997
199998 | TEXT199998
199999 | TEXT199999
200000 | TEXT200000
| TEXT
(6 строк)
```

Модифицируем запрос, явно указав, что значения NULL должны располагаться в самом начале выборки, и посмотрим, будет ли использоваться индекс теперь.

```sgl
EXPLAIN
SELECT *
FROM nulls
ORDER BY num NULLS FIRST;
```

Теперь индекс не используется. Вместо этого выполняется последовательное сканирование таблицы и сортировка выбранных строк.

```sql
QUERY PLAN
--------------------------------------------------------------------
Sort (cost=24110.14..24610.14 rows=200000 width=14)
Sort Key: num NULLS FIRST
-> Seq Scan on nulls (cost=0.00..3081.00 rows=200000 width=14)
(3 строки)
```

Задание 1. Ниже приведена модифицированная команды выборки из таблицы
nulls. Не выполняя эту команду, попытайтесь ответить, будет ли использоваться созданный нами индекс при выполнении такой выборки, а затем проверьте вашу гипотезу на практике.

```sql
EXPLAIN
SELECT *
FROM nulls
ORDER BY num DESC NULLS FIRST;
```

Обратите внимание на ключевое слово Backward в плане выполнения запроса. Что оно означает?

Задание 2. Модифицируйте команду создания индекса таким образом, чтобы он использовался при выполнении следующей выборки:

```sql
SELECT *
FROM nulls
ORDER BY num NULLS FIRST;
```

Задание 3. Выполните аналогичные эксперименты, задавая убывающий порядок сортировки с помощью ключевого слова DESC и изменяя расположение значений NULL в выборке с помощью ключевых слов NULLS FIRST и NULLS LAST предложения ORDER BY. С помощью команды EXPLAIN ANALYZE посмотрите, каким будет фактическое время выполнения команд. За дополнительной информацией обратитесь к описанию команды CREATE INDEX, приведенному в документации.

**Решение 10.14:**

```sql
demo=# CREATE TABLE nulls AS
SELECT num::integer, 'TEXT' || num::text AS txt
FROM generate_series( 1, 200000 ) AS gen_ser( num );
SELECT 200000

demo=# CREATE INDEX nulls_ind
ON nulls ( num );
CREATE INDEX

demo=# INSERT INTO nulls
VALUES ( NULL, 'TEXT' );
INSERT 0 1

demo=# EXPLAIN
SELECT *
FROM nulls
ORDER BY num;
                                   QUERY PLAN
--------------------------------------------------------------------------------
 Index Scan using nulls_ind on nulls  (cost=0.42..6289.42 rows=200000 width=14)
(1 row)

demo=# SELECT *
FROM nulls
ORDER BY num
OFFSET 199995;
  num   |    txt
--------+------------
 199996 | TEXT199996
 199997 | TEXT199997
 199998 | TEXT199998
 199999 | TEXT199999
 200000 | TEXT200000
        | TEXT
(6 rows)

demo=# EXPLAIN
SELECT *
FROM nulls
ORDER BY num NULLS FIRST;
                             QUERY PLAN
--------------------------------------------------------------------
 Sort  (cost=24111.14..24611.14 rows=200000 width=14)
   Sort Key: num NULLS FIRST
   ->  Seq Scan on nulls  (cost=0.00..3082.00 rows=200000 width=14)
(3 rows)

-- Задание 1.
-- Индекс использоваться будет, т.к. порядок запрашиваемой сортировки в запросе (включая положение NULLS)
-- не нарушает сортировку по-умолчанию в индексе, т.е. запрашиваются уже отсортированные данные.

demo=# EXPLAIN
SELECT *
FROM nulls
ORDER BY num DESC NULLS FIRST;
                                       QUERY PLAN
-----------------------------------------------------------------------------------------
 Index Scan Backward using nulls_ind on nulls  (cost=0.42..6289.42 rows=200000 width=14)
(1 row)

-- Index Scan Backward - это версия  Index Scan с тем же функционалом, но используется для сканирования
-- по убыванию. Мы в запросе указали отсортировать по убыванию и планировщик использует версию Backward.

-- Задание 2.

demo=# DROP INDEX nulls_ind;
DROP INDEX

demo=# CREATE INDEX ON nulls (num NULLS FIRST);
CREATE INDEX

demo=# EXPLAIN
SELECT *
FROM nulls
ORDER BY num NULLS FIRST;
                                     QUERY PLAN
------------------------------------------------------------------------------------
 Index Scan using nulls_num_idx on nulls  (cost=0.42..6289.44 rows=200001 width=14)
(1 row)

- Задание 3.

-- Запрос упорядоченных по убыванию данных, включая NULLS (DESC NULLS 
-- Индекс по-умолчанию (ASC NULLS LAST).  Index Scan Backward.

demo=# DROP INDEX nulls_num_idx;
DROP INDEX
demo=# CREATE INDEX ON nulls (num);
CREATE INDEX

demo=# EXPLAIN ANALYZE SELECT *
FROM nulls
ORDER BY num DESC NULLS FIRST;

                                                                 QUERY PLAN

---------------------------------------------------------------------------------------------------------------------------------------------
 Index Scan Backward using nulls_num_idx on nulls  (cost=0.42..6289.44 rows=200001 width=14) (actual time=0.015..36.207 rows=200001 loops=1)
 Planning Time: 0.069 ms
 Execution Time: 44.257 ms
(3 rows)

-- Запрос упорядоченных по убыванию данных, но NULLS передвинуты в конец списка (DESC NULLS LAST). 
-- Индекс по-умолчанию (ASC NULLS LAST). Seq Scan.

demo=# EXPLAIN ANALYZE 
SELECT *
FROM nulls
ORDER BY num DESC NULLS LAST;
                                                     QUERY PLAN
--------------------------------------------------------------------------------------------------------------------
 Sort  (cost=24111.25..24611.25 rows=200001 width=14) (actual time=70.561..92.871 rows=200001 loops=1)
   Sort Key: num DESC NULLS LAST
   Sort Method: external merge  Disk: 4792kB
   ->  Seq Scan on nulls  (cost=0.00..3082.01 rows=200001 width=14) (actual time=0.008..15.890 rows=200001 loops=1)
 Planning Time: 0.069 ms
 Execution Time: 101.503 ms
(6 rows)

-- Запрос упорядоченных по убыванию данных, включая NULLS (DESC NULLS FIRST). 
-- В индексе NULLS передвинуты в начало (ASC NULLS LAST). Index Scan Backward.

demo=# DROP INDEX nulls_num_idx;
DROP INDEX
demo=# CREATE INDEX ON nulls (num NULLS LAST);
CREATE INDEX

demo=# EXPLAIN ANALYZE 
SELECT *
FROM nulls
ORDER BY num DESC NULLS FIRST;
                                                                 QUERY PLAN

---------------------------------------------------------------------------------------------------------------------------------------------
 Index Scan Backward using nulls_num_idx on nulls  (cost=0.42..6289.44 rows=200001 width=14) (actual time=0.015..34.727 rows=200001 loops=1)
 Planning Time: 0.073 ms
 Execution Time: 42.375 ms
(3 rows)

-- Запрос упорядоченныых по убыванию данных, но NULLS передвинуты в конец списка (DESC NULLS LAST). 
-- В индексе NULLS передвинуты в начало (ASC NULLS LAST). Seq Scan.

demo=# EXPLAIN ANALYZE
SELECT *
FROM nulls
ORDER BY num DESC NULLS LAST;
                                                     QUERY PLAN
--------------------------------------------------------------------------------------------------------------------
 Sort  (cost=24111.25..24611.25 rows=200001 width=14) (actual time=81.910..103.786 rows=200001 loops=1)
   Sort Key: num DESC NULLS LAST
   Sort Method: external merge  Disk: 4792kB
   ->  Seq Scan on nulls  (cost=0.00..3082.01 rows=200001 width=14) (actual time=0.010..16.021 rows=200001 loops=1)
 Planning Time: 0.055 ms
 Execution Time: 112.271 ms
(6 rows)

- ORDER BY и индекс по-умолчанию (ASC NULLS LAST)

demo=# DROP INDEX nulls_num_idx;
DROP INDEX
demo=# CREATE INDEX ON nulls (num);
CREATE INDEX

demo=# EXPLAIN ANALYZE
SELECT *
FROM nulls
ORDER BY num;
                                                             QUERY PLAN
------------------------------------------------------------------------------------------------------------------------------------
 Index Scan using nulls_num_idx on nulls  (cost=0.42..6289.44 rows=200001 width=14) (actual time=0.017..34.342 rows=200001 loops=1)
 Planning Time: 0.072 ms
 Execution Time: 42.278 ms
(3 rows)

-- Использование индекса для таблицы nulls дает выигрыш во времени выполнения запроса 
-- примерно в два раза (каждый запрос теста выполнялся дважды).
-- Когда в запросе в параметрах ORDER BY или в индексе переопределены веса NULLS,  
-- планировщик использует последовательное сканирование вместо индексного. 
-- Index Scan и  Index Scan Backward одинаковые по эффективности.
```

</details>
