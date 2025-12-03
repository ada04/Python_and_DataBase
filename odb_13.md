# under construction

# 13. Использование данных JSON
Данные JSON можно использовать с функциями реляционных баз данных, включая транзакции, индексацию, декларативные запросы и представления. JSON-данные можно проецировать реляционно, делая их доступными для реляционных процессов и инструментов. JSON-реляционные представления Duality предоставляют преимущества реляционной модели и SQL-доступа, а также позволяют читать и записывать данные в виде JSON-документов. См. также Simple Oracle Document Access (SODA) , который обеспечивает доступ к JSON-документам через набор API в стиле NoSQL.

Поддержка JSON появилась в Oracle Database 12c. Подробнее об использовании JSON в Oracle Database см. в руководстве разработчика Database JSON .

13.1 Использование JSON-типа базы данных Oracle в python-oracledb
В Oracle Database 21c появился специальный тип данных JSON с новым двоичным форматом хранения OSON , который повышает производительность и функциональность по сравнению с предыдущими версиями. Как режимы Thin, так и Thick python-oracledb поддерживают тип данных JSON. Для использования режима Thick клиентские библиотеки Oracle должны быть версии 21 или более поздней.

Чтобы создать таблицу со столбцом JSON_DATA для данных JSON, вы можете использовать:

create table CustomersAsJson (
    id integer not null primary key,
    json_data json
);
В Oracle Database 21c (или более поздней версии) при использовании тонкого режима python-oracledb или толстого режима с Oracle Client 21c (или более поздней версии) можно вставлять данные JSON, выполняя прямую привязку:

data = dict(name="Sally", dept="Sales", location="France")
insert_sql = "insert into CustomersAsJson values (:1, :2)"

# Take advantage of direct binding
cursor.setinputsizes(None, oracledb.DB_TYPE_JSON)
cursor.execute(insert_sql, [1, data])
Вызов Cursor.setinputsizes()использует None для указания того, что тип для первого заполнителя переменной привязки должен быть выведен из значения данных (то есть он является числовым), а использование , oracledb.DB_TYPE_JSONчтобы указать, что второй заполнитель в операторе SQL должен быть связан как тип JSON.

Извлечение столбца JSON автоматически возвращает объект Python:

for row in cursor.execute("select * from CustomersAsJson"):
    print(row)
Это дает:

(1, {'name': 'Sally', 'dept': 'Sales', 'location': 'France'})
Пример выполнения см . в файле json_direct.py . В этом примере также показано, как использовать этот тип, когда в режиме python-oracledb Thick используются старые клиентские библиотеки Oracle.

13.2 Использование JSON, хранящегося в столбцах BLOB, CLOB или VARCHAR2
В Oracle Database 12c и более поздних версиях данные JSON могут храниться в столбцах реляционных таблиц типа BLOB, CLOB или VARCHAR2. Все эти типы данных могут использоваться с python-oracledb в режимах Thin (тонкий) и Thick (толстый). Предпочтительнее использовать BLOB, чтобы избежать накладных расходов на преобразование кодировок.

Синтаксис создания таблицы со столбцом JSON с использованием хранилища BLOB выглядит следующим образом:

create table CustomersAsBlob (
    id integer not null primary key,
    json_data blob check (json_data is json)
);
Проверочное ограничение с предложением гарантирует, что в этом столбце хранятся только данные JSON.IS JSON

Этот старый синтаксис всё ещё можно использовать в Oracle Database 21c (и более поздних версиях). Однако рекомендуется перейти на новый тип JSON.

При использовании Oracle Database 12c или более поздней версии с JSON с использованием хранилища BLOB вы можете вставлять строки JSON следующим образом:

import json

data = dict(name="Rod", dept="Sales", location="Germany")
inssql = "insert into CustomersAsBlob values (:1, :2)"

cursor.setinputsizes(None, oracledb.DB_TYPE_LONG_RAW)
cursor.execute(inssql, [1, json.dumps(data)])
Вы можете извлекать столбцы VARCHAR2 и LOB, содержащие данные JSON, так же, как столбцы типа JSON извлекаются в Oracle Database 21c и более поздних версиях. При использовании режима python-oracledb Thick необходимо использовать Oracle Client 19c (или более поздней версии). Например:

for row in cursor.execute("select * from CustomersAsBlob"):
    print(row)
Изменено в версии 2.0: Поведение при выборке данных JSON, хранящихся в столбцах VARCHAR2 и LOB, имеющих проверочное ограничение, изменилось в версиях python-oracledb 1.4 и 2.0.IS JSON

В python-oracledb 2.0 выборка из этих столбцов в Oracle Database 12c (или более поздней версии) происходит так же, как выборка из столбца типа JSON в Oracle Database 21c (или более поздней версии): объект Python возвращается автоматически. Вам не нужно явно вызывать json.loads().

В python-oracledb 1.4 можно было задать атрибут oracledb.__future__.old_json_col_as_obj, чтобы изменить способ возврата этих столбцов. При значении False (значение по умолчанию) приложению требовалось вызывать json.loads()извлеченные значения. При значении True данные автоматически возвращались в виде объектов Python. Атрибут обеспечивал прямой путь миграции в python-oracledb 2.0.

Во всех версиях python-oracledb до 1.4 вашему приложению всегда требовалось вызывать json.loads()возвращаемые данные.

Атрибут oracledb.__future__.old_json_col_as_objбыл добавлен в python-oracledb 1.4 и удален в версии 2.0.

Смотрите json_blob.py для работающего примера.

13.2.1 Использование хранилища OSON
При использовании JSON с хранилищем VARCHAR или LOB в базах данных, поддерживающих OSON , оптимизированный двоичный формат JSON от Oracle, вы можете задать следующий параметр хранения:

create table mytab (json_data blob check (json_data is json format oson));
Чтобы вставить данные в эту таблицу, закодируйте их с помощью Connection.encode_oson(), а затем используйте Cursor.setinputsizes(), чтобы указать, что заполнитель переменной привязки представляет собой JSON:

data = dict(name="Sally", dept="Sales", location="France")

oson = connection.encode_oson(data)
cursor.setinputsizes(oracledb.DB_TYPE_JSON)
cursor.execute("insert into mytab (json_data) values (:1)", [oson])
При выборке используйте Connection.decode_json()для извлечения значений:

for (o,) in cursor.execute("select * from mytab"):
    d = connection.decode_oson(o)
    print(d)
13.3 Сопоставление типов привязки IN
При привязке к значению JSON typeпараметр переменной должен быть указан как oracledb.DB_TYPE_JSON. Значения Python преобразуются в значения JSON, как показано в следующей таблице. Синтаксис «SQL Equivalent» можно использовать в операторах SQL INSERT и UPDATE, если требуются определённые типы атрибутов, но прямое сопоставление из Python отсутствует.

Тип или значение Python

Тип или значение атрибута JSON

Пример эквивалента SQL

Никто

нулевой

НУЛЕВОЙ

Истинный

истинный

н/д

ЛОЖЬ

ЛОЖЬ

н/д

инт

ЧИСЛО

json_scalar(1)

плавать

ЧИСЛО

json_scalar(1)

decimal.Decimal

ЧИСЛО

json_scalar(1)

ул.

VARCHAR2

json_scalar('Строка')

датавремя.дата

МЕТКА ВРЕМЕНИ

json_scalar(to_timestamp('2020-03-10', 'ГГГГ-ММ-ДД'))

дата и время.дата и время

МЕТКА ВРЕМЕНИ

json_scalar(to_timestamp('2020-03-10', 'ГГГГ-ММ-ДД'))

байты

СЫРОЙ

json_scalar(utl_raw.cast_to_raw('Необработанное значение'))

список

Множество

json_array(1, 2, 3 возвращает json)

дикт

Объект

json_object(ключ 'Фред' значение json_scalar(5), ключ 'Джордж' значение json_scalar('Строка') возвращающий json)

н/д

КЛОБ

json_scalar(to_clob('Короткий CLOB'))

н/д

БЛОБ

json_scalar(to_blob(utl_raw.cast_to_raw('Короткий BLOB')))

н/д

ДАТА

json_scalar(to_date('2020-03-10', 'ГГГГ-ММ-ДД'))

oracledb.IntervalYM

ИНТЕРВАЛ ИЗ ГОДА В МЕСЯЦ

json_scalar(to_yminterval('+5-9'))

datetime.timedelta

ИНТЕРВАЛ ДЕНЬ-ВТОРОЙ

json_scalar(to_dsinterval('P25DT8H25M'))

н/д

BINARY_DOUBLE

json_scalar(to_binary_double(25))

н/д

BINARY_FLOAT

json_scalar(to_binary_float(15.5))

Пример создания атрибута CLOB с ключом mydocumentв столбце JSON с использованием SQL:

cursor.execute("""
    insert into mytab (
        myjsoncol
    ) values (
        json_object(key 'mydocument' value json_scalar(to_clob(:b)) returning json)
    )""",
    ['A short CLOB'])
При запросе к mytab в python-oracledb данные CLOB будут возвращены в виде строки Python, как показано в следующей таблице. Вывод может быть следующим:

{mydocument: 'A short CLOB'}
13.4 Сопоставление типов запросов и исходящих привязок
При получении значений JSON Oracle Database 21 из базы данных происходит следующее сопоставление атрибутов:

Тип или значение атрибута JSON базы данных

Тип или значение Python

нулевой

Никто

ЛОЖЬ

ЛОЖЬ

истинный

Истинный

ЧИСЛО

decimal.Decimal

VARCHAR2

ул.

СЫРОЙ

байты

КЛОБ

ул.

БЛОБ

байты

ДАТА

дата и время.дата и время

МЕТКА ВРЕМЕНИ

дата и время.дата и время

ИНТЕРВАЛ ИЗ ГОДА В МЕСЯЦ

oracledb.IntervalYM

ИНТЕРВАЛ ДЕНЬ-ВТОРОЙ

datetime.timedelta

BINARY_DOUBLE

плавать

BINARY_FLOAT

плавать

Массивы

список

Объекты

дикт

13.5 Выражения пути SQL/JSON
Oracle Database предоставляет SQL-доступ к данным JSON с помощью выражений пути SQL/JSON. Выражение пути выбирает ноль или более значений JSON, соответствующих или удовлетворяющих условию. Выражения пути могут использовать подстановочные знаки и диапазоны массивов. Простое выражение пути — это $.friendsзначение поля JSON friends.

Например, ранее созданную таблицу CUSTOMERS со столбцом JSON_DATA можно запросить следующим образом:

select c.json_data.location FROM customers c
Если JSON '{"name":"Rod","dept":"Sales","location":"Germany"}'сохранен в таблице, запрашиваемое значение будет Germany.

Функция JSON_EXISTS проверяет наличие определённого значения в некоторых JSON-данных. Чтобы найти записи JSON с locationполем:

import json

for blob, in cursor.execute("""
    select
        json_data
    from
        customers
    where
        json_exists(json_data,
        '$.location')"""):
    data = json.loads(blob.read())
    print(data)
Этот запрос может отображать:

{'name': 'Rod', 'dept': 'Sales', 'location': 'Germany'}
JSON_VALUEТакже могут быть использованы функции SQL/JSON JSON_QUERY.

Обратите внимание, что поведение обработки ошибок по умолчанию для этих функций — , то есть в случае возникновения ошибки значение не возвращается. Чтобы гарантировать возникновение ошибки, используйте .NULL ON ERRORERROR ON ERROR

Дополнительные сведения см. в разделе Выражения пути SQL/JSON в Руководстве разработчика Oracle JSON.

13.6 Доступ к реляционным данным в формате JSON
В Oracle Database 12.2 и более поздних версиях функция JSON_OBJECT — отличный способ преобразования данных реляционной таблицы в JSON:

cursor.execute("""
    select
        json_object('deptId' is d.department_id,
                    'name' is d.department_name) department
    from
        departments d
    where
        department_id < :did
    order by
        d.department_id""",
        [50]);
for row in cursor:
    print(row)
Это производит:

('{"deptId":10,"name":"Administration"}',)
('{"deptId":20,"name":"Marketing"}',)
('{"deptId":30,"name":"Purchasing"}',)
('{"deptId":40,"name":"Human Resources"}',)
Чтобы выбрать набор результатов из реляционного запроса как один объект, можно использовать JSON_ARRAYAGG , например:

cursor.execute("""
    select
        json_arrayagg(
            json_object('deptid' is d.department_id,
                        'name' is d.department_name) returning clob)
    from
        departments d
    where
        department_id < :did""",
   [50],
   fetch_lobs=False)
j, = cursor.fetchone()
print(j)
Это производит:

[{"deptid":10,"name":"Administration"},{"deptid":20,"name":"Marketing"},{"deptid":30,"name":"Purchasing"},{"deptid":40,"name":"Human Resources"}]
13.7. JSON-реляционные представления двойственности
Представления JSON-Relational Duality в Oracle AI Database 26ai позволяют хранить данные в виде строк в таблицах, используя преимущества реляционной модели и SQL-доступа, а также обеспечивают доступ к данным для чтения и записи в виде JSON-документов для упрощения работы приложений. Подробнее см. в руководстве разработчика JSON-Relational Duality .

Например, если таблицы AuthorTabи BookTabсуществуют:

create table AuthorTab (
    AuthorId number generated by default on null as identity primary key,
    AuthorName varchar2(100)
);

create table BookTab (
    BookId number generated by default on null as identity primary key,
    BookTitle varchar2(100),
    AuthorId number references AuthorTab (AuthorId)
);
Затем в SQL*Plus можно создать JSON Duality View для таблиц:

create or replace json relational duality view BookDV as
BookTab @insert @update @delete
{
    _id: BookId,
    book_title: BookTitle,
    author: AuthorTab @insert @update
    {
        author_id: AuthorId,
        author_name: AuthorName
    }
};
Приложения могут выбирать, использовать ли реляционный доступ к базовым таблицам или использовать дуальное представление.

Вы можете использовать SQL/JSON для запроса к представлению и возврата JSON. Запрос использует специальный столбец DATA:

sql = """select b.data.book_title, b.data.author.author_name
         from BookDV b
         where b.data.author.author_id = :1"""
for r in cursor.execute(sql, [1]):
    print(r)
Вставка JSON в представление обновит базовые реляционные таблицы:

data = dict(_id=1000, book_title="My New Book",
            author=dict(author_id=2000, author_name="John Doe"))
cursor.setinputsizes(oracledb.DB_TYPE_JSON)
cursor.execute("insert into BookDV values (:1)", [data])
Смотрите json_duality.py для работающего примера.

Вы также можете получить доступ к представлению дуальности напрямую, используя API документов в стиле SODA NoSQL, открыв представление как коллекцию SODA. Например, чтобы выполнить запрос по примерам по названиям книг:

soda = connection.getSodaDatabase()
collection = soda.openCollection('BOOKDV')

qbe = {'book_title': {'$like': 'The%'}}
for doc in collection.find().filter(qbe).getDocuments():
    content = doc.getContent()
    print(content["book_title"])
Смотрите soda_json_duality.py для работающего примера.
