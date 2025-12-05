# under construction

# 12. Использование данных CLOB, BLOB, NCLOB и BFILE

Oracle Database использует LOB-объекты для хранения больших объёмов данных, таких как текст, изображения, видео и другие мультимедийные форматы. Максимальный размер LOB-объекта ограничен размером табличного пространства, в котором он хранится.

Существует четыре типа LOB :

- CLOB — большой символьный объект, используемый для хранения строк в формате набора символов базы данных. python-oracledb использует тип oracledb.DB_TYPE_CLOB.

- BLOB — большой двоичный объект, используемый для хранения двоичных данных. python-oracledb использует тип oracledb.DB_TYPE_BLOB.

- NCLOB — большой объект национальных символов, используемый для хранения строк в формате национального набора символов. python-oracledb использует тип oracledb.DB_TYPE_NCLOB.

- BFILE — внешний двоичный файл, используемый для ссылки на файл, хранящийся в операционной системе хоста за пределами базы данных. python-oracledb использует тип oracledb.DB_TYPE_BFILE.

LOB-объекты могут быть постоянными или временными. Их можно добавлять в базу данных Oracle и извлекать из неё по частям по мере необходимости.

В python-oracledb можно обрабатывать большие объекты размером до 1 ГБ как строки или байты. Это упрощает работу с большими объектами и даёт значительный выигрыш в производительности по сравнению с потоковой обработкой. Однако для этого требуется, чтобы все данные больших объектов находились в памяти Python, что может быть невозможно.

Примеры LOB-объектов смотрите на [GitHub](https://github.com/oracle/python-oracledb/tree/main/samples).

## 12.1 Простая вставка больших объектов

Рассмотрим таблицу со столбцами CLOB и BLOB:

```
CREATE TABLE lob_tbl (
    id NUMBER,
    c CLOB,
    b BLOB
);
```

С помощью python-oracledb данные LOB можно вставлять в таблицу с использованием bind переменных:

```
with open('example.txt', 'r') as f:
    text_data = f.read()

with open('image.png', 'rb') as f:
    img_data = f.read()

cursor.execute("""
        insert into lob_tbl (id, c, b)
        values (:lobid, :clobdata, :blobdata)""",
        lobid=10, clobdata=text_data, blobdata=img_data)
```

> Обратите внимание, что при таком подходе размер LOB-данных ограничен 1 ГБ.

## 12.2 Извлечение больших объектов в виде строк и байтов

Объекты CLOB и BLOB размером менее 1 ГБ можно запрашивать из базы данных напрямую как строки и байты. Это может быть гораздо быстрее, чем потоковая передача LOB-объектов. Поддержка включается установкой `oracledb.defaults.fetch_lobs`, или установкой параметра `fetch_lobs` при выполнении оператора:

```
import oracledb

. . .

id_val = 1
text_data = "The quick brown fox jumps over the lazy dog"
binary_data = b"Some binary data"
cursor.execute("insert into lob_tbl (id, c, b) values (:1, :2, :3)",
               [id_val, text_data, binary_data])

cursor.execute("select c, b from lob_tbl where id = :1", [id_val], fetch_lobs=False)
clob_data, blob_data = cursor.fetchone()
print("CLOB length:", len(clob_data))
print("CLOB data:", clob_data)
print("BLOB length:", len(blob_data))
print("BLOB data:", blob_data)
```

Результат:

```
CLOB length: 43
CLOB data: The quick brown fox jumps over the lazy dog
BLOB length: 16
BLOB data: b'Some binary data'
```

Более старая альтернатива использованию `fetch_lobs` — использование обработчика типов:

```
def output_type_handler(cursor, metadata):
    if metadata.type_code is oracledb.DB_TYPE_CLOB:
        return cursor.var(oracledb.DB_TYPE_LONG, arraysize=cursor.arraysize)
    if metadata.type_code is oracledb.DB_TYPE_BLOB:
        return cursor.var(oracledb.DB_TYPE_LONG_RAW, arraysize=cursor.arraysize)
    if metadata.type_code is oracledb.DB_TYPE_NCLOB:
        return cursor.var(oracledb.DB_TYPE_LONG_NVARCHAR, arraysize=cursor.arraysize)

connection.outputtypehandler = output_type_handler
```

## 12.3. Потоковые LOB-объекты (чтение)

Если не установить `oracledb.defaults.fetch_lobs` значение `False` или эквивалентный параметр, или не использовать обработчик выходных типов, значения CLOB и BLOB будут извлечены как объекты LOB. Размер объекта LOB можно получить, вызвав `LOB.size()`, а данные можно прочитать, вызвав `LOB.read()`:

```
id_val = 1
text_data = "The quick brown fox jumps over the lazy dog"
binary_data = b"Some binary data"
cursor.execute("insert into lob_tbl (id, c, b) values (:1, :2, :3)",
               [id_val, text_data, binary_data])

cursor.execute("select b, c from lob_tbl where id = :1", [id_val])
b, c = cursor.fetchone()
print("CLOB length:", c.size())
print("CLOB data:", c.read())
print("BLOB length:", b.size())
print("BLOB data:", b.read())
```

Этот подход даёт те же результаты, что и предыдущий пример, но работает медленнее, поскольку требует большего количества обращений к Oracle Database и ведёт к более высоким накладным расходам. Однако он необходим, если данные LOB невозможно получить с сервера одним блоком.

Для потоковой передачи столбца BLOB метод `LOB.read()` можно вызывать многократно, пока все данные не будут прочитаны, как показано ниже:

```
cursor.execute("select b from lob_tbl where id = :1", [10])
blob, = cursor.fetchone()
offset = 1
num_bytes_in_chunk = 65536
with open("image.png", "wb") as f:
    while True:
        data = blob.read(offset, num_bytes_in_chunk)
        if data:
            f.write(data)
        if len(data) < num_bytes_in_chunk:
            break
        offset += len(data)
```

## 12.4 Потоковые LOB-объекты (запись)

Если вставляется или обновляется строка, содержащая LOB, и объем данных, которые должны быть вставлены или обновлены, не может поместиться в один блок данных, данные можно передать потоком с помощью метода `LOB.write()`, как показано в следующем коде:

```
id_val = 9
lob_var = cursor.var(oracledb.DB_TYPE_BLOB)
cursor.execute("""
        insert into lob_tbl (id, b)
        values (:1, empty_blob())
        returning b into :2""", [id_val, lob_var])
blob, = lobVar.getvalue()
offset = 1
num_bytes_in_chunk = 65536
with open("image.png", "rb") as f:
    while True:
        data = f.read(num_bytes_in_chunk)
        if data:
            blob.write(data, offset)
        if len(data) < num_bytes_in_chunk:
            break
        offset += len(data)
connection.commit()
```

## 12.5 Временные LOB

Во всех показанных до сих пор примерах использовались постоянные LOB-объекты. Это LOB-объекты, хранящиеся в базе данных. Oracle также поддерживает временные LOB-объекты, которые не хранятся в базе данных, но могут использоваться для передачи больших объемов данных. Эти LOB-объекты занимают место во временном табличном пространстве до тех пор, пока все ссылающиеся на них переменные не выйдут из области действия или соединение, в котором они были созданы, не будет явно закрыто.

При вызове процедур PL/SQL с данными, длина которых превышает 32 767 байт, python-oracledb автоматически создаёт временный LOB объект и передаёт его значение процедуре. Однако, если объём данных, передаваемых процедуре, превышает объём, помещающийся в один блок данных, можно использовать метод `Connection.createlob()` для создания временного LOB объекта. Этот LOB объект затем можно читать и записывать так же, как в примерах выше для постоянных LOB объектов.

## 12.6 Использование BFILE

BFILE — это объекты, хранящиеся в каталоге файловой системы сервера Oracle Database, а не в базе данных. Столбец базы данных типа BFILE хранит ссылку на этот внешний двоичный файл. Каждый столбец BFILE может ссылаться на один внешний файл. BFILE — это тип данных, доступный только для чтения, поэтому вы не можете изменять файл из приложения.

Перед использованием типа данных BFILE необходимо:

- Создать объект DIRECTORY, который является псевдонимом для полного пути к каталогу, содержащему данные BFILE в файловой системе сервера базы данных. Например, вы можете создать объект DIRECTORY следующим образом:

```
create or replace directory my_bfile_dir as '/demo/bfiles'
```

  В приведенном выше примере «my_bfile_dir» — это псевдоним каталога. «/demo/bfiles» — это физический каталог в файловой системе сервера базы данных, содержащий файлы. Это строка, содержащая полный путь к каталогу и соответствующая правилам операционной системы.

  Чтобы разрешить непривилегированным пользователям доступ к этому каталогу, вы можете предоставить доступ с помощью:

```
grant read on directory my_bfile_dir to hr;
```

  Убедитесь, что процессы Oracle Server имеют доступ на чтение к каталогу.

- Сохраните физический двоичный файл в каталоге файловой системы сервера базы данных. Например, двоичный файл «my_bfile.txt» хранится в каталоге «/demo/bfiles».

Предположим, что файл «/demo/bfiles/my_bfile.txt» существует на сервере и содержит текст «Это мои данные BFILE». Вы можете получить доступ к файлу «my_bfile.txt», как описано ниже.

Следующая таблица будет использоваться в последующих примерах.

create table bfile_tbl(
    id number,
    bfile_data bfile
);

#### Вставка BFILE

Для связи файла и столбца BFILE необходимо использовать функцию `BFILENAME` в операторе INSERT. Функция `BFILENAME` принимает два аргумента: псевдоним каталога и имя файла. Чтобы вставить ссылку BFILE, например:

```
cursor.execute("""
    insert into bfile_tbl (id, bfile_data) values
    (:id, bfilename(:bfiledir, :bfilename))""",
    id=102, bfiledir="my_bfile_dir", bfilename="my_bfile.txt")

connection.commit()
```

Это вставляет ссылку на файл «my_bfile.txt», расположенный в каталоге, на который ссылается псевдоним «my_bfile_dir», в таблицу bfile_tbl.

#### Извлечение BFILE

Чтобы запросить таблицу bfile_tbl и получить локатор LOB-объектов BFILE, можно использовать функцию `BFILENAME`, как показано ниже:

```
cursor.execute("select bfilename(:bfiledir, :bfilename) from bfile_tbl where id = :id",
    id=102, bfiledir="my_bfile_dir", bfilename="my_bfile.txt")
bfile, = cursor.fetchone()
print(bfile.read())
```

Результат:

```
This is my BFILE data
```

Для полученного LOB-объекта можно использовать `LOB.fileexists()` для проверки того, существует ли файл, на который ссылается LOB-объект типа BFILE.

Вы можете получить псевдоним каталога и имя файла этого извлеченного LOB-объекта, используя `LOB.getfilename()`. Вы также можете задать псевдоним каталога и имя файла для этого извлеченного LOB-объекта, используя `LOB.setfilename()`.
