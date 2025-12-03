# Under construction

# 7. Выполнение PL/SQL
PL/SQL — это процедурный язык, используемый для создания пользовательских процедур, функций и анонимных блоков. Программные модули PL/SQL компилируются и выполняются в Oracle Database, что позволяет им эффективно работать с данными. Процедуры и функции можно хранить в базе данных, инкапсулируя бизнес-логику для повторного использования в других приложениях.

Код PL/SQL можно хранить в базе данных и выполнять с помощью python-oracledb.

Примеры в этой главе демонстрируют одиночные вызовы с использованием Cursor.callproc(), Cursor.callfunc(), или Cursor.execute(). Примеры повторных вызовов с использованием Cursor.executemany()приведены в разделе Пакетное выполнение PL/SQL .

Пользовательские процедуры в JavaScript

Вас также может заинтересовать создание пользовательских процедур на JavaScript вместо PL/SQL. См. раздел «Введение в Oracle Database Multilingual Engine для JavaScript» . Эти процедуры можно вызывать в python-oracledb так же, как и в PL/SQL.

7.1. Хранимые процедуры PL/SQL
Метод Cursor.callproc()используется для вызова процедур PL/SQL.

Если существует процедура со следующим определением:

create or replace procedure myproc (
    a_Value1                            number,
    a_Value2                            out number
) as
begin
    a_Value2 := a_Value1 * 2;
end;
то для его вызова можно использовать следующий код Python:

out_val = cursor.var(int)
cursor.callproc('myproc', [123, out_val])
print(out_val.getvalue())        # will print 246
Внутренний вызов Cursor.callproc()генерирует анонимный блок PL/SQL и выполняет его. Это эквивалентно коду приложения:

cursor.execute("begin myproc(:1,:2); end;", [123, out_val])
Информацию о связывании см. в разделе Использование переменных привязки .

7.2 Хранимые функции PL/SQL
Метод Cursor.callfunc()используется для вызова функций PL/SQL.

Первый параметр — callfunc()это имя функции. Второй параметр представляет возвращаемое значение функции PL/SQL и должен иметь тип Python, один из типов Oracledb или тип объекта . Любая последующая последовательность значений или именованных параметров передается как аргументы функции PL/SQL.

Если существует функция PL/SQL со следующим определением:

create or replace function myfunc (
    a_StrVal varchar2,
    a_NumVal number,
    a_Date out date
) return number as
begin
    select sysdate into a_Date from dual;
    return length(a_StrVal) + a_NumVal * 2;
end;
то для его вызова можно использовать следующий код Python:

d = cursor.var(oracledb.DB_TYPE_DATE)   # for the a_Date OUT parameter
return_val = cursor.callfunc("myfunc", int, ["a string", 15, d])
print(return_val)        # prints 38
print(d.getvalue())      # like 2024-12-04 22:35:23
Более сложный пример, возвращающий пространственный (SDO) объект, представлен ниже. Сначала — SQL-операторы, необходимые для настройки примера:

create table MyPoints (
    id number(9) not null,
    point sdo_point_type not null
);

insert into MyPoints values (1, sdo_point_type(125, 375, 0));

create or replace function spatial_queryfn (
    a_Id     number
) return sdo_point_type is
    t_Result sdo_point_type;
begin
    select point
    into t_Result
    from MyPoints
    where Id = a_Id;

    return t_Result;
end;
/
Код Python, который будет вызывать эту процедуру, выглядит следующим образом:

obj_type = connection.gettype("SDO_POINT_TYPE")
cursor = connection.cursor()
return_val = cursor.callfunc("spatial_queryfn", obj_type, [1])
print(f"({return_val.X}, {return_val.Y}, {return_val.Z})")
# will print (125, 375, 0)
Информацию о связывании см. в разделе Использование переменных привязки .

7.3 Анонимные блоки PL/SQL
Анонимный блок PL/SQL можно вызвать, как показано:

var = cursor.var(int)
cursor.execute("""
        begin
            :out_val := length(:in_val);
        end;""", in_val="A sample string", out_val=var)
print(var.getvalue())        # will print 15
Информацию о связывании см. в разделе Использование переменных привязки .

7.4 Передача значений NULL в PL/SQL
Oracle Database требует тип, даже для значений NULL. При передаче значения None python-oracledb предполагает, что тип — строка. Если это не тот тип, который требуется, вы можете указать его явно. Например, чтобы передать NULL- объект Oracle Spatial SDO_GEOMETRY в хранимую процедуру PL/SQL с сигнатурой:

procedure myproc(p in sdo_geometry)
Вы можете использовать:

type_obj = connection.gettype("SDO_GEOMETRY")
var = cursor.var(type_obj)
cursor.callproc("myproc", [var])
7.5 Создание хранимых процедур и пакетов
Для создания хранимых процедур и пакетов PL/SQL используйте Cursor.execute() команду CREATE. Например:

cursor.execute("""
        create or replace procedure myprocedure
        (p_in in number, p_out out number) as
        begin
            p_out := p_in * 2;
        end;""")
7.5.1 Предупреждения компиляции PL/SQL
При создании процедур, функций или типов PL/SQL в python-oracledb оператор может быть выполнен успешно без ошибки, но могут появиться дополнительные информационные сообщения. Эти сообщения в Oracle иногда называются сообщениями «success with info» (успешно с информацией). Если вашему приложению требуется вывод таких сообщений, их необходимо явно искать с помощью Cursor.warning. Последующий запрос к таблице типа python-oracledb USER_ERRORSвыведет более подробную информацию. Например:

with connection.cursor() as cursor:

    cursor.execute("""
            create or replace procedure badproc as
            begin
                WRONG WRONG WRONG
            end;""")

    if cursor.warning and cursor.warning.full_code == "DPY-7000":
        print(cursor.warning)

        # Get details
        cursor.execute("""
                select line, position, text
                from user_errors
                where name = 'BADPROC' and type = 'PROCEDURE'
                order by line, position""")
        for info in cursor:
            print("Error at line {} position {}:\n{}".format(*info))
Результат будет следующим:

DPY-7000: creation succeeded with compilation errors
Error at line 3 position 23:
PLS-00103: Encountered the symbol "WRONG" when expecting one of the following:

   := . ( @ % ;
7.6 Использование атрибута %ROWTYPE
В PL/SQL атрибут %ROWTYPE позволяет объявить запись, представляющую полную или частичную строку таблицы или представления базы данных.

Для работы с %ROWTYPE в python-oracledb используйте Connection.gettype()для получения соответствующей информации о типе атрибута.

Получение значения %ROWTYPE из PL/SQL

Дана функция PL/SQL, которая возвращает строку таблицы LOCATIONS:

create or replace function TestFuncOUT return locations%rowtype as
  p locations%rowtype;
begin
   select * into p from locations where rownum < 2;
   return p;
end;
/
Вы можете использовать gettype(), чтобы получить тип возвращаемого значения функции PL/SQL, и указать его в качестве callfunc() типа возвращаемого значения. Например:

rt = connection.gettype("LOCATIONS%ROWTYPE")
r = cursor.callfunc("TESTFUNCOUT", rt)
Переменная rбудет содержать возвращаемое значение функции PL/SQL в виде объекта типа Object . Доступ к её содержимому можно получить с помощью методов, описанных в разделе «Извлечение объектов и коллекций базы данных Oracle» . Вспомогательная функция dump_object(), определённая там, служит удобным примером:

dump_object(r)
На выходе получим:

{
  LOCATION_ID: 1000
  STREET_ADDRESS: '1297 Via Cola di Rie'
  POSTAL_CODE: '00989'
  CITY: 'Roma'
  STATE_PROVINCE: None
  COUNTRY_ID: 'IT'
}
Создание значения %ROWTYPE в python-oracledb

Вы можете создать аналогичный объект непосредственно в python-oracledb, используя DbObjectType.newobject()и устанавливая любые необходимые поля. Например:

rt = connection.gettype("LOCATIONS%ROWTYPE")
r = rt.newobject()
r.CITY = 'Roma'
Передача значения %ROWTYPE в PL/SQL

Учитывая процедуру PL/SQL:

create or replace procedure TestProcIN(p in locations%rowtype, city out varchar2) as
begin
    city := p.city;
end;
вы можете вызвать callproc()передачу переменной rиз предыдущего примера callfunc()или newobject() в соответствующей позиции параметра, например:

c = cursor.var(oracledb.DB_TYPE_VARCHAR)
cursor.callproc("TESTPROCIN", [r, c])
print(c.getvalue())
Это печатает:

Roma
См. plsql_rowtype.py для работающего примера.

7.7 Использование DBMS_OUTPUT
Стандартный способ вывода данных из PL/SQL — использование пакета DBMS_OUTPUT . Обратите внимание, что код PL/SQL, использующий DBMS_OUTPUT, DBMS_OUTPUTвыполняется до завершения, прежде чем какие-либо выходные данные становятся доступны пользователю. Кроме того, другие соединения с базой данных не могут получить доступ к буферу.

Чтобы использовать DBMS_OUTPUT:

Вызовите процедуру PL/SQL DBMS_OUTPUT.ENABLE(), чтобы включить буферизацию вывода для соединения.

Выполнить некоторый код PL/SQL, который вызывает DBMS_OUTPUT.PUT_LINE()помещение текста в буфер.

Вызывайте DBMS_OUTPUT.GET_LINE()или DBMS_OUTPUT.GET_LINES()повторно извлекайте текст из буфера, пока вывод не прекратится.

Например:

# enable DBMS_OUTPUT
cursor.callproc("dbms_output.enable")

# execute some PL/SQL that calls DBMS_OUTPUT.PUT_LINE
cursor.execute("""
        begin
            dbms_output.put_line('This is the python-oracledb manual');
            dbms_output.put_line('Demonstrating how to use DBMS_OUTPUT');
        end;""")

# tune this size for your application
chunk_size = 100

# create variables to hold the output
lines_var = cursor.arrayvar(str, chunk_size)
num_lines_var = cursor.var(int)
num_lines_var.setvalue(0, chunk_size)

# fetch the text that was added by PL/SQL
while True:
    cursor.callproc("dbms_output.get_lines", (lines_var, num_lines_var))
    num_lines = num_lines_var.getvalue()
    lines = lines_var.getvalue()[:num_lines]
    for line in lines:
        print(line or "")
    if num_lines < chunk_size:
        break
Это даст следующий результат:

This is the python-oracledb manual
Demonstrating use of DBMS_OUTPUT
Альтернативой является вызов DBMS_OUTPUT.GET_LINE()по одному разу для каждой выходной строки, что может быть намного медленнее:

text_var = cursor.var(str)
status_var = cursor.var(int)
while True:
    cursor.callproc("dbms_output.get_line", (text_var, status_var))
    if status_var.getvalue() != 0:
        break
    print(text_var.getvalue())
7.8 Неявные результаты
Неявные результаты позволяют программе на Python использовать курсоры, возвращаемые блоком PL/SQL, без необходимости использования параметров OUT REF CURSORCursor.getimplicitresults() . Этот метод может быть использован для этой цели. Требуется Oracle Database 12.1 (или более поздняя версия). Для режима python-oracledb Thick дополнительно требуется Oracle Client 12.1 (или более поздняя версия).

Пример использования неявных результатов показан ниже:

cursor.execute("""
        declare
            cust_cur sys_refcursor;
            sales_cur sys_refcursor;
        begin
            open cust_cur for SELECT * FROM cust_table;
            dbms_sql.return_result(cust_cur);

            open sales_cur for SELECT * FROM sales_table;
            dbms_sql.return_result(sales_cur);
        end;""")

for implicit_cursor in cursor.getimplicitresults():
    for row in implicit_cursor:
        print(row)
Возвращаются данные из обоих наборов результатов:

(1, 'Tom')
(2, 'Julia')
(1000, 1, 'BOOKS')
(2000, 2, 'FURNITURE')
При использовании режима Thick в python-oracledb необходимо оставить родительский курсор открытым до тех пор, пока все неявные наборы результатов не будут извлечены или пока они не перестанут быть нужны приложению. Закрытие родительского курсора до извлечения всех неявных наборов результатов приведёт к закрытию курсоров неявных наборов результатов. При попытке извлечь данные из неявного набора результатов после закрытия его родительского курсора возникнет следующая ошибка:

DPI-1039: statement was already closed
Обратите внимание, что указанное выше требование не применимо к тонкому режиму python-oracledb. См. раздел «Неявные результаты в тонком и толстом режимах» .

7.9. Переопределение на основе издания (EBR)
Функция переопределения на основе редакций (Edition-Based Redefinition) в Oracle Database позволяет обновлять компоненты базы данных приложения во время его использования, минимизируя или полностью устраняя время простоя. Эта функция позволяет одновременно использовать несколько версий представлений, синонимов, объектов PL/SQL и профилей перевода SQL. Различные версии объектов базы данных связаны с одной редакцией.

Самый простой способ задать редакцию, используемую вашими приложениями, — передать editionпараметр oracledb.connect()или oracledb.create_pool():

connection = oracledb.connect(user="hr", password=userpwd,
                               dsn="dbhost.example.com/orclpdb",
                               edition="newsales")
Редакцию также можно установить, выполнив оператор SQL:

alter session set edition = <edition name>;
Вы также можете задать переменную окружения, ORA_EDITIONуказав имя вашей редакции.

Независимо от того, какой метод устанавливает редакцию, используемое значение можно увидеть, проверив атрибут Connection.edition. Если значение не задано, будет использоваться значение None. Это соответствует редакции базы данных по умолчанию ORA$BASE.

Рассмотрим пример, где одна версия функции PL/SQL Discountопределена в редакции базы данных по умолчанию ORA$BASE, а другая версия той же функции — в редакции, созданной пользователем DEMO. В редакторе SQL выполните:

connect <username>/<password>

-- create function using the database default edition
CREATE OR REPLACE FUNCTION Discount(price IN NUMBER) RETURN NUMBER IS
BEGIN
    return price * 0.9;
END;
/
Создаётся новое издание с именем «DEMO», и пользователю предоставляется разрешение на использование изданий. Использование FORCEнеобходимо, если у пользователя уже есть один или несколько объектов редактируемого типа, которые также имеют нередактируемые зависимые объекты.

connect system/<password>

CREATE EDITION demo;
ALTER USER <username> ENABLE EDITIONS FORCE;
GRANT USE ON EDITION demo to <username>;
Функционал Discountдемонстрационной версии следующий:

connect <username>/<password>

alter session set edition = demo;

-- Function for the demo edition
CREATE OR REPLACE FUNCTION Discount(price IN NUMBER) RETURN NUMBER IS
BEGIN
    return price * 0.5;
END;
/
Затем приложение Python может вызвать требуемую версию функции PL/SQL, как показано:

connection = oracledb.connect(user=user, password=password,
                               dsn="dbhost.example.com/orclpdb")
print("Edition is:", repr(connection.edition))

cursor = connection.cursor()
discounted_price = cursor.callfunc("Discount", int, [100])
print("Price after discount is:", discounted_price)

# Use the edition parameter for the connection
connection = oracledb.connect(user=user, password=password,
                               dsn="dbhost.example.com/orclpdb",
                               edition="demo")
print("Edition is:", repr(connection.edition))

cursor = connection.cursor()
discounted_price = cursor.callfunc("Discount", int, [100])
print("Price after discount is:", discounted_price)
Вывод вызова функции для версии по умолчанию и демонстрационной версии выглядит следующим образом:

Edition is: None
Price after discount is:  90
Edition is: 'DEMO'
Price after discount is:  50
