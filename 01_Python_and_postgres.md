# Работа с PostgreSQL в Python
В последнее время все чаще используется БД Postres и есть много задач, которые не всегда удобно автоматизировать используя встроенный язык СУБД. На помощь приходит Python.

## Подготовка
Для работы с БД Postgres нам необходимо установить пакет `psycopg2` 

```bash
pip install psycopg2
```

В процессе написания статей я использовал Python 3.12 и пакет `psycopg2==2.9.9`
```bash
> pip install psycopg2
Collecting psycopg2
  Using cached psycopg2-2.9.9-cp312-cp312-win_amd64.whl.metadata (4.5 kB)
Using cached psycopg2-2.9.9-cp312-cp312-win_amd64.whl (1.2 MB)
Installing collected packages: psycopg2
Successfully installed psycopg2-2.9.9
```

PS: Сгенерировать файл requirements.txt можно при помощи команды `pip freeze > requirements.txt`

## Начало работы

Для подключения к серверу БД нам необходимо знать:
 - hostname
 - имя базы данных
 - имя пользователя
 - пароль (если не настроен беспарольный вход)

Для примера все данные для подключения передаются в открытом виде, но в дальнейшем мы будем использовать файл конфигурации. Кроме подключения к БД нам необходимо создать "курсор" через который мы будем взаимодейстовать с БД.
После работы нам необходимо закрыть курсор и подключение.

```python
import psycopg2

conn = psycopg2.connect(dbname='pgdb', user='pguser', password='pgpwd', host='pghost')

cur = conn.cursor()
...
cur.close()
conn.close()

```

Для выполнения запросов к БД используется метод execute у созданного нами курсора. В качестве аргумента можно просто передать текст запроса в виде строки. 
Результат этого запроса можно получить тремя способами (в зависимости от того сколько предполагается получить данных):
- cursor.fetchone() — возвращает 1 строку
- cursor.fetchall() — возвращает список всех строк
- cursor.fetchmany(NumRows) — возвращает заданное количество строк

Ниже приведен пример получения текущей даты.

```python
import psycopg2

conn = psycopg2.connect(dbname='pgdb', user='pguser', password='pgpwd', host='pghost')

cur = conn.cursor()

cur.execute('select now();')
row, = cur.fetchone()
print(row)

cur.close()
conn.close()
```
