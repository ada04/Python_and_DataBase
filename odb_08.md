# Under constraction

# 8. Использование переменных привязки

Операторы SQL и PL/SQL, передающие данные в базу данных Oracle и обратно, должны использовать плейсхолдеры в операторах SQL и PL/SQL, которые отмечают, где данные предоставляются или возвращаются. Плейсхолдер для переменной привязки — это идентификатор или число с префиксом в виде двоеточия. Например, :dept_idи :dept_name— это два плейсхолдера для переменной привязки в этом операторе SQL:

sql = """insert into departments (department_id, department_name)
         values (:dept_id, :dept_name)"""
cursor.execute(sql, [280, "Facility"])
В процессе выполнения предоставленные значения переменных связывания 280заменяются "Facility"базой данных на заполнители. Это называется связыванием.

Использование переменных связывания важно для масштабируемости и безопасности. Они помогают избежать проблем безопасности, связанных с SQL-инъекциями, поскольку данные при разборе никогда не рассматриваются как часть исполняемого оператора.

Переменные связывания снижают затраты на анализ и выполнение, когда операторы выполняются более одного раза с разными значениями данных. Если переменные связывания не используются, Oracle приходится повторно анализировать и кэшировать несколько операторов. При использовании переменных связывания Oracle Database может повторно использовать план выполнения и контекст оператора.

> **! Важно**
>  Никогда не объединяйте и не интерполируйте пользовательские данные в операторы SQL:
  ```
  did = 280
  dnm = "Facility"

  # !! Never do this !!
  sql = f"""insert into departments (department_id, department_name)
            values ({did}, '{dnm}')"""
  cursor.execute(sql)
  ```
> Это представляет угрозу безопасности и может повлиять на производительность и масштабируемость.

Переменные связывания можно использовать для подстановки данных, но нельзя использовать для подстановки текста оператора. Например, нельзя использовать плейсхолдеры переменных связывания там, где требуется имя столбца или таблицы. Пласхолдеры переменных связывания также нельзя использовать в операторах DDL, таких как CREATE TABLE или ALTER.

# 8.1. Обязательство по имени или должности

Привязка может осуществляться «по имени» или «по должности».

8.1.1. Привязка по имени
Именованная привязка выполняется, когда переменные привязки в операторе Python используют имена плейсхолдеров в операторе SQL или PL/SQL. Например:

cursor.execute("""
        insert into departments (department_id, department_name)
        values (:dept_id, :dept_name)""",
        dept_id=280, dept_name="Facility")
В качестве альтернативы параметры можно передать как словарь, а не как ключевые параметры:

data = dict(dept_id=280, dept_name="Facility")
cursor.execute("""
        insert into departments (department_id, department_name)
        values (:dept_id, :dept_name)""", data)
В приведенных выше примерах имена ключевых параметров или ключи словаря должны совпадать с именами заполнителей переменных привязки.

Преимущества именованной привязки заключаются в том, что порядок значений привязки в execute()параметре не важен, имена могут быть осмысленными, а имена заполнителей могут повторяться, при этом значение в приложении по-прежнему предоставляется только один раз.

Пример повторного использования заполнителя переменной привязки:

cursor.execute("""
        update departments set department_id = :dept_id + 10
        where department_id = :dept_id""",
        dept_id=280)
8.1.2. Привязка по позиции
Позиционное связывание происходит, когда в вызов передаётся список или кортеж значений связывания execute(). Например:

cursor.execute("""
        insert into departments (department_id, department_name)
        values (:1, :2)""", [280, "Facility"])
Следующий пример (где изменяется порядок имён плейсхолдеров привязки) работает точно так же. Значение, используемое для замены плейсхолдера «:2», будет первым элементом списка, а «:1» будет заменён вторым элементом. Привязка по позиции работает слева направо и не учитывает имя переменной привязки:

cursor.execute("""
        insert into departments (department_id, department_name)
        values (:2, :1)""", [280, "Facility"])
В следующем примере также используется привязка по позиции, несмотря на то, что плейсхолдеры имеют буквенные имена. Сам процесс привязки использует позиции входных данных в списке для связывания данных с местоположением плейсхолдеров:

cursor.execute("""
        insert into departments (department_id, department_name)
        values (:dept_id, :dept_name)""", [280, "Facility"])
Кортежи Python также можно использовать для привязки по позиции:

cursor.execute("""
        insert into departments (department_id, department_name)
        values (:1, :2)""", (280, "Facility"))
Если в операторе SQL или PL/SQL используется только один заполнитель привязки, данные могут быть списком типа , [280]или кортежем из одного элемента типа (280,).

8.2. Дублирующиеся заполнители переменных привязки
Связывание по имени рекомендуется в тех случаях, когда имена заполнителей переменных связывания повторяются в операторах.

В тонком режиме python-oracledb при связывании по позиции для операторов SQL порядок значений привязки должен точно соответствовать порядку каждого заполнителя переменной привязки, а дублирующиеся имена должны иметь повторяющиеся значения:

cursor.execute("""
        select dname from dept1 where deptno = :1
        union all
        select dname from dept2 where deptno = :1 = """, [30, 30])
В некоторых случаях толстый режим python-oracledb может допускать недублирующиеся значения для SQL-выражений, но такое использование не является последовательным и не рекомендуется. В тонком режиме python-oracledb это приведёт к ошибке.

При связывании по позиции для вызовов PL/SQL в режимах python-oracledb Thin или Thick порядок значений привязки должен точно соответствовать порядку каждого уникального заполнителя, найденного в блоке PL/SQL, и значения не должны повторяться.

Привязка по имени не имеет таких проблем.

8.3 Направление привязки
Вызывающий объект может предоставить данные в базу данных (IN), база данных может вернуть данные вызывающему объекту (OUT), или вызывающий объект может предоставить исходные данные в базу данных, а база данных может вернуть изменённые данные вызывающему объекту (IN/OUT). Это называется направлением связывания.

Все представленные выше примеры предоставляют данные в базу данных и, следовательно, классифицируются как переменные связывания IN. Чтобы база данных вернула данные вызывающей стороне, необходимо создать переменную. Это делается путем вызова метода Cursor.var(), который, помимо прочего, определяет тип данных, которые будут найдены в этой переменной связывания, и её максимальный размер.

Вот пример использования OUT-связываний. Он вычисляет сумму целых чисел 8 и 7 и сохраняет результат в переменной OUT-связывания типа integer:

out_val = cursor.var(int)
cursor.execute("""
        begin
            :out_val := :in_bind_var1 + :in_bind_var2;
        end;""",
        out_val=out_val, in_bind_var1=8, in_bind_var2=7)
print(out_val.getvalue())        # will print 15
Если вместо того, чтобы просто получить данные обратно, вы хотите предоставить базе данных начальное значение, вы можете установить начальное значение переменной. Этот пример аналогичен предыдущему, но сначала задаёт начальное значение:

in_out_var = cursor.var(int)
in_out_var.setvalue(0, 25)
cursor.execute("""
        begin
            :in_out_bind_var := :in_out_bind_var + :in_bind_var1 +
                    :in_bind_var2;
        end;""",
        in_out_bind_var=in_out_var, in_bind_var1=8, in_bind_var2=7)
print(in_out_var.getvalue())        # will print 40
При привязке данных к параметрам процедур PL/SQL, объявленным как параметры OUT, следует учитывать, что любое значение, заданное в переменной привязки, будет проигнорировано. Кроме того, любые параметры, объявленные как IN/OUT, для которых не задано значение, будут изначально иметь значение null.

8.4 Привязка нулевых значений
Чтобы вставить значение NULL в символьный столбец, можно связать синглтон Python None. Например, с таблицей:

create table tab (id number, val varchar2(50));
Вы можете использовать:

cursor.execute("insert into tab (id, val) values (:i, :v)", i=280, v=None)
Python-oracledb предполагает, что значение будет строкой (эквивалентно столбцу VARCHAR2). Если вам нужно использовать другой тип данных Oracle Database, вам потребуется вызвать Cursor.setinputsizes()или создать переменную привязки с правильным типом, вызвав Cursor.var().

Например, если таблица была создана с использованием столбца объекта Oracle Spatial SDO_GEOMETRY :

create table tab (id number, val sdo_geometry);
Тогда предыдущий код завершится ошибкой:

ORA-00932: expression is of data type CHAR, which is incompatible with expected data type MDSYS.SDO_GEOMETRY
Чтобы вставить значение NULL в новую таблицу, используйте:

type_obj = connection.gettype("SDO_GEOMETRY")
var = cursor.var(type_obj)
cursor.execute("insert into tab (id, val) values (:i, :v)", i=280, v=var)
В качестве альтернативы используйте:

type_obj = connection.gettype("SDO_GEOMETRY")
cursor.setinputsizes(i=None, v=type_obj)
cursor.execute("insert into tab (id, val) values (:i, :v)", i=280, v=None)
8.5 Привязка значений ROWID
Псевдостолбец ROWID уникально идентифицирует строку в таблице. В python-oracledb значения ROWID представлены в виде строк. В примере ниже показано извлечение строки и её последующее обновление путём привязки её rowid:

# fetch the row
cursor.execute("""
        select rowid, manager_id
        from departments
        where department_id = :dept_id""", dept_id=280)
rowid, manager_id = cursor.fetchone()

# update the row by binding ROWID
cursor.execute("""
        update departments set
            manager_id = :manager_id
        where rowid = :rid""", manager_id=205, rid=rowid)
8.6 Привязка значений UROWID
Универсальные идентификаторы строк (UROWID) используются для уникальной идентификации строк в индексно-организованных таблицах. В python-oracledb значения UROWID представлены в виде строк. В примере ниже показано извлечение строки из индексно-организованной таблицы universal_rowidsи её последующее обновление путём привязки её urowid:

CREATE TABLE universal_rowids (
    int_col number(9) not null,
    str_col varchar2(250) not null,
    date_col date not null,
    CONSTRAINT universal_rowids_pk PRIMARY KEY(int_col, str_col, date_col)
) ORGANIZATION INDEX
ridvar = cursor.var(oracledb.DB_TYPE_UROWID)

# fetch the row
cursor.execute("""
        begin
            select rowid into :rid from universal_rowids
            where int_col = 3;
        end;""", rid=ridvar)

# update the row by binding UROWID
cursor.execute("""
        update universal_rowids set
            str_col = :str_val
        where rowid = :rowid_val""",
        str_val="String #33", rowid_val=ridvar)
Обратите внимание, что этот тип oracledb.DB_TYPE_UROWIDподдерживается только в тонком режиме python-oracledb. В толстом режиме python-oracledb тип базы данных UROWID может быть связан с типом oracledb.DB_TYPE_ROWID. См. раздел Запрос метаданных в тонком и толстом режимах .

8.7. DML RETURNING Связанные переменные
При использовании предложения RETURNING с оператором DML, таким как UPDATE, INSERT или DELETE, значения возвращаются в приложение посредством переменных связывания OUT. Рассмотрим следующий пример:

# The RETURNING INTO bind variable is a string
dept_name = cursor.var(str)

cursor.execute("""
        update departments set
            location_id = :loc_id
        where department_id = :dept_id
        returning department_name into :dept_name""",
        loc_id=1700, dept_id=50, dept_name=dept_name)
print(dept_name.getvalue())     # will print ['Shipping']
В приведенном выше примере, поскольку предложение WHERE соответствует только одной строке, вывод содержит один элемент из списка. Если бы предложение WHERE соответствовало нескольким строкам, вывод содержал бы столько элементов, сколько строк было обновлено.

Одно и то же имя плейсхолдера переменной связывания не может использоваться одновременно до и после предложения RETURNING. Например, если :dept_nameпеременная связывания используется одновременно до и после предложения RETURNING:

# a variable cannot be used for both input and output in a DML returning
# statement
dept_name_var = cursor.var(str)
dept_name_var.setvalue(0, 'Input Department')
cursor.execute("""
        update departments set
            department_name = :dept_name || ' EXTRA TEXT'
        returning department_name into :dept_name""",
        dept_name=dept_name_var)
В приведенном выше примере переменная привязки не будет обновлена ​​ожидаемым образом, но при использовании режима «Толстый» python-oracledb ошибка не возникнет. В режиме «Тонкий» python-oracledb этот пример возвращает следующую ошибку:

DPY-2048: the bind variable placeholder ":dept_name" cannot be used
both before and after the RETURNING clause in a DML RETURNING statement
8.8. Переменные привязки LOB
Объекты CLOB, NCLOBS, BLOB и BFILE базы данных могут быть связаны с типами oracledb.DB_TYPE_CLOB, oracledb.DB_TYPE_NCLOB, oracledb.DB_TYPE_BLOBи соответственно. Также могут быть связаны oracledb.DB_TYPE_BFILE объекты LOB, извлеченные из базы данных или созданные с помощью .Connection.createlob()

LOB-объекты могут представлять собой постоянные LOB-объекты Oracle Database (хранящиеся в таблицах) или временные LOB-объекты (например, созданные Connection.createlob()или возвращаемые некоторыми операциями SQL и PL/SQL).

LOB-объекты можно использовать как переменные связывания IN, OUT или IN/OUT.

Примеры см. в разделе Использование данных CLOB, BLOB, NCLOB и BFILE .

8.9. Переменные связывания REF CURSOR
Python-oracledb предоставляет возможность привязывать и определять курсоры PL/SQL REF. В качестве примера рассмотрим следующую процедуру PL/SQL:

CREATE OR REPLACE PROCEDURE find_employees (
    p_query IN VARCHAR2,
    p_results OUT SYS_REFCURSOR
) AS
BEGIN
    OPEN p_results FOR
        SELECT employee_id, first_name, last_name
        FROM employees
        WHERE UPPER(first_name || ' ' || last_name || ' ' || email)
            LIKE '%' || UPPER(p_query) || '%';
END;
/
Вновь открытый курсор можно привязать к параметру REF CURSOR, как показано в следующем коде Python. После вызова процедуры PL/SQL с помощью Cursor.callproc()курсор можно извлечь, как и любой другой курсор, выполнивший SQL-запрос:

ref_cursor = connection.cursor()
cursor.callproc("find_employees", ['Smith', ref_cursor])
for row in ref_cursor:
    print(row)
В примере HR-схемы Oracle есть два сотрудника с фамилией «Смит», поэтому результат следующий:

(159, 'Lindsey', 'Smith')
(171, 'William', 'Smith')
Чтобы вернуть REF CURSOR из функции PL/SQL, используйте oracledb.DB_TYPE_CURSORдля возвращаемого типа Cursor.callfunc():

ref_cursor = cursor.callfunc('example_package.f_get_cursor',
                             oracledb.DB_TYPE_CURSOR)
for row in ref_cursor:
    print(row)
Информацию о настройке REF CURSORS см. в разделе Настройка python-oracledb .

См. также раздел Неявные результаты , который предоставляет альтернативный способ возврата результатов запроса из процедур PL/SQL.

8.10 Связывание коллекций PL/SQL
Коллекции PL/SQL, такие как ассоциативные массивы, можно связывать как переменные IN, OUT и IN/OUT. При связывании значений IN массив можно передавать напрямую, как показано в этом примере, который суммирует длины всех строк в предоставленном массиве. Сначала определение пакета PL/SQL:

create or replace package mypkg as

    type udt_StringList is table of varchar2(100) index by binary_integer;

    function DemoCollectionIn (
        a_Values            udt_StringList
    ) return number;

end;
/

create or replace package body mypkg as

    function DemoCollectionIn (
        a_Values            udt_StringList
    ) return number is
        t_ReturnValue       number := 0;
    begin
        for i in 1..a_Values.count loop
            t_ReturnValue := t_ReturnValue + length(a_Values(i));
        end loop;
        return t_ReturnValue;
    end;

end;
/
Далее код Python:

values = ["String One", "String Two", "String Three"]
return_val = cursor.callfunc("mypkg.DemoCollectionIn", int, [values])
print(return_val)        # will print 32
Для получения значений из базы данных необходимо создать переменную привязки с помощью Cursor.arrayvar(). Первый параметр этого метода — тип данных Python, поддерживаемый python-oracledb, или один из типов API базы данных Oracle . Второй параметр — максимальное количество элементов, которые может содержать массив, или массив, предоставляющий значение (и косвенно максимальную длину). Последний параметр необязателен и используется только для строк и байтов. Он определяет максимальную длину строк и байтов, которые могут быть сохранены в массиве. Если не указано иное, длина по умолчанию равна 4000 байт.

Рассмотрим следующий пакет PL/SQL:

create or replace package mypkg as

    type udt_StringList is table of varchar2(100) index by binary_integer;

    procedure DemoCollectionOut (
        a_NumElements       number,
        a_Values            out nocopy udt_StringList
    );

    procedure DemoCollectionInOut (
        a_Values            in out nocopy udt_StringList
    );

end;
/

create or replace package body mypkg as

    procedure DemoCollectionOut (
        a_NumElements       number,
        a_Values            out nocopy udt_StringList
    ) is
    begin
        for i in 1..a_NumElements loop
            a_Values(i) := 'Demo out element #' || to_char(i);
        end loop;
    end;

    procedure DemoCollectionInOut (
        a_Values            in out nocopy udt_StringList
    ) is
    begin
        for i in 1..a_Values.count loop
            a_Values(i) := 'Converted element #' || to_char(i) ||
                    ' originally had length ' || length(a_Values(i));
        end loop;
    end;

end;
/
Код Python для обработки коллекции OUT будет выглядеть следующим образом. Обратите внимание, что вызов , который создаёт место для массива строк. Каждая строка может содержать до 100 байтов, и допускается только 10 строк. Если блок PL/SQL превысит максимально допустимое количество строк, возникнет Cursor.arrayvar()ошибка .ORA-06513: PL/SQL: index for PL/SQL table out of range for host language array

out_array_var = cursor.arrayvar(str, 10, 100)
cursor.callproc("mypkg.DemoCollectionOut", [5, out_array_var])
for val in out_array_var.getvalue():
    print(val)
Это приведет к следующему результату:

Demo out element #1
Demo out element #2
Demo out element #3
Demo out element #4
Demo out element #5
Код Python для обработки коллекций IN/OUT аналогичен. Обратите внимание на другой вызов, Cursor.arrayvar()который создаёт место для массива строк, но использует массив для определения как максимальной длины массива, так и его начального значения.

in_values = ["String One", "String Two", "String Three", "String Four"]
in_out_array_var = cursor.arrayvar(str, in_values)
cursor.callproc("mypkg.DemoCollectionInOut", [in_out_array_var])
for val in in_out_array_var.getvalue():
    print(val)
Это даст следующий результат:

Converted element #1 originally had length 10
Converted element #2 originally had length 10
Converted element #3 originally had length 12
Converted element #4 originally had length 11
Если переменная массива должна иметь начальное значение, но также должна допускать больше элементов, чем содержит начальное значение, вместо этого можно использовать следующий код:

in_out_array_var = cursor.arrayvar(str, 10, 100)
in_out_array_var.setvalue(0, ["String One", "String Two"])
Все коллекции, связанные в предыдущих примерах, использовали элементы непрерывного массива. Если требуется ассоциативный массив с разреженными элементами, следует использовать другой подход. Рассмотрим следующий код PL/SQL:

create or replace package mypkg as

    type udt_StringList is table of varchar2(100) index by binary_integer;

    procedure DemoCollectionOut (
        a_Value                         out nocopy udt_StringList
    );

end;
/

create or replace package body mypkg as

    procedure DemoCollectionOut (
        a_Value                         out nocopy udt_StringList
    ) is
    begin
        a_Value(-1048576) := 'First element';
        a_Value(-576) := 'Second element';
        a_Value(284) := 'Third element';
        a_Value(8388608) := 'Fourth element';
    end;

end;
/
Обратите внимание, что индексы элементов коллекции разделены большими значениями. Применённый выше метод не сработает, за исключением . Код, необходимый для обработки этой коллекции, выглядит следующим образом:ORA-06513: PL/SQL: index for PL/SQL table out of range for host language array

collection_type = connection.gettype("MYPKG.UDT_STRINGLIST")
collection = collection_type.newobject()
cursor.callproc("mypkg.DemoCollectionOut", [collection])
print(collection.aslist())
Это дает следующий результат:

['First element', 'Second element', 'Third element', 'Fourth element']
Обратите внимание, что использование метода Object.aslist()which возвращает значения элементов коллекции в порядке индексов в виде простого списка Python. При таком подходе сами индексы теряются. Ассоциативный массив можно преобразовать в словарь Python с помощью метода Object.asdict(). Если бы это значение было выведено в предыдущем примере, вывод был бы следующим:

{-1048576: 'First element', -576: 'Second element', 284: 'Third element', 8388608: 'Fourth element'}
Если элементы необходимо обойти в порядке индекса, можно использовать методы Object.first()и Object.next(). Метод Object.getelement()можно использовать для получения элемента с определённым индексом. Это показано в следующем коде:

ix = collection.first()
while ix is not None:
    print(ix, "->", collection.getelement(ix))
    ix = collection.next(ix)
Это дает следующий результат:

-1048576 -> First element
-576 -> Second element
284 -> Third element
8388608 -> Fourth element
Аналогично элементы можно обходить в обратном порядке индекса, используя методы Object.last()и , Object.prev()как показано в следующем коде:

ix = collection.last()
while ix is not None:
    print(ix, "->", collection.getelement(ix))
    ix = collection.prev(ix)
Это дает следующий результат:

8388608 -> Fourth element
284 -> Third element
-576 -> Second element
-1048576 -> First element
8.11 Связывание записей PL/SQL
Объекты типа «запись» PL/SQL также могут быть связаны с переменными IN, OUT и IN/OUT. Например:

create or replace package mypkg as

    type udt_DemoRecord is record (
        NumberValue                     number,
        StringValue                     varchar2(30),
        DateValue                       date,
        BooleanValue                    boolean
    );

    procedure DemoRecordsInOut (
        a_Value                         in out nocopy udt_DemoRecord
    );

end;
/

create or replace package body mypkg as

    procedure DemoRecordsInOut (
        a_Value                         in out nocopy udt_DemoRecord
    ) is
    begin
        a_Value.NumberValue := a_Value.NumberValue * 2;
        a_Value.StringValue := a_Value.StringValue || ' (Modified)';
        a_Value.DateValue := a_Value.DateValue + 5;
        a_Value.BooleanValue := not a_Value.BooleanValue;
    end;

end;
/
Затем этот код Python можно использовать для вызова хранимой процедуры, которая обновит запись:

# create and populate a record
record_type = connection.gettype("MYPKG.UDT_DEMORECORD")
record = record_type.newobject()
record.NUMBERVALUE = 6
record.STRINGVALUE = "Test String"
record.DATEVALUE = datetime.datetime(2016, 5, 28)
record.BOOLEANVALUE = False

# show the original values
print("NUMBERVALUE ->", record.NUMBERVALUE)
print("STRINGVALUE ->", record.STRINGVALUE)
print("DATEVALUE ->", record.DATEVALUE)
print("BOOLEANVALUE ->", record.BOOLEANVALUE)
print()

# call the stored procedure which will modify the record
cursor.callproc("mypkg.DemoRecordsInOut", [record])

# show the modified values
print("NUMBERVALUE ->", record.NUMBERVALUE)
print("STRINGVALUE ->", record.STRINGVALUE)
print("DATEVALUE ->", record.DATEVALUE)
print("BOOLEANVALUE ->", record.BOOLEANVALUE)
Это даст следующий результат:

NUMBERVALUE -> 6
STRINGVALUE -> Test String
DATEVALUE -> 2016-05-28 00:00:00
BOOLEANVALUE -> False

NUMBERVALUE -> 12
STRINGVALUE -> Test String (Modified)
DATEVALUE -> 2016-06-02 00:00:00
BOOLEANVALUE -> True
Обратите внимание, что при манипулировании записями все атрибуты должны быть установлены программой Python, чтобы избежать ошибки клиента Oracle, которая может привести к непредвиденным значениям или сбою сегментации приложения Python.

8.12 Связывание пространственных типов данных
Объекты типов данных Oracle Spatial могут быть представлены объектами Python, а значения их атрибутов можно читать и обновлять. Кроме того, объекты можно привязывать и фиксировать в базе данных. Это аналогично примерам выше.

Пример выборки SDO_GEOMETRY можно найти в Oracle Database Objects and Collections .

8.13 Уменьшение количества версий SQL
При повторных вызовах Cursor.execute()или Cursor.executemany() связывании строковых данных разной длины использование Cursor.setinputsizes() может помочь сократить « количество версий » SQL-выражения в Oracle Database для оператора. Количество версий — это количество дочерних курсоров, используемых для одного и того же текста оператора. В базе данных будет родительский курсор, представляющий текст оператора, и несколько дочерних курсоров для различных вариантов выполнения оператора, например, при использовании переменных связывания разных типов или длины.

Например, если таблица создана следующим образом:

create table mytab (c1 varchar2(25), c2 varchar2(100), c3 number);
setinputsizes()Для уменьшения количества дочерних курсоров можно использовать :

sql = "insert into mytab (c1, c2) values (:1, :2)"

cursor.setinputsizes(25, 15)

s1 = "abc"
s2 = "def"
cursor.execute(sql, [s1, s2])

s1 = "aaaaaaaaaaaaaaaaaaaaaaaaa"
s2 = "z"
cursor.execute(sql, [s1, s2])
Вызов setinputsizes()указывает, что первое связанное значение будет строкой Python длиной не более 25 символов, а второе связанное значение — строкой длиной не более 15 символов. Если длина строки данных превышает заданные setinputsizes()значения, python-oracledb примет их, но это не даст никакого выигрыша в обработке.

Нередко SQL-операторы имеют несколько сотен версий. Иногда это ожидаемо и не является результатом какой-либо проблемы. Чтобы определить причину, найдите идентификатор SQL-оператора и выполните запрос к представлению базы данных Oracle V$SQL_SHARED_CURSOR .

SQL-идентификатор оператора можно найти в представлениях Oracle Database, например, V$SQLAREA, после выполнения оператора, или до его выполнения с помощью функции DBMS_SQL_TRANSLATOR.SQL_ID() . Убедитесь, что вы передаете точно такой же текст SQL, включая те же пробелы:

sql = "insert into mytab (c1, c2) values (:1, :2)"  # statement to examine

cursor.execute("select dbms_sql_translator.sql_id(:1) from dual", [sql])
(sqlid,) = cursor.fetchone();
print(sqlid)
Это может вывести такое значение:

6h6gj3ztw2wd8
Затем, чтобы найти версии SQL, выполните запрос для просмотра дочерних курсоров. Например:

cursor.execute("""select child_number, reason
                  from v$sql_shared_cursor
                  where sql_id = :1 order by 1""", [sqlid])
col_names = [c.name for c in cursor.description]
for row in cursor.fetchall():
    r = [dict(zip(col_names, row))]
    print(r)
В предыдущем коде, который использовал setinputsizes()и вставлял данные разной длины, вы могли увидеть:

[{'CHILD_NUMBER': 0, 'REASON': ' '}]
Однако если setinputsizes()бы он не использовался, вы бы увидели две строки, а REASON включал бы текст «Несоответствие привязки», например:

[{'CHILD_NUMBER': 0, 'REASON': '<ChildNode><ChildNumber>0</ChildNumber><ID>39</ID><reason>Bind mismatch(22)</reason><size>4x8</size><bind_position>0</bind_position><original_oacflg>1</original_oacflg><original_oacmxl>32</original_oacmxl><upgradeable_new_oacmxl>128</upgradeable_new_oacmxl></ChildNode> '}]
[{'CHILD_NUMBER': 1, 'REASON': '<ChildNode><ChildNumber>1</ChildNumber><ID>39</ID><reason>Bind mismatch(22)</reason><size>4x8</size><bind_position>0</bind_position><original_oacflg>1</original_oacflg><original_oacmxl>128</original_oacmxl><upgradeable_new_oacmxl>32</upgradeable_new_oacmxl></ChildNode> '}]
8.14 Изменение типов данных привязки с помощью обработчика типов ввода
Обработчики типов входных данных позволяют приложениям изменять способ привязки данных к операторам или даже разрешать прямую привязку новых типов.

Обработчик типа ввода включается путем установки атрибута Cursor.inputtypehandlerили Connection.inputtypehandler.

Вставка значений NaN как NULL в столбцы NUMBER

Чтобы вставить значения NaN как NULL в столбец NUMBER, используйте обработчик типа ввода с преобразователем:

def input_type_handler(cursor, value, arraysize):
  if isinstance(value, float):
      return cursor.var(oracledb.DB_TYPE_NUMBER, arraysize=arraysize,
                        inconverter=lambda x: None if math.isnan(x) else x)

connection.inputtypehandler = input_type_handler
Обратите внимание, что это не требуется для столбцов BINARY_FLOAT или BINARY_DOUBLE.

Привязка объектов Python

Обработчики типов входных данных можно комбинировать с преобразователями переменных для бесшовной привязки объектов Python:

# A standard Python object
class Building:

    def __init__(self, build_id, description, num_floors, date_built):
        self.building_id = build_id
        self.description = description
        self.num_floors = num_floors
        self.date_built = date_built

building = Building(1, "Skyscraper 1", 5, datetime.date(2001, 5, 24))

# Get Python representation of the Oracle user defined type UDT_BUILDING
obj_type = con.gettype("UDT_BUILDING")

# convert a Python Building object to the Oracle user defined type
# UDT_BUILDING
def building_in_converter(value):
    obj = obj_type.newobject()
    obj.BUILDINGID = value.building_id
    obj.DESCRIPTION = value.description
    obj.NUMFLOORS = value.num_floors
    obj.DATEBUILT = value.date_built
    return obj

def input_type_handler(cursor, value, num_elements):
    if isinstance(value, Building):
        return cursor.var(obj_type, arraysize=num_elements,
                          inconverter=building_in_converter)


# With the input type handler, the bound Python object is converted
# to the required Oracle object before being inserted
cur.inputtypehandler = input_type_handler
cur.execute("insert into myTable values (:1, :2)", (1, building))
8.15 Привязка нескольких значений к предложению SQL WHERE IN
Чтобы использовать список SQL IN с несколькими значениями, используйте один плейсхолдер для переменной привязки на каждое значение. Список или словарь Python нельзя напрямую привязать к одной переменной привязки. Например, чтобы использовать два значения в предложении IN, код должен выглядеть так:

items = ["Smith", "Taylor"]
cursor.execute("""
    select employee_id, first_name, last_name
    from employees
    where last_name in (:1, :2)""",
    items)
for row in cursor:
    print(row)
Это дает результат:

(159, 'Lindsey', 'Smith')
(171, 'William', 'Smith')
(176, 'Jonathon', 'Taylor')
(180, 'Winston', 'Taylor')
Если запрос выполняется несколько раз с разным количеством значений, в SQL-оператор следует включить плейсхолдер для связывания переменных для каждого из максимально возможного количества значений. Если оператор выполняется с меньшим количеством значений данных, следует выполнить связывание Noneдля отсутствующих значений. Например, если запрос используется для пяти значений, но в конкретном выполнении используются только два, код может быть следующим:

items = ["Smith", "Taylor", None, None, None]
cursor.execute("""
    select employee_id, first_name, last_name
    from employees
    where last_name in (:1, :2, :3, :4, :5)""",
    items)
for row in cursor:
    print(row)
Это даст тот же результат, что и исходный пример.

Повторное использование одного и того же SQL-оператора, подобное этому, для переменного количества значений, вместо создания уникального оператора для каждого набора значений, позволяет максимально эффективно повторно использовать ресурсы Oracle Database. Кроме того, если оператор с большим количеством плейсхолдеров переменных связывания выполняется много раз с разной длиной строк при каждом выполнении, рассмотрите возможность использования , чтобы уменьшить « количество версийCursor.setinputsizes() » SQL-оператора Oracle Database для этого оператора. Например, если столбцы имеют тип VARCHAR2(25), добавьте перед вызовом следующее:Cursor.execute()

cursor.setinputsizes(25,25,25,25,25)
Если в операторе требуются другие переменные связывания, соответствующим образом скорректируйте номера заполнителей переменных связывания:

binds = [120]                                   # employee id
binds += ["Smith", "Taylor", None, None, None]  # IN list
cursor.execute("""
    select employee_id, first_name, last_name
    from employees
    where employee_id > :1
    and last_name in (:2, :3, :4, :5, :6)""",
    binds)
for row in cursor:
    print(row)
Если оператор, содержащий WHERE IN, не будет выполняться повторно или количество значений будет известно только во время выполнения, то оператор SQL можно построить динамически:

bind_values = ["Gates", "Marvin", "Fay"]
bind_names = ",".join(":" + str(i + 1) for i in range(len(bind_values)))
sql = f"select first_name, last_name from employees where last_name in ({bind_names})"
cursor.execute(sql, bind_values)
for row in cursor:
    print(row)
8.15.1. Связывание большого количества элементов в списке IN
Количество элементов в списке IN ограничено 65 535 в Oracle Database версии 23 и 1000 в более ранних версиях. При превышении лимита база данных вернет ошибку вида .ORA-01795: maximum number of expressions in a list is 65535

Чтобы использовать больше значений в списке предложений IN, вы можете добавить предложения OR, например:

sql = """select . . .
         where key in (:0,:1,:2,:3,:4,:5,:6,:7,:8,:9,:10,:11,...)
            or key in (:50,:51,:52,:53,:54,:55,:56,:57,:58,:59,...)
            or key in (:100,:101,:102,:103,:104,:105,:106,:107,:108,:109,...)"""
Более общее решение для большего количества значений — создание SQL-оператора следующего вида:

SELECT ... WHERE col IN ( <something that returns a list of values> )
Лучший способ реализовать «<нечто, возвращающее список значений>» зависит от изначального представления данных и количества элементов. Например, можно рассмотреть возможность использования глобальной временной таблицы.

Одним из методов для больших списков IN является использование коллекции Oracle Database с TABLE()предложением . Например, если был создан следующий тип:

SQL> CREATE OR REPLACE TYPE name_array AS TABLE OF VARCHAR2(25);
  2  /
то приложение может сделать:

type_obj = connection.gettype("NAME_ARRAY")
obj = type_obj.newobject()
obj.extend(["Smith", "Taylor"])
cursor.execute("""select employee_id, first_name, last_name
                  from employees
                  where last_name in (select * from table(:1))""",
               [obj])
for row in cursor:
    print(row)
При использовании этого метода важно пересмотреть план оптимизатора базы данных, чтобы убедиться в его эффективности.

Поскольку это TABLE()решение использует тип объекта, производительность снижается из-за дополнительных циклов, необходимых для получения информации о типе. Если у вас не слишком много привязок, вы можете предпочесть одно из предыдущих решений. Для эффективности Connection.gettype()по возможности сохраняйте возвращаемое значение для повторного использования вместо того, чтобы вызывать его повторно.

Некоторые пользователи используют типы SYS.ODCINUMBERLIST, SYS.ODCIVARCHAR2LIST или SYS.ODCIDATELIST вместо создания своего собственного типа, но это следует использовать с пониманием того, что база данных может удалить их в будущей версии, и что их размер составляет 32 КБ - 1.

8.16 Связывание имен столбцов и таблиц
Имена таблиц нельзя связывать в SQL-запросах. Вы можете объединить текст для создания SQL-выражения, но обязательно используйте список разрешенных операторов или другие средства проверки данных, чтобы избежать проблем безопасности, связанных с SQL-инъекциями:

table_allow_list = ['employees', 'departments']
table_name = get_table_name() #  get the table name from user input
if table_name.lower() not in table_allow_list:
    raise Exception('Invalid table name')
sql = f'select * from {table_name}'
Связывание имён столбцов можно выполнить либо аналогичным способом, либо с помощью оператора CASE. В примере ниже показано связывание имени столбца в предложении ORDER BY:

sql = """
    select *
    from departments
    order by
        case :bindvar
            when 'DEPARTMENT_ID' then
                department_id
            else
                manager_id
        end"""

col_name = get_column_name() # Obtain a column name from the user
cursor.execute(sql, [col_name])
В зависимости от имени, предоставленного пользователем, результаты запроса будут упорядочены либо по столбцу DEPARTMENT_ID, либо по столбцу MANAGER_ID.
