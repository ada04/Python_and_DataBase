# under construction

# 23. Параллельное программирование с использованием asyncio и конвейеризации
Параллельное программирование с использованием asyncio и Oracle Database Pipelining значительно повышает общую производительность и скорость реагирования приложений.

23.1 Параллельное программирование с помощью asyncio
Библиотеку Python для асинхронного ввода-вывода (asyncio) можно использовать с режимом python-oracledb Thin для параллельного программирования. Эта библиотека позволяет выполнять операции параллельно, например, выполнять длительные операции в фоновом режиме, не блокируя остальную часть приложения. С помощью asyncio можно легко писать параллельный код, используя синтаксис asyncи . Полезные советы awaitсм. в документации Python « Разработка с использованием asyncio» .

Асинхронный API python-oracledb является частью стандартного модуля python-oracledb. Все синхронные методы, требующие обращения к базе данных, имеют соответствующие асинхронные аналоги. Вы можете выбрать, использовать ли в коде синхронный или асинхронный API. Рекомендуется не использовать оба API одновременно в вашем приложении.

Классы асинхронного API — это AsyncConnection , AsyncConnectionPool , AsyncCursor и AsyncLOB .

В отличие от синхронных, асинхронные соединения и курсоры не закрываются автоматически по завершении действия. Эти асинхронные ресурсы должны быть либо явно закрыты, либо изначально созданы через блок менеджера контекста with .

Примечание

Параллельное программирование с использованием asyncio поддерживается только в тонком режиме python-oracledb.

23.1.1 Асинхронное подключение к базе данных Oracle
С помощью python-oracledb можно создать асинхронное подключение к Oracle Database, используя как отдельные соединения , так и объединённые соединения . (Обсуждение синхронного программирования см. в разделе Подключение к Oracle Database .)

23.1.1.1. Отдельные соединения
Автономные соединения полезны для приложений, которым требуется только одно соединение с базой данных.

Асинхронное автономное соединение можно создать, вызвав метод asynchronous oracledb.connect_async(), который устанавливает соединение с базой данных и возвращает объект AsyncConnection . После создания соединения все объекты, созданные этими соединениями, следуют модели асинхронного программирования. При условии правильного использования awaitдля вызовов, требующих кругового обращения к базе данных, асинхронные соединения используются так же, как синхронные программы используют Standalone Connections .

Асинхронные соединения следует разрывать, когда они больше не нужны, чтобы обеспечить корректную очистку Oracle Database. Предпочтительным методом является использование асинхронного менеджера контекста. Например:

import asyncio
import oracledb

async def main():

    async with oracledb.connect_async(user="hr", password=userpwd,
                                      dsn="localhost/orclpdb") as connection:
        with connection.cursor() as cursor:
            await cursor.execute("select user from dual")
            async for result in cursor:
                print(result)

asyncio.run(main())
Этот код гарантирует, что после завершения блока соединение будет закрыто, а ресурсы будут возвращены базе данных. Кроме того, любая попытка использовать переменную connectionвне блока завершится неудачей.

Если вы не используете менеджер контекста, вам следует явно закрывать соединения, когда они больше не нужны, например:

connection = await oracle.connect_async(user="hr", password=userpwd,
                                        dsn="localhost/orclpdb")

cursor = connection.cursor()

await cursor.execute("select user from dual")
async for result in cursor:
    print(result)

cursor.close()
await connection.close()
Обратите внимание, что асинхронные соединения не закрываются автоматически по завершении действия. Это отличается от поведения синхронных соединений.

23.1.1.2. Пулы соединений
Пулы соединений позволяют приложениям создавать и поддерживать пул открытых подключений к базе данных. Пулы соединений важны для производительности и масштабируемости, когда приложениям необходимо обрабатывать большое количество пользователей, которые работают с базой данных в течение коротких периодов времени, но имеют относительно длительные периоды, когда соединения не нужны. Высокая доступность пулов также делает небольшие пулы полезными для приложений, которым требуется несколько подключений для нечастого использования, и которые должны быть готовы к использованию сразу после их получения.

Асинхронный пул соединений можно создать, вызвав метод oracledb.create_pool_async(), который возвращает объект AsyncConnectionPool . Обратите внимание, что этот метод является синхронным и не использует await. После создания пула ваше приложение может получить из него соединение, вызвав AsyncConnectionPool.acquire(). После того, как приложение использовало соединение, его следует вернуть в пул, чтобы сделать доступным для других пользователей. Это можно сделать, явно закрыв соединение или используя асинхронный менеджер контекста, например:

import asyncio
import oracledb

async def main():

    pool = oracle.create_pool_async(user="hr", password=userpwd,
                                    dsn="localhost/orclpdb",
                                    min=1, max=4, increment=1)

    async with pool.acquire() as connection:
        with connection.cursor() as cursor:
            await cursor.execute("select user from dual")
            async for result in cursor:
                print(result)

    await pool.close()

asyncio.run(main())
23.1.2 Выполнение SQL с использованием асинхронных методов
В этом разделе рассматривается выполнение SQL с использованием модели асинхронного программирования. Подробнее о синхронном программировании см. в разделе « Выполнение SQL» .

Ваше приложение взаимодействует с базой данных Oracle, выполняя операторы SQL. Такие операторы, как запросы (оператор, начинающиеся с SELECT или WITH), язык манипулирования данными (DML) и язык определения данных (DDL), выполняются с использованием асинхронных методов AsyncCursor.execute()или AsyncCursor.executemany(). Строки можно итерировать или извлекать с помощью одного из методов AsyncCursor.fetchone(), AsyncCursor.fetchone(), AsyncCursor.fetchmany(), или AsyncCursor.fetchall(). Обратите внимание, что явно открытые асинхронные курсоры не закрываются автоматически по завершении области действия. Это отличается от синхронного поведения. Асинхронные курсоры должны быть либо явно закрыты, либо изначально созданы с помощью блока менеджера контекста with .

Вы также можете использовать сокращённые методы для объекта API: AsyncConnection Objects, например AsyncConnection.execute()или AsyncConnection.executemany(). Строки можно извлекать с помощью одного из сокращённых методов AsyncConnection.fetchone(), AsyncConnection.fetchmany(), AsyncConnection.fetchall(), AsyncConnection.fetch_df_all()или AsyncConnection.fetch_df_batches().

Пример использования AsyncConnection.fetchall():

import asyncio
import oracledb

async def main():

    async with oracledb.connect_async(user="hr", password=userpwd,
                                      dsn="localhost/orclpdb") as connection:
        res = await connection.fetchall("select * from locations")
        print(res)

asyncio.run(main())
Пример, который использует asyncio для распараллеливания и показывает выполнение нескольких сопрограмм:

import asyncio
import oracledb

# Number of coroutines to run
CONCURRENCY = 5

# Query the unique session identifier/serial number combination of a connection
SQL = """SELECT UNIQUE CURRENT_TIMESTAMP AS CT, sid||'-'||serial# AS SIDSER
         FROM V$SESSION_CONNECT_INFO
         WHERE sid = SYS_CONTEXT('USERENV', 'SID')"""

# Show the unique session identifier/serial number of each connection that the
# pool opens
async def init_session(connection, requested_tag):
    res = await connection.fetchone(SQL)
    print(res[0].strftime("%H:%M:%S.%f"), '- init_session with SID-SERIAL#', res[1])

# The coroutine simply shows the session identifier/serial number of the
# connection returned by the pool.acquire() call
async def query(pool):
    async with pool.acquire() as connection:
        await connection.callproc("dbms_session.sleep", [1])
        res = await connection.fetchone(SQL)
        print(res[0].strftime("%H:%M:%S.%f"), '- query with SID-SERIAL#', res[1])

async def main():

    pool = oracledb.create_pool_async(user="hr", password=userpwd,
                                      dsn="localhost/orclpdb",
                                      min=1, max=CONCURRENCY,
                                      session_callback=init_session)

    coroutines = [ query(pool) for i in range(CONCURRENCY) ]

    await asyncio.gather(*coroutines)

    await pool.close()

asyncio.run(main())
При запуске вы увидите, что открыто и используется несколько соединений (идентифицированных уникальной комбинацией идентификатора сеанса и серийного номера) query(). Например:

12:09:29.711525 - init_session with SID-SERIAL# 36-38096
12:09:29.909769 - init_session with SID-SERIAL# 33-56225
12:09:30.085537 - init_session with SID-SERIAL# 14-31431
12:09:30.257232 - init_session with SID-SERIAL# 285-40270
12:09:30.434538 - init_session with SID-SERIAL# 282-32608
12:09:30.730166 - query with SID-SERIAL# 36-38096
12:09:30.933957 - query with SID-SERIAL# 33-56225
12:09:31.115008 - query with SID-SERIAL# 14-31431
12:09:31.283593 - query with SID-SERIAL# 285-40270
12:09:31.457474 - query with SID-SERIAL# 282-32608
Ваши результаты могут отличаться в зависимости от скорости окружающей среды.

Смотрите async_gather.py для работающего примера.

23.1.3 Управление транзакциями с использованием асинхронных методов
В этом разделе рассматривается управление транзакциями с использованием модели асинхронного программирования. Подробнее о синхронном программировании см. в разделе « Управление транзакциями» .

При выполнении SQL-оператора AsyncCursor.execute()или AsyncCursor.executemany() транзакция запускается или продолжается. По умолчанию python-oracledb не фиксирует эту транзакцию в базе данных. Для явного фиксирования или отката транзакции можно использовать AsyncConnection.commit()следующие методы:AsyncConnection.rollback()

async def main():
    async with oracledb.connect_async(user="hr", password=userpwd,
                                      dsn="localhost/orclpdb") as connection:

        with connection.cursor as cursor:
            await cursor.execute("INSERT INTO mytab (name) VALUES ('John')")
            await connection.commit()
Когда соединение с базой данных закрывается, например, с помощью AsyncConnection.close(), или когда переменные, ссылающиеся на соединение, выходят из области действия, любая незафиксированная транзакция будет откачена.

Альтернативный способ фиксации — установить атрибут AsyncConnection.autocommitсоединения в значение True. Это гарантирует фиксацию всех операторов DML (INSERT, UPDATE и т. д.) при их выполнении.

Обратите внимание, что независимо от значения autocommit, Oracle Database всегда фиксирует открытую транзакцию при выполнении оператора DDL.

При выполнении нескольких операторов DML, составляющих одну транзакцию, рекомендуется использовать режим автоподтверждения только для последнего оператора DML в последовательности операций. Ненужное подтверждение приводит к дополнительной нагрузке на базу данных и может нарушить согласованность транзакций.

23.1.3.1 Управление транзакциями без сеанса с использованием асинхронных методов
При AsyncConnection.begin_sessionless_transaction()выполнении с использованием идентификатора транзакции, выбранного пользователем или сгенерированного python-oracledb, запускается бессеансовая транзакция. После запуска все SQL-операторы выполняются в рамках этой бессеансовой транзакции. Используется AsyncConnection.suspend_sessionless_transaction()для явной приостановки активной транзакции после завершения операций с базой данных. Это освобождает соединение, которое может использоваться другим пользователем, пока транзакция остаётся открытой, и может быть возобновлено позже с помощью соединения, использующего AsyncConnection.resume_sessionless_transaction(). Методы AsyncConnection.commit()и AsyncConnection.rollback()можно использовать для явного подтверждения или отката транзакции. Например:

async def main():
    txn_id = b"new_sessionless_txn"
    # Begin and suspend a sessionless transaction in a connection
    async with oracledb.connect_async(user="hr", password=userpwd,
                                      dsn="localhost/orclpdb") as connection1:
        await connection1.begin_sessionless_transaction(transaction_id=txn_id, timeout=120)
        await connection1.execute("INSERT INTO mytab (name) VALUES ('John')")
        await connection1.suspend_sessionless_transaction()

    # Resume the sessionless transaction in another connection
    async with oracledb.connect_async(user="hr", password=userpwd,
                                      dsn="localhost/orclpdb") as connection2:
        await connection2.resume_sessionless_transaction(transaction_id=txn_id)
        await connection2.commit()
23.2 Конвейеризация операций с базой данных
Конвейеризация позволяет приложению отправлять несколько независимых операторов в Oracle Database одним вызовом. База данных может быть загружена, не дожидаясь, пока приложение получит набор результатов и отправит следующий оператор. Пока база данных обрабатывает конвейер операторов, приложение может продолжать выполнять задачи, не связанные с базой данных. После того, как база данных выполнит все конвейерные операции, их результаты возвращаются приложению.

Конвейерные операции выполняются базой данных последовательно. Они не выполняются параллельно. Локальные задачи могут выполняться одновременно с работой базы данных.

Эффективное использование Oracle Database Pipelining может повысить скорость отклика приложения и общую производительность системы. Конвейеризация полезна, когда множество небольших операций выполняется быстро и последовательно. Она наиболее выгодна при медленном сетевом подключении к базе данных. Это объясняется сокращением числа циклов передачи данных по сравнению с теми, которые потребовались бы при индивидуальном выполнении эквивалентных SQL-операторов с помощью вызовов типа AsyncCursor.execute().

Конвейеризация поддерживается только в тонком режиме python-oracledb с asyncio .

Дополнительные сведения о конвейеризации Oracle Database см. в разделе Oracle Call Interface Pipelining .

Примечание

Настоящая конвейеризация происходит только при подключении к Oracle AI Database 26ai или более поздней версии.

При подключении к старой базе данных операции выполняются последовательно python-oracledb. Каждая операция завершается до отправки следующей в базу данных. Это не приводит к сокращению числа циклов передачи данных и не повышает производительность. Такое использование рекомендуется только для обеспечения переносимости кода, например, при подготовке к обновлению базы данных.

23.2.1 Использование конвейеров
Чтобы создать конвейер для обработки набора операций с базой данных, используйте oracledb.create_pipeline().

pipeline = oracledb.create_pipeline()
Затем вы можете добавлять различные операции в конвейер, используя add_callfunc(), add_callproc(), add_commit(), add_execute(), add_executemany(), add_fetchall(), add_fetchmany()и add_fetchone(). Например:

pipeline.add_execute("insert into mytable (mycol) values (1234)")
pipeline.add_fetchone("select user from dual")
pipeline.add_fetchmany("select employee_id from employees", num_rows=20)
Обратите внимание, что запросы, возвращающие результаты, не вызывают add_execute().

Из каждой операции запроса может быть возвращён только один набор результатов. Например, add_fetchmany()будет выбран только первый набор записей запроса, вплоть до предела, указанного параметром метода num_rows . Аналогично, add_fetchone()может быть выбрана только первая строка. Извлечь дополнительные данные из этих операций невозможно. Чтобы предотвратить обработку базой данных строк, которые приложение не может извлечь, рассмотрите возможность добавления соответствующих WHEREусловий или использования предложения в операторе (см. раздел Ограничение строк) .FETCH NEXT

Результаты запроса или OUT-привязки из одной операции не могут быть переданы последующим операциям в том же конвейере.

Чтобы выполнить конвейер, вызовите AsyncConnection.run_pipeline().

results = await connection.run_pipeline(pipeline)
Все операции отправляются в базу данных и выполняются. Метод возвращает список объектов PipelineOpResult , по одному элементу на операцию. Эти объекты содержат информацию о выполнении соответствующей операции, например, номер ошибки, возвращаемое значение функции PL/SQL или метаданные строк и столбцов запроса.

Это Connection.call_timeoutзначение не влияет на работу конвейера. Чтобы ограничить время выполнения конвейера, используйте asyncio timeout , доступный с версии Python 3.11.

Чтобы настроить выборку строк с помощью Pipeline.add_fetchall(), задайте oracledb.defaults.arraysizeили передайте arraysizeпараметр.

23.2.1.1 Примеры конвейеризации
Пример конвейеризации:

import asyncio
import oracledb

async def main():
    # Create a pipeline and define the operations
    pipeline = oracledb.create_pipeline()
    pipeline.add_fetchone("select temperature from weather")
    pipeline.add_fetchall("select name from friends where active = true")
    pipeline.add_fetchmany("select story from news order by popularity", num_rows=5)

    connection = await oracle.connect_async(user="hr", password=userpwd,
                                            dsn="localhost/orclpdb")

    # Run the operations in the pipeline
    result_1, result_2, result_3 = await connection.run_pipeline(pipeline)

    # Print the database responses
    print("Current temperature:", result_1.rows)
    print("Active friends:", result_2.rows)
    print("Top news stories:", result_3.rows)

    await connection.close()

asyncio.run(main())
Смотрите файл pipelining_basic.py для работающего примера.

Чтобы разрешить приложению продолжить работу вне базы данных до обработки каких-либо ответов из базы данных, используйте код, аналогичный следующему:

async def run_thing_one():
    return "thing_one"

async def run_thing_two():
    return "thing_two"

async def main():
    connection = await oracle.connect_async(user="hr", password=userpwd,
                                            dsn="localhost/orclpdb")

    pipeline = oracledb.create_pipeline()
    pipeline.add_fetchone("select user from dual")
    pipeline.add_fetchone("select sysdate from dual")

    # Run the pipeline and non-database operations concurrently
    return_values = await asyncio.gather(
        run_thing_one(), run_thing_two(), connection.run_pipeline(pipeline)
    )

    for r in return_values:
        if isinstance(r, list):  # the pipeline return list
            for result in r:
                if result.rows:
                    for row in result.rows:
                        print(*row, sep="\t")
        else:
            print(r)             # a local operation result

    await connection.close()

asyncio.run(main())
Вывод будет таким:

thing_one
thing_two
HR
2024-10-29 03:34:43
Смотрите файл pipelining_parallel.py для работающего примера.

23.2.2 Использование исходящих привязок с конвейерами
Чтобы получить привязки OUT из выполненных операторов, создайте явный курсор и используйте Cursor.var(). Эти переменные связаны с соединением и могут использоваться другими курсорами, созданными внутри для каждой конвейерной операции. Например:

cursor = connection.cursor()
v1 = cursor.var(oracledb.DB_TYPE_BOOLEAN)
v2 = cursor.var(oracledb.DB_TYPE_VARCHAR)

pipeline = oracledb.create_pipeline()

pipeline.add_execute("""
    begin
      :1 := true;
      :2 := 'Python';
    end;
    """, [v1, v2])
pipeline.add_fetchone("select 1234 from dual")

results = await connection.run_pipeline(pipeline)

for r in results:
    if r.rows:
        print(r.rows)

print(v1.getvalue(), v2.getvalue())
Это печатает:

[(1234,)]
True Python
Связывания OUT из одной операции не могут использоваться в последующих операциях. Например, следующий код будет выведен только Trueпотому, что условие WHERE оператора SQL не выполнено:

cursor = connection.cursor()
v1 = cursor.var(oracledb.DB_TYPE_BOOLEAN)

pipeline = oracledb.create_pipeline()

pipeline.add_execute("""
    begin
      :1 := TRUE;
    end;
    """, [v1])
pipeline.add_fetchone("select 1234 from dual where :1 = TRUE", [v1])

results = await connection.run_pipeline(pipeline)

for r in results:
    if r.rows:
        print(r.rows)

print(v1.getvalue())  # prints True
23.2.3 Обработка ошибок конвейера
Параметр continue_on_errorto AsyncConnection.run_pipeline() определяет, следует ли продолжать выполнение последующих операций после сбоя одной из них. Если установлено значение по умолчанию False, то при возникновении ошибки в любой операции в конвейере база данных завершает все последующие операции.

Например:

# Stop on error

pipeline.add_fetchall("select 1234 from does_not_exist")
pipeline.add_fetchone("select 5678 from dual")

r1, r2 = await connection.run_pipeline(pipeline)
выполнит только первую операцию и выдаст сообщение об ошибке:

oracledb.exceptions.DatabaseError: ORA-00942: table or view "HR"."DOES_NOT_EXIST" does not exist
Help: https://docs.oracle.com/error-help/db/ora-00942/
тогда как этот код:

# Continue on error

pipeline.add_fetchall("select 1234 from does_not_exist")
pipeline.add_fetchone("select 5678 from dual")

r1, r2 = await connection.run_pipeline(pipeline, continue_on_error=True)

print(r1.error)
print(r2.rows)
выполнит все операции и отобразит:

ORA-00942: table or view "HR"."DOES_NOT_EXIST" does not exist
Help: https://docs.oracle.com/error-help/db/ora-00942/
[(5678,)]
Предупреждения компиляции PL/SQL

Предупреждения компиляции PL/SQL можно определить, проверив атрибут PipelineOpResult PipelineOpResult.warning . Например:

pipeline.add_execute(
    """create or replace procedure myproc as
       begin
          bogus;
       end;"""
)
(result,) = await connection.run_pipeline(pipeline)

print(result.warning.full_code)
print(result.warning)
напечатает:

DPY-7000
DPY-7000: creation succeeded with compilation errors
См. файл pipelining_error.py для работающего примера, показывающего предупреждения и ошибки.

23.2.4 Использование конвейерного курсора
Для каждой операции, добавленной в конвейер, за исключением Pipeline.add_commit(), при вызове будет открыт курсор AsyncConnection.run_pipeline(). Например, следующий код откроет два курсора:

pipeline = oracledb.create_pipeline()
pipeline.add_execute("insert into t1 (c1) values (1234)")
pipeline.add_fetchone("select user from dual")

await connection.run_pipeline(pipeline)
Убедитесь, что длина конвейера не превышает лимит курсора. Правильно настройте параметр базы данных open_cursors .

23.2.5. Круговые маршруты по трубопроводу
Полный набор операций в конвейере будет выполнен за один цикл обработки при AsyncConnection.run_pipeline()вызове , за следующими исключениями:

Запросы, содержащие LOB-объекты , требуют дополнительного кругового обхода

Запросы, содержащие значения DbObject , могут потребовать нескольких циклов обработки.

Запросы add_fetchall()могут потребовать нескольких циклов обработки.

Сокращение числа циклов передачи данных (round-trip) существенно повышает производительность конвейеризации по сравнению с явным выполнением эквивалентных SQL-операторов по отдельности. В высокоскоростных сетях конвейеризация может дать небольшой выигрыш в производительности, однако эффективность базы данных и сети может повысить общую масштабируемость системы.

Обратите внимание, что традиционный метод мониторинга круговых передач путем создания снимков представления V$SESSTAT неточен для конвейеров.
