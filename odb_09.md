# under construction

# 9. Пакетные операции выписки и массового копирования
Python-oracledb идеально подходит для больших операций с данными ETL («извлечение, преобразование, загрузка»).

В этой главе мы рассмотрим эффективный приём данных. Python-oracledb позволяет легко оптимизировать пакетную вставку, а также фильтровать «шумные» данные (значения в неподходящем формате) для анализа, одновременно вставляя другие, правильные значения.

Помимо пакетной загрузки Oracle Database «Array DML», для очень быстрой загрузки больших наборов данных можно использовать функцию Direct Path Loads при соблюдении определённых критериев схемы. Другой вариант для частых небольших вставок — загрузка данных с помощью Oracle Database Memoptimized Rowstore .

Смежные темы включают настройку python-oracledb и работу с фреймами данных .

9.1. Пакетное выполнение операторов
Вставка, обновление или удаление нескольких строк может быть эффективно выполнена с помощью Cursor.executemany()python-oracledb, что упрощает работу с большими наборами данных в python-oracledb. Этот метод может значительно превосходить по производительности многократные вызовы, Cursor.execute()снижая затраты на передачу данных по сети и накладные расходы на базу данных. Этот executemany()метод также можно использовать для многократного выполнения оператора PL/SQL за один вызов.

Примеры можно найти в каталоге примеров GitHub .

В представленных ниже примерах будут использоваться следующие таблицы:

create table ParentTable (
    ParentId              number(9) not null,
    Description           varchar2(60) not null,
    constraint ParentTable_pk primary key (ParentId)
);

create table ChildTable (
    ChildId               number(9) not null,
    ParentId              number(9) not null,
    Description           varchar2(60) not null,
    constraint ChildTable_pk primary key (ChildId),
    constraint ChildTable_fk foreign key (ParentId)
            references ParentTable
);
9.1.1 Пакетное выполнение SQL
В следующем примере в таблицу вставляются пять строк ParentTable:

data = [
    (10, "Parent 10"),
    (20, "Parent 20"),
    (30, "Parent 30"),
    (40, "Parent 40"),
    (50, "Parent 50")
]
cursor.executemany("insert into ParentTable values (:1, :2)", data)
Каждое значение кортежа сопоставляется с одним из заполнителей переменных привязки.

Этот код требует только одного кругового пути от клиента к базе данных вместо пяти круговых путей, которые потребовались бы при повторных вызовах execute().

Чтобы вставить один столбец, убедитесь, что переменные привязки правильно созданы как кортежи, например:

data = [
    (10,),
    (20,),
    (30,),
]
cursor.executemany('insert into mytable (mycol) values (:1)', data)
Именованные привязки можно выполнить, передав массив словарей, где ключи соответствуют именам заполнителей переменных привязки:

data = [
    {"pid": 10, "pdesc": "Parent 10"},
    {"pid": 20, "pdesc": "Parent 20"},
    {"pid": 30, "pdesc": "Parent 30"},
    {"pid": 40, "pdesc": "Parent 40"},
    {"pid": 50, "pdesc": "Parent 50"}
]
cursor.executemany("insert into ParentTable values :pid, :pdesc)", data)
В качестве альтернативы можно передать фрейм данных Cursor.executemany(), см. раздел Вставка фреймов данных .

9.1.2. Предопределение областей памяти
При обработке нескольких строк данных существует вероятность, что данные будут неоднородны по типу и размеру. В таких случаях python-oracledb прилагает некоторые усилия для компенсации таких различий. Определение типа для каждого столбца откладывается до тех пор, пока Noneв данных столбца не будет найдено значение, которое не является . Если все значения в определенном столбце имеют тип None, то python-oracledb предполагает, что тип — строка длиной 1. Python-oracledb также корректирует размер буферов, используемых для хранения строк и байтов, при обнаружении в данных более длинных значений. Подобные операции влекут за собой накладные расходы, поскольку необходимо перераспределять память и копировать данные. Чтобы устранить эти накладные расходы, using setinputsizes()сообщает python-oracledb тип и размер данных, которые будут использоваться.

Рассмотрим следующий код:

data = [
    (110, "Parent 110"),
    (2000, "Parent 2000"),
    (30000, "Parent 30000"),
    (400000, "Parent 400000"),
    (5000000, "Parent 5000000")
]
cursor.setinputsizes(None, 20)
cursor.executemany("""
        insert into ParentTable (ParentId, Description)
        values (:1, :2)""", data)
Если в этом примере не вызывается setinputsizes(), то python-oracledb выполняет пять выделений памяти увеличивающегося размера и копирует данные по мере обнаружения каждой новой, более длинной строки. Однако сообщает python-oracledb, что максимальный размер обрабатываемых строк составляет 20 символов. Первый параметр сообщает python-oracledb, что обработки по умолчанию будет достаточно, поскольку числовые данные уже эффективно хранятся. Поскольку python-oracledb выделяет память для каждой строки на основе предоставленных значений, не превышайте их размер.cursor.setinputsizes(None, 20)None

Если размер буферов, выделенных для любого из значений привязки, превысит 2 ГБ, возникнет ошибка , где <n> меняется в зависимости от размера каждого элемента, выделяемого в буфере. Если возникает эта ошибка, уменьшите количество вставляемых строк.DPI-1015: array size of <n> is too large

При использовании именованных переменных связывания используйте именованные параметры при вызове setinputsizes():

data = [
    {"pid": 110, "pdesc": "Parent 110"},
    {"pid": 2000, "pdesc": "Parent 2000"},
    {"pid": 30000, "pdesc": "Parent 30000"},
    {"pid": 400000, "pdesc": "Parent 400000"},
    {"pid": 5000000, "pdesc": "Parent 5000000"}
]
cursor.setinputsizes(pdesc=20)
cursor.executemany("""
        insert into ParentTable (ParentId, Description)
        values (:pid, :pdesc)""", data)
9.1.3 Пакетирование больших наборов данных
Для очень больших наборов данных может существовать ограничение на количество обрабатываемых строк, обусловленное буфером или сетью. Это ограничение зависит как от количества записей, так и от размера каждой обрабатываемой записи. В других случаях обработка меньших наборов записей может быть быстрее.

Чтобы уменьшить объём данных, можно либо многократно вызывать функцию , executemany()как показано далее в примерах CSV, либо использовать batch_sizeпараметр для оптимизации передачи данных по сети в базу данных. Например:

data = [
    (1, "Parent 1"),
    (2, "Parent 2"),
    . . .
    (9_999_999, "Parent 9,999,999"),
    (10_000_000, "Parent 10,000,000"),

]

cursor.executemany("insert into ParentTable values (:1, :2)", data, batch_size=200_000)
Данные будут отправляться в базу данных партиями по 200 000 записей, пока не будут вставлены все 10 000 000 записей.

Если Connection.autocommitравно True, то фиксация будет производиться для каждого пакета обработанных записей.

9.1.4 Пакетное выполнение PL/SQL
Использование executemany()может повысить производительность, когда функции, процедуры или анонимные блоки PL/SQL необходимо вызывать несколько раз.

Примеры выполнения находятся в plsql_batch.py .

В привязке

Пример использования привязки по положению для привязок IN:

data = [
    (10, "Parent 10"),
    (20, "Parent 20"),
    (30, "Parent 30"),
    (40, "Parent 40"),
    (50, "Parent 50")
]
cursor.executemany("begin mypkg.create_parent(:1, :2); end;", data)
Обратите внимание, что batcherrorsпараметр executemany() (обсуждаемый в разделе Обработка ошибок данных ) нельзя использовать при выполнении блока PL/SQL.

Связывания OUT

При использовании OUT-связываний в PL/SQL входные данные не содержат записей для плейсхолдеров переменных OUT-связывания. Пример процедуры PL/SQL, возвращающей OUT-связывания:

create or replace procedure myproc(p1 in number, p2 out number) as
begin
    p2 := p1 * 2;
end;
Это можно вызвать в python-oracledb, используя позиционные привязки, например:

data = [
    (100,),
    (200,),
    (300,)
]

outvals = cursor.var(oracledb.DB_TYPE_NUMBER, arraysize=len(data))
cursor.setinputsizes(None, outvals)

cursor.executemany("begin myproc(:1, :2); end;", data)
print(outvals.values)
Вывод такой:

[200, 400, 600]
Эквивалентный код, использующий именованные привязки:

data = [
    {"p1bv": 100},
    {"p1bv": 200},
    {"p1bv": 300}
]

outvals = cursor.var(oracledb.DB_TYPE_NUMBER, arraysize=len(data))
cursor.setinputsizes(p1bv=None, p2bv=outvals)

cursor.executemany("begin myproc(:p1bv, :p2bv); end;", data)
print(outvals.values)
Обратите внимание, что в режиме python-oracledb Thick при executemany()использовании для кода PL/SQL, возвращающего привязки OUT, он будет иметь те же характеристики производительности, что и повторные вызовы execute().

Связывания ВХОД/ВЫХОД

Пример процедуры PL/SQL, которая возвращает привязки IN/OUT:

create or replace procedure myproc2 (p1 in number, p2 in out varchar2) as
begin
    p2 := p2 || ' ' || p1;
end;
Это можно вызвать в python-oracledb, используя позиционные привязки, например:

data = [
    (440, 'Gregory'),
    (550, 'Haley'),
    (660, 'Ian')
]
outvals = cursor.var(oracledb.DB_TYPE_VARCHAR, size=100, arraysize=len(data))
cursor.setinputsizes(None, outvals)

cursor.executemany("begin myproc2(:1, :2); end;", data)
print(outvals.values)
Параметр sizeуказывает Cursor.var()максимальную длину строки, которая может быть возвращена.

Выходные данные:

['Gregory 440', 'Haley 550', 'Ian 660']
Эквивалентный код, использующий именованные привязки:

data = [
    {"p1bv": 440, "p2bv": 'Gregory'},
    {"p1bv": 550, "p2bv": 'Haley'},
    {"p1bv": 660, "p2bv": 'Ian'}
]
outvals = cursor.var(oracledb.DB_TYPE_VARCHAR, size=100, arraysize=len(data))
cursor.setinputsizes(p1bv=None, p2bv=outvals)

cursor.executemany("begin myproc2(:p1bv, :p2bv); end;", data)
print(outvals.values)
9.1.5 Обработка ошибок данных
Большие наборы данных могут содержать некоторые недопустимые данные. При использовании пакетного выполнения, как обсуждалось выше, весь пакет будет отброшен при обнаружении одной ошибки, что потенциально сводит на нет преимущества пакетного выполнения в производительности и увеличивает сложность кода, необходимого для обработки этих ошибок. Однако, если параметру при вызове batchErrorsзадано значение , обработка будет продолжена, даже если в некоторых строках есть ошибки данных, а строки с ошибками можно будет впоследствии проверить, чтобы определить, как следует действовать приложению. Обратите внимание, что при обнаружении каких-либо ошибок транзакция будет запущена, но не зафиксирована, даже если для установлено значение . После изучения ошибок и принятия решения о дальнейших действиях приложение должно явно зафиксировать или откатить транзакцию с помощью или , по мере необходимости.Trueexecutemany()Connection.autocommitTrueConnection.commit()Connection.rollback()

В этом примере показано, как можно выявить ошибки данных:

data = [
    (60, "Parent 60"),
    (70, "Parent 70"),
    (70, "Parent 70 (duplicate)"),
    (80, "Parent 80"),
    (80, "Parent 80 (duplicate)"),
    (90, "Parent 90")
]
cursor.executemany("insert into ParentTable values (:1, :2)", data,
                   batcherrors=True)
for error in cursor.getbatcherrors():
    print("Error", error.message, "at row offset", error.offset)
Вывод такой:

Error ORA-00001: unique constraint (PYTHONDEMO.PARENTTABLE_PK) violated at row offset 2
Error ORA-00001: unique constraint (PYTHONDEMO.PARENTTABLE_PK) violated at row offset 4
Смещение строки — это индекс в массиве данных, которые не удалось вставить из-за ошибок. Приложение может зафиксировать или откатить остальные строки, которые были успешно вставлены. В качестве альтернативы, оно может исправить данные для двух недопустимых строк и попытаться вставить их снова перед фиксацией.

9.1.6. Определение затронутых строк
При выполнении оператора DML с использованием execute()количество затронутых строк можно узнать, посмотрев на атрибут rowcount. При пакетном выполнении с использованием Cursor.executemany()количество строк вернет общее количество затронутых строк. Чтобы узнать общее количество строк, затронутых каждой строкой привязанных данных, необходимо установить параметр arraydmlrowcountsв True, как показано ниже:

parent_ids_to_delete = [20, 30, 50]
cursor.executemany("delete from ChildTable where ParentId = :1",
                   [(i,) for i in parent_ids_to_delete],
                   arraydmlrowcounts=True)
row_counts = cursor.getarraydmlrowcounts()
for parent_id, count in zip(parent_ids_to_delete, row_counts):
    print("Parent ID:", parent_id, "deleted", count, "rows.")
Используя данные, найденные в примерах GitHub, вывод будет следующим:

Parent ID: 20 deleted 3 rows.
Parent ID: 30 deleted 2 rows.
Parent ID: 50 deleted 4 rows.
9.1.7. ВОЗВРАТ DML
Такие операторы DML, как INSERT, UPDATE, DELETE и MERGE, могут возвращать значения, используя синтаксис DML RETURNING. Для приема этих данных можно создать переменную связывания. Подробнее см. в разделе Использование переменных связывания .

Если вместо того, чтобы просто удалить строки, как показано в предыдущем примере, вы также хотите узнать некоторую информацию о каждой из удаленных строк, вы можете использовать следующий код:

parent_ids_to_delete = [20, 30, 50]
child_id_var = cursor.var(int, arraysize=len(parent_ids_to_delete))
cursor.setinputsizes(None, child_id_var)
cursor.executemany("""
        delete from ChildTable
        where ParentId = :1
        returning ChildId into :2""",
        [(i,) for i in parent_ids_to_delete])
for ix, parent_id in enumerate(parent_ids_to_delete):
    print("Child IDs deleted for parent ID", parent_id, "are",
          child_id_var.getvalue(ix))
Результат будет следующим:

Child IDs deleted for parent ID 20 are [1002, 1003, 1004]
Child IDs deleted for parent ID 30 are [1005, 1006]
Child IDs deleted for parent ID 50 are [1012, 1013, 1014, 1015]
Обратите внимание, что переменная привязки, создаваемая для приёма возвращаемых данных, должна иметь размер массива, достаточный для хранения данных для каждой обрабатываемой строки. Кроме того, вызов привязывает Cursor.setinputsizes()эту переменную немедленно, поэтому её не нужно передавать в каждой строке данных.

9.2 Операции массового копирования
Операции массового копирования облегчаются благодаря использованию Cursor.executemany()соответствующих операторов SQL и модулей Python.

См. также разделы Работа с фреймами данных и Конвейеризация базы данных Oracle .

9.2.1 Загрузка CSV-файлов в базу данных Oracle
Этот метод и модульCursor.executemany() Python csv можно использовать для эффективной вставки CSV-данных (значений, разделенных запятыми). Например, рассмотрим файл :data.csv

101,Abel
154,Baker
132,Charlie
199,Delta
. . .
И схема:

create table test (id number, name varchar2(25));
Загрузку данных можно выполнять пакетами записей, поскольку ограничения памяти Python могут помешать сохранению всех записей в памяти одновременно:

import oracledb
import csv

# CSV file
FILE_NAME = 'data.csv'

# Adjust the number of rows to be inserted in each iteration
# to meet your memory and performance requirements
BATCH_SIZE = 10000

connection = oracledb.connect(user="hr", password=userpwd,
                              dsn="dbhost.example.com/orclpdb")

with connection.cursor() as cursor:

    # Predefine the memory areas to match the table definition.
    # This can improve performance by avoiding memory reallocations.
    # Here, one parameter is passed for each of the columns.
    # "None" is used for the ID column, since the size of NUMBER isn't
    # variable.  The "25" matches the maximum expected data size for the
    # NAME column
    cursor.setinputsizes(None, 25)

    with open(FILE_NAME, 'r') as csv_file:
        csv_reader = csv.reader(csv_file, delimiter=',')
        sql = "insert into test (id, name) values (:1, :2)"
        data = []
        for line in csv_reader:
            data.append((line[0], line[1]))
            if len(data) % BATCH_SIZE == 0:
                cursor.executemany(sql, data)
                data = []
        if data:
            cursor.executemany(sql, data)
        connection.commit()
В зависимости от размеров данных и бизнес-требований также могут быть полезны изменения в базе данных, такие как временное отключение ведения журнала повторных операций в таблице или отключение индексов.

Методы CSV пакета PyArrow могут быть более эффективными, чем модуль CSV по умолчанию.

См. examples/load_csv.py для работающего примера, демонстрирующего как модуль CSV, так и функциональность CSV PyArrow.

Также следует проверить, подходят ли вашей среде специализированные инструменты и функции загрузки данных Oracle. Они могут быть быстрее, чем использование Python. См. раздел SQL*Loader и внешние таблицы .

9.2.2 Создание CSV-файлов из базы данных Oracle
Модуль csv в Python можно использовать для эффективного создания CSV-файлов (файлов с разделителями-запятыми). Например:

cursor.arraysize = 1000  # tune this for large queries
print(f"Writing to {FILE_NAME}")
with open(FILE_NAME, "w") as f:
    writer = csv.writer(
        f, lineterminator="\n", quoting=csv.QUOTE_NONNUMERIC
    )
    cursor.execute("""select rownum, sysdate, mycol from BigTab""")
    writer.writerow(info.name for info in cursor.description)
    writer.writerows(cursor)
Смотрите examples/write_csv.py для работающего примера.

9.2.3 Массовое копирование данных между базами данных
Функция Cursor.executemany()полезна для копирования данных из одной базы данных в другую, например, в рабочем процессе ETL («Извлечение, преобразование, загрузка»):

# Connect to both databases
source_connection = oracledb.connect(user=un1, password=pw1, dsn=cs1)
target_connection = oracledb.connect(user=un2, password=pw2, dsn=cs2)

# Setup cursors
source_cursor = source_connection.cursor()
source_cursor.arraysize = 1000              # tune this for query performance

target_cursor = target_connection.cursor()
target_cursor.setinputsizes(None, 25)       # set according to column types

# Perform bulk fetch and insertion
source_cursor.execute("select c1, c2 from MySrcTable")
while True:

    # Extract the records
    rows = source_cursor.fetchmany()
    if not rows:
        break

    # Optionally transform the records here
    # ...

    # Load the records into the target database
    target_cursor.executemany("insert into MyDestTable values (:1, :2)", rows)

target_connection.commit()
Это arraysizeзначение определяет количество строк Cursor.fetchmany(), возвращаемых каждым вызовом (см. раздел «Настройка производительности выборки» ). setinputsizes()Вызов используется для оптимизации выделения памяти при вставке ( executemany()см. раздел «Предопределение областей памяти» ). Также может потребоваться настроить параметр SDU для достижения максимальной производительности сети (см. раздел «Настройка python-oracledb» ).

Если вы вставляете данные обратно в ту же базу данных, из которой изначально были получены записи, открывать второе соединение не нужно. Вместо этого оба курсора можно получить из одного соединения.

Избегание копирования данных по сети

При копировании данных в другую таблицу в той же базе данных может быть предпочтительнее использовать INSERT INTO SELECT или CREATE AS SELECT, чтобы избежать накладных расходов на копирование данных в процесс Python и обратно. Это также позволяет избежать любых изменений типов данных. Например, для создания полной копии таблицы:

cursor.execute("create table new_table as select * from old_table")
Аналогично, при копировании в другую базу данных рассмотрите возможность создания ссылки между базами данных и использования INSERT INTO SELECT или CREATE AS SELECT.

Вы можете управлять передачей данных, изменяя оператор SELECT.

9.3. Нагрузки прямого пути
Функция Direct Path Loads позволяет вставлять данные в базу данных Oracle, минуя уровни кода, такие как кэш буфера базы данных. Кроме того, не используются операторы INSERT. Это может быть очень эффективно при загрузке больших объёмов данных, но из-за особенностей архитектуры существуют ограничения на использование Direct Path Loads. Подробнее см. в документации Oracle Database, например, в разделах SQL*Loader Direct Path Loads и Oracle Call Interface Direct Path Load Interface .

Время сквозной вставки при использовании Direct Path Loads для небольших наборов данных может быть не меньше, чем при использовании Cursor.executemany(), однако нагрузка на базу данных все равно может быть снижена.

Примечание

Загрузка по прямому пути поддерживается только в тонком режиме python-oracledb.

Загрузка по прямому пути выполняется методом Connection.direct_path_load() . Например, если у вас есть таблица:

create table TestDirectPathLoad (
    id    number(9),
    name  varchar2(20)
);
Затем вы можете загрузить в него данные с помощью кода:

SCHEMA_NAME = "HR"
TABLE_NAME = "TESTDIRECTPATHLOAD"
COLUMN_NAMES = ["ID", "NAME"]
DATA = [
    (1, "A first row"),
    (2, "A second row"),
    (3, "A third row"),
]

connection.direct_path_load(
    schema_name=SCHEMA_NAME,
    table_name=TABLE_NAME,
    column_names=COLUMN_NAMES,
    data=DATA
)
Записи всегда подразумеваются.

Параметром dataможет быть список последовательностей, объект DataFrame или сторонний экземпляр DataFrame, поддерживающий интерфейс Apache Arrow PyCapsule, см. раздел Вставка фреймов данных с прямой загрузкой по пути .

Для загрузки в столбцы VECTOR передайте соответствующее значение Python array.array() или список значений. Например, если у вас есть таблица:

create table TestDirectPathLoad (
    id    number(9),
    name  varchar2(20),
    v64   vector(3, float64)
);
Затем вы можете загрузить в него данные с помощью кода:

SCHEMA_NAME = "HR"
TABLE_NAME = "TESTDIRECTPATHLOAD"
COLUMN_NAMES = ["ID", "NAME", "V64"]
DATA = [
    (1, "A first row", array.array("d", [1, 2, 3])),
    (2, "A second row", [4, 5, 6]),
    (3, "A third row", array.array("d", [7, 8, 9])),
]

connection.direct_path_load(
    schema_name=SCHEMA_NAME,
    table_name=TABLE_NAME,
    column_names=COLUMN_NAMES,
    data=DATA
)
Более подробную информацию о векторах см. в разделе Использование векторных данных .

Примеры запускаемой прямой загрузки находятся в каталоге примеров GitHub .

Примечания о нагрузках прямого пути

Данные передаются неявно.

Данные, вставляемые в столбцы CLOB или BLOB, должны быть строками или байтами, а не объектами LOB python-oracledb .

Вставка объектов DbObjectType python-oracledb не поддерживается.

Ознакомьтесь с документацией по Oracle Database для получения информации о требованиях и ограничениях базы данных.

9.3.1 Формирование партий грузов прямого пути
Если ограничения буфера, сети или базы данных требуют обработки меньших наборов записей, можно либо повторно вызывать функцию, Connection.direct_path_load()либо использовать batch_size параметр . Например:

SCHEMA_NAME = "HR"
TABLE_NAME = "TESTDIRECTPATHLOAD"
COLUMN_NAMES = ["ID", "NAME"]
DATA = [
    (1, "A first row"),
    (2, "A second row"),
    . . .
    (10_000_000, "Ten millionth row"),
]

connection.direct_path_load(
    schema_name=SCHEMA_NAME,
    table_name=TABLE_NAME,
    column_names=COLUMN_NAMES,
    data=DATA,
    batch_size=1_000_000
)
Данные будут отправляться в базу данных партиями по 1 000 000 записей, пока не будут вставлены все 10 000 000 записей.

9.4 Мемооптимизированное строковое хранилище
Memoptimized Rowstore — ещё одна функция Oracle Database для приёма данных, особенно при частой вставке отдельных строк. Она также может повысить производительность запросов. Конфигурация и управление осуществляются конфигурацией базы данных и использованием специальных SQL-операторов. Таким образом, для использования этой функции не требуется никаких специальных требований к python-oracledb или API.

Чтобы использовать Memoptimized Rowstore, см. документацию Oracle Database « Включение высокопроизводительной потоковой передачи данных с помощью Memoptimized Rowstore» .

