# Решения практических заданий из учебного пособия [Моргунова Е. П. "PostgreSQL. Основы языка SQL"](https://postgrespro.ru/education/books/sqlprimer)

[Посмотреть решения >>](https://github.com/rubussta/sql-basics-course/blob/main/sql-basics.md)

В конце каждой главы учебного пособия есть "Контрольные вопросы и задания", но нет готовых ответов. Используя мои решения (с критическим подходом к ним) можно самостоятельно выполнять задания и потом сверяться с решениями либо пользоваться данной информацией как небольшым справочником практического применения синтаксиса, если что-то подзабылось. Например, как применять шаблоны POSIX или полнотекстовой поиск и т.п.

Как следует из названия учебного пособия, в нем рассматриваются начальные навыки работы с SQL и частично с базой данных. Не смотря на это, некоторые задания довольно сложные, особенно для новичков ранее не знакомых с SQL.

Задачи выполнены в [демонстрационной базе данных "Авиаперевозки".](https://postgrespro.ru/education/demodb) Есть три версии БД: small, medium и big. По структуре они одинаковые, но разные по количеству строк в таблицах фактов. В зависимости от того, какую юверсию БД вы себе установите, результаты запросов могут различаться по цифрам или результатам группировок и выборок. Эти задачи выполнены в БД demo-medium.zip версии (15.08.2017)

 ```sql
SELECT count(*) FROM flights;
 count
--------
 214867
(1 row)

```
Хочется отметить положительные стороны методического подхода к обучению авторами учебного пособия. Для работы с ним Вы должны самостоятельно развернуть у себя экземпляр базы данных и осваивать SQL пусть и на учебной, но близкой к реальности базе данных. В отличие от данного подхода, множество распространенных в сети тренажеров SQL упускают из виду целые пласты знаний и навыков по работам с базами данных, например, понимание структуры БД, оптимизацию запросов или администрирование БД и прочее. Одновременно выполнение заданий учебного пособия никак не огранивает пользователя в способах развертывания БД и использования ПО: можно установить БД к себе локально, или в виртуальную машину, или даже на удаленный сервер и для ввода команд использовать любую подходящую IDE (например, Dbeaver или pgAdmin) или просто терминал psql.

Большинство данных заданий сделаны в рамках учебного курса от компании Postgres Pro. Спасибо компании, преподаваталям и авторам учебного пособия за прекрасный материал.

[Посмотреть решения >>](https://github.com/rubussta/sql-basics-course/blob/main/sql-basics.md)  
[Скачать книгу Моргунова Е. П. "PostgreSQL. Основы языка SQL" >>](https://postgrespro.ru/education/books/sqlprimer)
