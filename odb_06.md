# Under construction

# 6. Выполнение SQL
Выполнение операторов SQL — основной способ взаимодействия приложения Python с базой данных Oracle. Операторы включают запросы, язык манипулирования данными (DML) и язык определения данных (DDL). Также возможно выполнение некоторых других специализированных операторов . Операторы выполняются одним из следующих методов Cursor.execute(): Cursor.executemany(), Connection.fetch_df_all(), Connection.fetch_df_batches(), AsyncCursor.execute(), AsyncCursor.executemany(), AsyncConnection.execute(), AsyncConnection.executemany(), AsyncConnection.fetch_df_all(), AsyncConnection.fetch_df_batches(), или AsyncConnection.run_pipeline().

В этой главе рассматриваются синхронные методы python-oracledb. Асинхронные методы и конвейеризация подробно рассматриваются в разделе « Параллельное программирование с использованием asyncio и конвейеризация» .

Операторы PL/SQL обсуждаются в разделе «Выполнение PL/SQL» . В следующих главах содержится информация о конкретных типах данных и функциях:

Пакетные операции выписки и массового копирования

Конвейеризация операций с базой данных

Использование данных CLOB, BLOB, NCLOB и BFILE

Использование данных JSON

Использование данных XMLTYPE

Для помощи в создании операторов SQL и PL/SQL Oracle предлагает FreeSQL.com — онлайн-редактор, мгновенно подключаемый к базе данных. Там же есть учебные материалы и библиотека операторов.

Выполнение SQL-скриптов

Python-oracledb можно использовать для выполнения отдельных операторов, по одному за раз. Только после завершения выполнения одного оператора будет выполнен следующий. При попытке одновременного выполнения операторов в одном соединении они будут поставлены в очередь и запущены последовательно в том порядке, в котором они выполняются в коде приложения. Это относится и к конвейерным операторам .

Python-oracledb не читает файлы SQL*Plus «.sql». Для чтения SQL-файлов используйте метод, описанный в run_sql_script()файле examples/sample_env.py .

Синтаксис оператора SQL

SQL-запросы, выполняемые в python-oracledb, не должны содержать точку с запятой («;») или косую черту («/»). Это приведёт к ошибке:

cursor.execute("select * from MyTable;")   # fails due to semicolon
Это верно:

cursor.execute("select * from MyTable")
Важный

Например, интерполяция или конкатенация пользовательских данных с операторами SQL представляет угрозу безопасности и влияет на производительность. Вместо этого используйте переменные связывания , например .cursor.execute("SELECT * FROM mytab WHERE mycol = '" + myvar + "'")cursor.execute("SELECT * FROM mytab WHERE mycol = :mybv", mybv=myvar)

6.1. SQL-запросы
Запросы (операторы, начинающиеся с SELECT или WITH) можно выполнять с помощью метода Cursor.execute(). Затем строки можно перебирать или извлекать с помощью одного из методов Cursor.fetchone(), Cursor.fetchmany()или Cursor.fetchall(). Это позволяет обрабатывать строки напрямую или при необходимости передавать их потоком. Существует сопоставление типов по умолчанию с типами Python, которое можно при необходимости переопределить .

6.1.1 Методы выборки
Строки можно извлекать различными способами.

После этого Cursor.execute()курсор возвращается для удобства. Это позволяет коду перебирать строки, например:

cursor = connection.cursor()
for row in cursor.execute("select * from MyTable"):
    print(row)
Строки также можно извлекать по одной с помощью метода Cursor.fetchone():

cursor = connection.cursor()
cursor.execute("select * from MyTable")
while True:
    row = cursor.fetchone()
    if row is None:
        break
    print(row)
Если строки необходимо передавать потоком или обрабатывать пакетами, Cursor.fetchmany()можно использовать этот метод. Размер пакета контролируется параметром size, который по умолчанию равен Cursor.arraysize.

cursor = connection.cursor()
cursor.execute("select * from MyTable")
num_rows = 10
while True:
    rows = cursor.fetchmany(size=num_rows)
    if not rows:
        break
    for row in rows:
        print(row)
Обратите внимание, что этот sizeпараметр влияет только на количество строк, возвращаемых приложению, а не на размер внутреннего буфера, используемого для настройки производительности выборки. Размер внутреннего буфера контролируется только изменением Cursor.arraysizeили oracledb.defaults.arraysize, см. раздел Настройка производительности выборки .

Если необходимо извлечь все строки и их можно поместить в память, Cursor.fetchall()можно использовать этот метод.

cursor = connection.cursor()
cursor.execute("select * from MyTable")
rows = cursor.fetchall()
for row in rows:
    print(row)
Методы выборки возвращают данные в виде кортежей. Чтобы получить результаты в виде словарей, см. раздел Изменение результатов запроса с помощью Rowfactories .

Данные также можно извлекать в формате фрейма данных, см. раздел Работа с фреймами данных .

6.1.2 Закрытие курсоров
Когда курсоры больше не нужны, их следует закрыть, чтобы освободить ресурсы базы данных. Обратите внимание: курсоры могут использоваться для выполнения нескольких операторов, прежде чем будут закрыты.

Курсоры можно закрыть различными способами:

Курсор будет автоматически закрыт, когда ссылающаяся на него переменная выйдет из области видимости (и дальнейшие ссылки не сохранятся). withБлок менеджера контекста — удобный и предпочтительный способ обеспечить это. Например:

with connection.cursor() as cursor:
    for row in cursor.execute("select * from MyTable"):
        print(row)
Этот код гарантирует, что после завершения блока курсор будет закрыт, и ресурсы базы данных могут быть освобождены. Кроме того, любая попытка использовать переменную cursorвне блока завершится неудачей.

Курсоры можно явно закрыть, вызвав Cursor.close():

cursor = connection.cursor()

...

cursor.close()
6.1.3. Запрос метаданных столбца
После выполнения запроса метаданные столбцов, такие как имена столбцов и типы данных, можно получить с помощью Cursor.description:

with connection.cursor() as cursor:
    cursor.execute("select * from MyTable")
    for column in cursor.description:
        print(column)
Это может привести к появлению таких метаданных:

('ID', <class 'oracledb.DB_TYPE_NUMBER'>, 39, None, 38, 0, 0)
('NAME', <class 'oracledb.DB_TYPE_VARCHAR'>, 20, 20, None, None, 1)
Чтобы извлечь имена столбцов из запроса, можно использовать такой код:

with connection.cursor() as cursor:
    cursor.execute("select * from locations")
    columns = [col.name for col in cursor.description]
    print(columns)
    for r in cursor:
        print(r)
Это напечатает:

['LOCATION_ID', 'STREET_ADDRESS', 'POSTAL_CODE', 'CITY', 'STATE_PROVINCE', 'COUNTRY_ID']
(1000, '1297 Via Cola di Rie', '00989', 'Roma', None, 'IT')
(1100, '93091 Calle della Testa', '10934', 'Venice', None, 'IT')
. . .
Изменение названий столбцов на строчные буквы

Чтобы изменить названия всех столбцов на строчные, можно сделать следующее:

cursor.execute("select * from locations where location_id = 1000")

columns = [col.name.lower() for col in cursor.description]
print(columns)
Вывод такой:

['location_id', 'street_address', 'postal_code', 'city', 'state_province',
'country_id']
6.1.4. Выборка типов данных
В следующей таблице представлен список всех типов данных, которые python-oracledb умеет извлекать. В среднем столбце указан тип, возвращаемый в метаданных запроса . В последнем столбце указан тип объекта Python, возвращаемый по умолчанию. Типы Python можно изменить с помощью обработчиков выходных типов .

Тип базы данных Oracle

Тип базы данных oracledb

Тип Python по умолчанию

BFILE

oracledb.DB_TYPE_BFILE

oracledb.LOB

BINARY_DOUBLE

oracledb.DB_TYPE_BINARY_DOUBLE

плавать

BINARY_FLOAT

oracledb.DB_TYPE_BINARY_FLOAT

плавать

БЛОБ

oracledb.DB_TYPE_BLOB

oracledb.LOB

СИМВОЛ

oracledb.DB_TYPE_CHAR

ул.

КЛОБ

oracledb.DB_TYPE_CLOB

oracledb.LOB

КУРСОР

oracledb.DB_TYPE_CURSOR

oracledb.Cursor

ДАТА

oracledb.DB_TYPE_DATE

дата и время.дата и время

ИНТЕРВАЛ ДЕНЬ-ВТОРОЙ

oracledb.DB_TYPE_INTERVAL_DS

datetime.timedelta

ИНТЕРВАЛ ИЗ ГОДА В МЕСЯЦ

oracledb.DB_TYPE_INTERVAL_YM

oracledb.IntervalYM

JSON

oracledb.DB_TYPE_JSON

dict, list или скалярное значение 4

ДЛИННЫЙ

oracledb.DB_TYPE_LONG

ул.

ДЛИННЫЙ СЫРОЙ

oracledb.DB_TYPE_LONG_RAW

байты

НЧАР

oracledb.DB_TYPE_NCHAR

ул.

НКЛОБ

oracledb.DB_TYPE_NCLOB

oracledb.LOB

ЧИСЛО

oracledb.DB_TYPE_NUMBER

float или int 1

NVARCHAR2

oracledb.DB_TYPE_NVARCHAR

ул.

ОБЪЕКТ 3

oracledb.DB_TYPE_OBJECT

oracledb.Object

СЫРОЙ

oracledb.DB_TYPE_RAW

байты

ROWID

oracledb.DB_TYPE_ROWID

ул.

МЕТКА ВРЕМЕНИ

oracledb.DB_TYPE_TIMESTAMP

дата и время.дата и время

ВРЕМЕННАЯ МЕТКА С МЕСТНЫМ ЧАСОВЫМ ПОЯСОМ

oracledb.DB_TYPE_TIMESTAMP_LTZ

дата и время.дата и время 2

ВРЕМЕННАЯ МЕТКА С ЧАСОВЫМ ПОЯСОМ

oracledb.DB_TYPE_TIMESTAMP_TZ

дата и время.дата и время 2

УРОВИД

oracledb.DB_TYPE_ROWID,oracledb.DB_TYPE_UROWID

ул.

VARCHAR2

oracledb.DB_TYPE_VARCHAR

ул.

[ 1 ]
Если точность и масштаб, полученные из метаданных столбца запроса, указывают на то, что значение может быть выражено целым числом, значение будет возвращено как int. Если столбец не ограничен (точность и масштаб не указаны), значение будет возвращено как float или int в зависимости от того, является ли само значение целым числом. Во всех остальных случаях значение возвращается как float.

[ 2 ]
( 1 , 2 )
Возвращаемые временные метки являются примитивными временными метками, не содержащими никакой информации о часовом поясе.

[ 3 ]
К ним относятся все определяемые пользователем типы данных, такие как VARRAY, NESTED TABLE и т. д.

[ 4 ]
Если JSON — объект, возвращается словарь. Если это массив, возвращается список. Если это скалярное значение, возвращается это скалярное значение.

6.1.5 Изменение извлеченных данных
Данные, возвращаемые запросами python-oracledb, можно изменять с помощью обработчиков типов вывода, с помощью «выходных преобразователей» или с помощью фабрик строк.

6.1.5.1 Изменение типов извлеченных данных с помощью обработчиков выходных типов
Иногда преобразование по умолчанию из типа данных Oracle Database в тип данных Python необходимо изменить, чтобы предотвратить потерю данных или соответствовать целям приложения Python. В таких случаях для запросов можно указать обработчик выходных типов. Это предписывает базе данных выполнить преобразование из типа столбца в другой тип данных перед возвратом данных из базы данных в python-oracledb. Если база данных не поддерживает такое преобразование, будет возвращена ошибка. Обработчики выходных типов влияют только на выходные данные запроса и не влияют на значения, возвращаемые из Cursor.callfunc()или Cursor.callproc().

Обработчики типов выходных данных могут быть указаны для connectionили для cursor. Если они указаны для курсора, обработка типа выборки изменяется только для этого конкретного курсора. Если они указаны для соединения, обработка типа выборки для всех курсоров, созданных этим соединением, будет изменена.

Ожидается, что обработчик выходного типа будет функцией со следующей сигнатурой:

handler(cursor, metadata)
Параметр метаданных — это объект FetchInfo , который имеет то же значение, что и в Cursor.description.

Функция вызывается один раз для каждого извлекаемого столбца. Ожидается, что функция вернет объект переменной (обычно посредством вызова Cursor.var()) или значение None. Значение Noneуказывает, что следует использовать тип данных по умолчанию.

Например:

def output_type_handler(cursor, metadata):
    if metadata.type_code is oracledb.DB_TYPE_NUMBER:
        return cursor.var(oracledb.DB_TYPE_VARCHAR, arraysize=cursor.arraysize)
Этот обработчик выходных данных вызывается один раз для каждого столбца в запросе SELECT. Для каждого числового столбца база данных теперь будет возвращать строковое представление значения каждой строки. Использование этого в запросе:

cursor.outputtypehandler = output_type_handler

cursor.execute("select 123 from dual")
r = cursor.fetchone()
print(r)
выводит ('123',)информацию о том, что число было преобразовано в строку. Без обработчика типов вывод был бы таким (123,): .

При создании переменных Cursor.var()в обработчике arraysizeпараметр должен совпадать с параметром Cursor.arraysizeкурсора запроса. В режиме Thick в Python-Oracledb var()размер массива запроса (и ), умноженный на размер байтов конкретного столбца, должен быть меньше INT_MAX.

Чтобы отменить обработчик типа выходных данных, установите его в значение None. Например, если вы ранее установили обработчик типа для курсора, вы можете отменить его с помощью:

cursor.outputtypehandler = None
Другие примеры обработчиков выходных данных приведены в разделах «Точность извлечения чисел» , «Извлечение больших объектов в виде строк и байтов» и «Извлечение необработанных данных» . См. также примеры, например, examples/type_handlers_json_strings.py .

6.1.5.2. Изменение результатов запроса с помощью преобразователей
«Выходные преобразователи» Python-oracledb можно использовать с обработчиками выходных типов для изменения возвращаемых данных.

Например, чтобы преобразовать числа в строки:

def output_type_handler(cursor, metadata):

    def out_converter(d):
        if isinstance(d, str):
            return f"{d} was a string"
        else:
            return f"{d} was not a string"

    if metadata.type_code is oracledb.DB_TYPE_NUMBER:
        return cursor.var(oracledb.DB_TYPE_VARCHAR,
             arraysize=cursor.arraysize, outconverter=out_converter)
Обработчик выходных типов вызывается один раз для каждого столбца в запросе SELECT. Для каждого числового столбца база данных теперь будет возвращать строковое представление значения каждой строки, и выходной преобразователь будет вызываться для каждого из этих значений.

Использование в запросе:

cursor.outputtypehandler = output_type_handler

cursor.execute("select 123 as col1, 'abc' as col2 from dual")
for r in cursor.fetchall():
    print(r)
отпечатки:

('123 was a string', 'abc')
Это показывает, что число было сначала преобразовано базой данных в строку, как того требовал обработчик выходных данных. out_converterЗатем функция добавила к данным «была строкой», прежде чем значение было возвращено приложению.

Обратите внимание, что выходные преобразователи не вызываются для значений данных NULL, если Cursor.var()параметр не convert_nullsимеет значение True .

Другой пример выходного преобразователя показан при извлечении ВЕКТОРОВ в виде списков .

6.1.5.3 Изменение результатов запроса с помощью Rowfactories
«Фабрики строк» ​​Python-oracledb — это методы, вызываемые для каждой строки, извлекаемой из базы данных. Метод Cursor.rowfactory()вызывается с кортежем, извлеченным из базы данных, перед его возвратом в приложение. Метод может преобразовать кортеж в другое значение или представление.

Строку rowfactory следует установить для курсора после вызова метода , Cursor.execute() прежде чем извлекать данные из курсора. execute()Повторный вызов очистит любую предыдущую строку rowfactory.

Извлечение строк с использованием класса данных

Классы данных Python предоставляют простой способ инкапсуляции данных. Пример их использования с запросом rowfactory:

import dataclasses
import datetime

. . .

@dataclasses.dataclass
class MyRow:
    employee_id: int
    last_name: str
    hire_date: datetime.datetime

cursor.execute(
    """select employee_id, last_name, hire_date
       from employees
       where employee_id < 105
       order by employee_id""")

cursor.rowfactory = MyRow

for row in cursor:
    print("Number:", row.employee_id)
    print("Name:", row.last_name)
    print("Hire Date:", row.hire_date)
Вывод такой:

Number: 100
Name: King
Hire Date: 2003-06-17 00:00:00
Number: 101
Name: Kochhar
Hire Date: 2005-09-21 00:00:00
Number: 102
Name: De Haan
Hire Date: 2001-01-13 00:00:00
Извлечение строк как словарей

Чтобы извлечь каждую строку запроса как словарь, можно использовать Cursor.rowfactory()следующее:

cursor.execute("select * from locations where location_id = 1000")

columns = [col.name for col in cursor.description]
cursor.rowfactory = lambda *args: dict(zip(columns, args))
data = cursor.fetchone()
print(data)
Вывод такой:

{'LOCATION_ID': 1000, 'STREET_ADDRESS': '1297 Via Cola di Rie',
'POSTAL_CODE': '00989', 'CITY': 'Roma', 'STATE_PROVINCE': None,
'COUNTRY_ID': 'IT'}
Также см. раздел JSON_OBJECTИспользование данных JSON , поскольку выполнение запросов напрямую в формате JSON может быть предпочтительнее.

Если вы объединяете таблицы, в которых одно и то же имя столбца встречается в обеих таблицах, но имеет разное значение или значение, используйте в запросе псевдоним столбца. В противном случае в словарь будет включён только один из столбцов с похожим именем:

select
    cat_name,
    cats.color as cat_color,
    dog_name,
    dogs.color
from cats, dogs
Пример с обработчиком выходных типов, выходным преобразователем и фабрикой строк

Пример, демонстрирующий обработчик выходных типов , выходной преобразователь и фабрику строк :

def output_type_handler(cursor, metadata):

    def out_converter(d):
        if type(d) is str:
            return f"{d} was a string"
        else:
            return f"{d} was not a string"

    if metadata.type_code is oracledb.DB_TYPE_NUMBER:
        return cursor.var(oracledb.DB_TYPE_VARCHAR,
            arraysize=cursor.arraysize, outconverter=out_converter)

cursor.outputtypehandler = output_type_handler

cursor.execute("select 123 as col1, 'abc' as col2 from dual")

columns = [col.name.lower() for col in cursor.description]
cursor.rowfactory = lambda *args: dict(zip(columns, args))
for r in cursor.fetchall():
    print(r)
База данных преобразует число в строку перед возвратом в python-oracledb. Выходной преобразователь добавляет к этому значению «was a string» (была строкой). Имена столбцов преобразуются в нижний регистр. Наконец, фабрика строк преобразует всю строку в словарь. Результат:

{'col1': '123 was a string', 'col2': 'abc'}
6.1.6 Точность извлеченного числа
В базе данных Oracle используются десятичные числа, которые невозможно преобразовать в двоичные представления, например, числа с плавающей точкой в ​​Python. Кроме того, диапазон чисел Oracle превышает диапазон чисел с плавающей точкой. В Python есть объекты десятичных чисел, не имеющие этих ограничений. В python-oracledb можно настроить oracledb.defaults.fetch_decimalsвозврат десятичных чисел в приложение, гарантируя сохранение точности при выборке определённых чисел.

Следующий пример кода демонстрирует проблему:

cursor.execute("create table test_float (X number(5, 3))")
cursor.execute("insert into test_float values (7.1)")

cursor.execute("select * from test_float")
val, = cursor.fetchone()
print(val, "* 3 =", val * 3)
Это отображает7.1 * 3 = 21.299999999999997

Однако при использовании десятичных объектов Python потери точности не происходит:

oracledb.defaults.fetch_decimals = True

cursor.execute("select * from test_float")
val, = cursor.fetchone()
print(val, "* 3 =", val * 3)
Это отображает7.1 * 3 = 21.3

См. examples/return_numbers_as_decimals.py

Эквивалентный, более длинный и старый способ кодирования для настройки oracledb.defaults.fetch_decimals— использование обработчика выходного типа для выполнения преобразования.

import decimal

def number_to_decimal(cursor, metadata):
    if metadata.type_code is oracledb.DB_TYPE_NUMBER:
        return cursor.var(decimal.Decimal, arraysize=cursor.arraysize)

cursor.outputtypehandler = number_to_decimal

cursor.execute("select * from test_float")
val, = cursor.fetchone()
print(val, "* 3 =", val * 3)
Это отображает7.1 * 3 = 21.3

Конвертер Python decimal.Decimalвызывается со строковым представлением числа Oracle. Выходные данные decimal.Decimalвозвращаются в выходном кортеже.

6.1.7 Прокручиваемые курсоры
Прокручиваемые курсоры позволяют приложениям перемещаться назад и вперёд, пропускать строки и переходить к определённой строке в наборе результатов запроса. Набор результатов кэшируется на сервере базы данных до закрытия курсора. В отличие от них, обычные курсоры ограничены перемещением вперёд.

Прокручиваемый курсор создаётся путём установки параметра scrollable=True при его создании. Этот метод Cursor.scroll()используется для перемещения между различными позициями в результирующем наборе.

Вот примеры:

cursor = connection.cursor(scrollable=True)
cursor.execute("select * from ChildTable order by ChildId")

cursor.scroll(mode="last")
print("LAST ROW:", cursor.fetchone())

cursor.scroll(mode="first")
print("FIRST ROW:", cursor.fetchone())

cursor.scroll(8, mode="absolute")
print("ROW 8:", cursor.fetchone())

cursor.scroll(6)
print("SKIP 6 ROWS:", cursor.fetchone())

cursor.scroll(-4)
print("SKIP BACK 4 ROWS:", cursor.fetchone())
Смотрите examples/scrollable_cursors.py для работающего примера.

6.1.8 Извлечение объектов и коллекций базы данных Oracle
Именованные типы объектов Oracle Database и пользовательские типы могут быть получены непосредственно в запросах. Каждый элемент представлен объектом Python, соответствующим объекту Oracle Database. Этот объект Python можно просматривать для доступа к его элементам. Доступны атрибуты, включая DbObjectType.nameи DbObjectType.iscollection, и методы, включая DbObject.aslist()и .DbObject.asdict()

Например, если таблица MYGEOMETRYTAB содержит столбец GEOMETRY предопределенного типа пространственного объекта Oracle SDO_GEOMETRY , то ее можно запросить и вывести:

cursor.execute("select geometry from mygeometrytab")
for obj, in cursor:
    dumpobject(obj)
Где dumpobject()определяется как:

def dumpobject(obj, prefix = ""):
    if obj.type.iscollection:
        print(prefix, "[")
        for value in obj.aslist():
            if isinstance(value, oracledb.Object):
                dumpobject(value, prefix + "  ")
            else:
                print(prefix + "  ", repr(value))
        print(prefix, "]")
    else:
        print(prefix, "{")
        for attr in obj.type.attributes:
            value = getattr(obj, attr.name)
            if isinstance(value, oracledb.Object):
                print(prefix + "   " + attr.name + ":")
                dumpobject(value, prefix + "  ")
            else:
                print(prefix + "   " + attr.name + ":", repr(value))
        print(prefix, "}")
Это может привести к такому результату:

{
  SDO_GTYPE: 2003
  SDO_SRID: None
  SDO_POINT:
  {
    X: 1
    Y: 2
    Z: 3
  }
  SDO_ELEM_INFO:
  [
    1
    1003
    3
  ]
  SDO_ORDINATES:
  [
    1
    1
    5
    7
  ]
}
Дополнительную информацию об использовании объектов Oracle можно найти в разделе Использование переменных привязки .

Приложениям, чувствительным к производительности, следует рассмотреть возможность использования скалярных типов вместо объектов. Если вы всё же используете объекты, избегайте Connection.gettype() ненужных вызовов и не используйте объекты с большим количеством атрибутов.

6.1.9 Ограничение строк
Данные запроса обычно разбиваются на один или несколько наборов:

Чтобы задать верхнюю границу количества строк, которые должен обработать запрос, что может помочь улучшить масштабируемость базы данных.

Для выполнения «веб-пагинации», которая позволяет переходить от одного набора строк к следующему или предыдущему набору по требованию.

Для выборки всех данных небольшими последовательными наборами для пакетной обработки. Это происходит, поскольку количество записей слишком велико для одновременной обработки Python.

Последнее можно сделать, вызвав Cursor.fetchmany()одно выполнение SQL-запроса.

В этом разделе подробно описывается «веб-пагинация» и ограничение максимального количества строк. Для каждой страницы результатов выполняется SQL-запрос для получения соответствующего набора строк из таблицы. Поскольку запрос может быть выполнен несколько раз, обязательно используйте переменные привязки для номеров строк и ограничения количества строк.

В Oracle Database 12c SQL появился оператор OFFSET/ FETCH, аналогичный ключевому LIMITслову MySQL. В Python для извлечения набора строк используется:

myoffset = 0       # do not skip any rows (start at row 1)
mymaxnumrows = 20  # get 20 rows

sql =
  """SELECT last_name
     FROM employees
     ORDER BY last_name
     OFFSET :offset ROWS FETCH NEXT :maxnumrows ROWS ONLY"""

with connection.cursor() as cursor:

    cursor.prefetchrows = mymaxnumrows + 1
    cursor.arraysize = mymaxnumrows

    for row in cursor.execute(sql, offset=myoffset, maxnumrows=mymaxnumrows):
        print(row)
В приложениях, где SQL-запрос заранее неизвестен, этот метод иногда подразумевает добавление предложения OFFSETк «реальному» пользовательскому запросу. Будьте очень осторожны, чтобы избежать проблем безопасности, связанных с SQL-инъекциями.

Для Oracle Database 11g и более ранних версий существует несколько альтернативных способов ограничения количества возвращаемых строк. Старый канонический запрос на страничный вывод выглядит следующим образом:

SELECT *
FROM (SELECT a.*, ROWNUM AS rnum
      FROM (YOUR_QUERY_GOES_HERE -- including the order by) a
      WHERE ROWNUM <= MAX_ROW)
WHERE rnum >= MIN_ROW
Здесь MIN_ROW— номер первой строки, а MAX_ROW— номер последней строки, которую нужно вернуть. Например:

SELECT *
FROM (SELECT a.*, ROWNUM AS rnum
      FROM (SELECT last_name FROM employees ORDER BY last_name) a
      WHERE ROWNUM <= 20)
WHERE rnum >= 1
Здесь всегда имеется «дополнительный» столбец, который здесь называется RNUM.

Альтернативный и предпочтительный синтаксис запроса для Oracle Database 11g использует аналитическую ROW_NUMBER()функцию. Например, чтобы получить имена с 1-го по 20-е, запрос выглядит следующим образом:

SELECT last_name FROM
(SELECT last_name,
        ROW_NUMBER() OVER (ORDER BY last_name) AS myr
        FROM employees)
WHERE myr BETWEEN 1 and 20
Обязательно используйте переменные привязки для верхних и нижних предельных значений.

6.1.10. Параллельная выборка данных
Выигрыш в производительности при параллельном выборе табличных данных из Oracle Database по сравнению с выполнением одного запроса зависит от многих факторов. Секционирование таблицы и чтение одной секции на одно подключение обычно является наиболее эффективным решением на стороне базы данных. Однако, даже если параллельное решение кажется более быстрым, оно может быть неэффективным, что скажется на производительности всех остальных систем или в конечном итоге будет ограничено ими. Только бенчмаркинг в вашей среде позволит определить целесообразность использования этого метода.

Наивный пример использования нескольких потоков:

# A naive example for fetching data in parallel.
# Many factors affect whether this is beneficial

# The degree of parallelism / number of connections to open
NUM_THREADS = 10

# How many rows to fetch in each thread
BATCH_SIZE = 1000

# Internal buffer size: Tune for performance
oracledb.defaults.arraysize = 1000

# Note OFFSET/FETCH is not particularly efficient.
# It would be better to use a partitioned table
SQL = """
    select data
    from demo
    order by id
    offset :rowoffset rows fetch next :maxrows rows only
    """

def do_query(tn):
    with pool.acquire() as connection:
        with connection.cursor() as cursor:
            cursor.execute(SQL, rowoffset=(tn*BATCH_SIZE), maxrows=BATCH_SIZE)
            while True:
                rows = cursor.fetchmany()
                if not rows:
                    break
                print(f'Thread {tn}', rows)


pool = oracledb.create_pool(user="hr", password=userpwd, dsn="dbhost.example.com/orclpdb",
                            min=NUM_THREADS, max=NUM_THREADS)

thread = []
for i in range(NUM_THREADS):
    t = threading.Thread(target=do_query, args=(i,))
    t.start()
    thread.append(t)

for i in range(NUM_THREADS):
    thread[i].join()
При рассмотрении вопроса о распараллеливании запросов из таблицы следует учитывать ряд факторов:

Каждое подключение к Oracle Database может выполнять только один оператор за раз, поэтому для распараллеливания запросов требуется использование нескольких подключений.

Потоковое поведение Python и влияние глобальной блокировки интерпретатора (GIL) могут оказывать влияние. Возможно, вам потребуется распределить работу между несколькими процессами.

Какой уровень параллелизма наиболее эффективен?

Сколько строк нужно выбрать в каждом пакете?

Что ваше приложение делает с данными — может ли принимающая сторона эффективно их обработать или записать на диск?

Синтаксис OFFSET FETCH по-прежнему будет приводить к сканированию блоков таблицы базы данных, даже если не все данные возвращаются в приложение. Можно ли вместо этого разбить таблицу на разделы?

На базу данных будет добавлена ​​дополнительная нагрузка, как из-за дополнительных подключений, так и из-за выполняемой ими работы.

Используют ли ваши запросы все параллельные серверы базы данных?

Распределены ли данные в базе данных по нескольким дисковым шпинделям или же им приходится постоянно осуществлять поиск на одном диске?

Используются ли карты зон Oracle Database?

Используется ли Oracle Exadata с индексами хранения?

Есть ли у вас индексы на основе функций, которые вызываются для каждой строки?

6.1.11. Получение необработанных данных
Иногда у python-oracledb могут возникнуть проблемы с преобразованием данных, хранящихся в базе данных, в строки Python. Это может произойти, если данные, хранящиеся в базе данных, не соответствуют набору символов, заданному в базе данных. Параметр encoding_errorsto Cursor.var()позволяет вернуть данные с заменой некоторых недопустимых данных, но для дополнительного контроля bypass_decodeможно установить параметр в значение True, и python-oracledb пропустит этап декодирования и вернет байты вместо str для данных, хранящихся в базе данных в виде строк. Затем данные можно будет проверить и исправить при необходимости. Этот подход следует использовать только для устранения неполадок и исправления недопустимых данных, а не для общего применения!

Следующий пример демонстрирует, как использовать эту функцию:

# define output type handler
def return_strings_as_bytes(cursor, metadata):
    if metadata.type_code is oracledb.DB_TYPE_VARCHAR:
        return cursor.var(str, arraysize=cursor.arraysize,
                          bypass_decode=True)

# set output type handler on cursor before fetching data
with connection.cursor() as cursor:
    cursor.outputtypehandler = return_strings_as_bytes
    cursor.execute("select content, charset from SomeTable")
    data = cursor.fetchall()
Это даст следующий результат:

[(b'Fianc\xc3\xa9', b'UTF-8')]
Обратите внимание, что в кодировке UTF-8 последним \xc3\xa9является é. Поскольку это допустимый формат UTF-8, можно выполнить декодирование данных (той части, которая была пропущена):

value = data[0][0].decode("UTF-8")
Это вернет значение «Жених».

Если вы хотите сохранить данные b'Fianc\xc3\xa9'в базу данных напрямую, без использования строки Python, вам потребуется создать переменную с типом Cursor.var(), указывающим тип oracledb.DB_TYPE_VARCHAR(иначе значение будет обработано как oracledb.DB_TYPE_RAW). Следующий пример демонстрирует это:

with oracledb.connect(user="hr", password=userpwd,
                       dsn="dbhost.example.com/orclpdb") as conn:
    with conn.cursor() cursor:
        var = cursor.var(oracledb.DB_TYPE_VARCHAR)
        var.setvalue(0, b"Fianc\xc4\x9b")
        cursor.execute("""
            update SomeTable set
                SomeColumn = :param
            where id = 1""",
            param=var)
Предупреждение

База данных предполагает, что предоставленные байты соответствуют набору символов, ожидаемому базой данных, поэтому используйте это только для устранения неполадок или по назначению.

6.1.12. Запрос поврежденных данных
Если при выборе данных запросы завершаются ошибкой «кодек не может декодировать байт», то:

Проверьте правильность кодировки . Проверьте кодировки базы данных . Проверьте раздел «Извлечение необработанных данных ». Обратите внимание, что для всех символьных данных в python-oracledb используется кодировка «UTF-8».

Проверьте базу данных на наличие повреждённых данных и исправьте их. Например, если у вас есть таблица MYTABLE с символьным столбцом MYVALUE, в котором, как вы подозреваете, есть повреждённые значения, вы можете определить проблемные данные, используя запрос, который выведет ключи строк с недопустимыми данными.select id from mytable where utl_i18n.validate_character_encoding(myvalue) > 0

Если повреждённые данные невозможно изменить, вы можете передать параметры внутреннему методу decode(), используемому python-oracledb, чтобы разрешить их выборку и предотвратить сбой всего запроса. Для этого создайте обработчик outputtypehandler и установите encoding_errors. Например, чтобы заменить повреждённые символы в столбцах с символами:

def output_type_handler(cursor, metadata):
    if metadata.type_code is oracledb.DB_TYPE_VARCHAR:
        return cursor.var(metadata.type_code, size,
                          arraysize=cursor.arraysize,
                          encoding_errors="replace")

cursor.outputtypehandler = output_type_handler

cursor.execute("select column1, column2 from SomeTableWithBadData")
Другие варианты поведения кодека можно выбрать encoding_errors, см. раздел Обработчики ошибок .

6.2. Операторы INSERT и UPDATE
Операторы языка манипулирования данными SQL (DML), такие как INSERT и UPDATE, можно легко выполнить с помощью python-oracledb. Например:

with connection.cursor() as cursor:
  cursor.execute("insert into MyTable values (:idbv, :nmbv)", [1, "Fredico"])
Не объединяйте и не интерполируйте пользовательские данные в SQL-выражения. См. раздел «Использование переменных привязки» .

При обработке нескольких значений данных используйте Cursor.executemany()или Connection.direct_path_load()для повышения производительности. См. разделы «Пакетный оператор» и «Операции массового копирования».

По умолчанию данные не фиксируются в базе данных, и другие пользователи не смогут увидеть ваши изменения, пока ваше подключение не зафиксирует их с помощью вызова Connection.commit(). При желании вы можете откатить изменения, вызвав Connection.rollback(). Неявный откат произойдет, если ваше приложение завершит работу и не зафиксирует работу явно.

Чтобы внести изменения, позвоните:

connection.commit()
Обратите внимание, что фиксация происходит в соединении, а не в курсоре.

Если атрибут Connection.autocommitравен True, то каждый выполненный оператор автоматически фиксируется без необходимости вызова Connection.commit(). Однако чрезмерное использование атрибута приводит к дополнительной нагрузке на базу данных и может нарушить согласованность транзакций.

Передовые методы фиксации и отката изменений данных см. в разделе Управление транзакциями .

6.2.1 Вставка значений NULL
Для Oracle Database требуется тип, даже для значений NULL. При передаче значения None python-oracledb предполагает, что тип — строка. Если это неподходящий тип, его можно указать явно. Например, чтобы вставить пустой объект Oracle Spatial SDO_GEOMETRY :

type_obj = connection.gettype("SDO_GEOMETRY")
cursor = connection.cursor()
cursor.setinputsizes(type_obj)
cursor.execute("insert into sometable values (:1)", [None])
