# в разработке

## 2. Установка python-oracledb
Драйвер python-oracledb позволяет приложениям Python подключаться к базе данных Oracle без необходимости установки дополнительных библиотек.

![архитектура драйвера python-oracledb](https://python-oracledb.readthedocs.io/en/latest/_images/python-oracledb-thin-arch.png)
Рис. 2.1 Архитектура драйвера python-oracledb

По умолчанию python-oracledb работает в режиме «тонкого» клиента, который подключается напрямую к Oracle Database. В этом режиме клиентские библиотеки Oracle не требуются. Однако при их использовании доступны некоторые [дополнительные функции](odb_29.md). При использовании подключения в режиме "толстого клиента" для python-oracledb требуются клиентские библиотеки Oracle. См. раздел «Включение толстого режима python-oracledb». Оба режима обладают полной функциональностью, поддерживающей спецификацию Python Database API v2.0.

### 2.1. Быстрая установка python-oracledb
Python-oracledb обычно устанавливается из репозитория пакетов Python [PyPI](https://pypi.org/project/oracledb/) с помощью [pip](https://pip.pypa.io/en/latest/installation/) или [uv](https://pypi.org/project/uv/).

1. Установите Python 3, если он еще не доступен.

Используйте любую версию от Python 3.9 до 3.14.

Предыдущие версии python-oracledb поддерживали старые версии Python.

2. Установите python-oracledb, например:

```bash
python -m pip install oracledb --upgrade --user
```

Обратите внимание, что имя модуля — `oracledb`.

Если вы используете прокси-сервер, воспользуйтесь опцией `--proxy`. Например:

```bash
python -m pip install oracledb --upgrade --user --proxy=http://proxy.example.com:80
```

По умолчанию python-oracledb подключается напрямую к Oracle Database. Это позволяет использовать его сразу, без необходимости установки дополнительных клиентских библиотек Oracle.

3. Создайте файл, `test.py` например:

```python
import oracledb
import getpass

user_name = "scott"
conn_string = "localhost/orclpdb"
# cs = "localhost/freepdb1"   # for Oracle AI Database Free users
# cs = "localhost/orclpdb1"   # some databases may have this service
pw = getpass.getpass(f"Enter password for {user_name}@{connect_string}: ")

with oracledb.connect(user=user_name, password=pw, dsn=connect_string) as connection:
    with connection.cursor() as cursor:
        sql = "select sysdate from dual"
        for r in cursor.execute(sql):
            print(r)
```

4. Отредактируйте `test.py` и задайте переменные `user_name` и `connect_string` для своего имени пользователя базы данных и строки подключения к базе данных соответственно.

Для простого подключения к базе данных требуются имя пользователя и пароль Oracle Database, а также строка подключения к базе данных. Для python-oracledb общепринятым форматом строки подключения является `hostname:port/servicename`, включающая имя хоста, на котором запущена база данных, имя службы Oracle Database экземпляра базы данных и порт, используемый базой данных. Если используется порт по умолчанию 1521, этот компонент строки подключения часто опускается.

База данных может быть локальной, удалённой или облачной.

5. Запустите программу:

```
python test.py
```

При появлении запроса введите пароль базы данных, и будет отображена запрашиваемая дата, например:

```
Enter password for cj@localhost/orclpdb: xxxxxxxxxx
(datetime.datetime(2024, 4, 30, 8, 24, 4),)
```

Если у вас возникли проблемы с установкой, обратитесь к подробным инструкциям ниже или ознакомьтесь с разделом [Устранение ошибок](odb_28.md) .

Дополнительную информацию о python-oracledb можно найти в [документации](https://python-oracledb.readthedocs.io/en/latest/index.html) и [примерах](https://github.com/oracle/python-oracledb/tree/main/samples).

## 2.2 Поддерживаемые версии базы данных Oracle

При использовании `python-oracledb` в режиме по умолчанию (тонкий клиент) он подключается напрямую к базе данных Oracle и не требует клиентских библиотек Oracle. В этом режиме можно подключаться к базе данных Oracle версии 12.1 и выше.

Для подключения к более старым версиям Oracle Database необходимо установить клиентские библиотеки Oracle и включить режим Thick в python-oracledb .

В режиме Thick сетевое взаимодействие обеспечивают клиентские библиотеки Oracle Database. Информация о текущих или ранее сертифицированных конфигурациях представлена ​​в документе Oracle Support с идентификатором 207303.1 . Вкратце:

- Oracle Client 23 может подключаться к Oracle Database 19 или более поздней версии.

- Oracle Client 21 может подключаться к Oracle Database 12.1 или более поздней версии.

- Oracle Client 19, 18 и 12.2 могут подключаться к Oracle Database 11.2 или более поздней версии.

- Oracle Client 12.1 может подключаться к Oracle Database 10.2 или более поздней версии.

- Oracle Client 11.2 может подключаться к Oracle Database 9.2 или более поздней версии.

Любая попытка использовать функции Oracle Database, не поддерживаемые определённым режимом или комбинацией клиентской библиотеки и базы данных, приведёт к ошибкам выполнения. Атрибут python-oracledb `Connection.thin` позволяет определить режим соединения. В режиме Thick функция `oracledb.clientversion()` позволяет определить используемую версию Oracle Client. Атрибут `Connection.version` позволяет определить, к какой версии Oracle Database обращается соединение. Эти атрибуты затем можно использовать для соответствующей настройки поведения приложения.

Примечание. Oracle ведет список версий базы данных, для которых можно получить разрешение проблем и исправление ошибок. См. [График выпуска текущих версий базы данных](https://support.oracle.com/epmos/faces/DocumentDisplay?id=742060.1).

## 2.3 Требования к установке
Для использования python-oracledb вам необходимо:

- Python 3.9, 3.10, 3.11, 3.12, 3.13 или 3.14

 - Пакет криптографии Python. Этот пакет автоматически устанавливается как зависимость `python-oracledb`. Настоятельно рекомендуется поддерживать пакет криптографии в актуальном состоянии при выходе новых версий. Если пакет криптографии недоступен, вы всё равно можете установить python-oracledb, но использовать его можно только в режиме Thick. См. раздел [Установка python-oracledb без пакета криптографии](!!!).

При желании можно установить клиентские библиотеки Oracle для реализации дополнительных расширенных функций. Библиотеки можно получить из бесплатных пакетов Oracle Instant Client Basic или Basic Light, из полной установки Oracle Client (например, установленной с помощью графического установщика Oracle) или из библиотек, входящих в состав Oracle Database, если Python установлен на том же компьютере, что и база данных. Клиентские библиотеки Oracle версий 23, 21, 19, 18, 12 и 11.2 можно использовать (при наличии) в Linux, Windows и macOS. Стандартная совместимость клиент-серверных версий Oracle позволяет подключаться как к старым, так и к новым базам данных.

База данных Oracle — локальная или удаленная, локальная или в облаке.

2.4. Установка python-oracledb на Linux
В этом разделе рассматриваются общие методы установки в Linux.

2.4.1. Установка python-oracledb
Обычный способ установки python-oracledb в Linux — использование пакета Python pip для установки из репозитория пакетов Python PyPI :

python -m pip install oracledb --upgrade
Это загрузит и установит предварительно скомпилированный двоичный файл из PyPI , если он доступен для вашей архитектуры. В противном случае исходный код будет загружен, скомпилирован, а полученный двоичный файл установлен. Для компиляции python-oracledb требуется Python.hзаголовочный файл. Если вы используете pythonпакет по умолчанию, этот файл находится в python-develпакете или его эквиваленте.

python3На некоторых платформах вместо . может вызываться двоичный файл Python python.

Параметр установки --userполезен, когда у вас нет прав на запись в системные каталоги:

python -m pip install oracledb --upgrade --user
Если вы используете прокси-сервер, воспользуйтесь этой --proxyопцией. Например:

python -m pip install oracledb --upgrade --proxy=http://proxy.example.com:80
2.4.2. При необходимости установите Oracle Client
По умолчанию python-oracledb работает в тонком режиме (Thin Mode), который напрямую подключается к Oracle Database, поэтому дополнительная установка не требуется. Однако для использования дополнительных функций, доступных в толстом режиме, необходимо установить библиотеки Oracle Client. Поддерживаются версии Oracle Client 23, 21, 19, 18, 12 и 11.2.

Если ваша база данных находится на удаленном компьютере, загрузите бесплатный пакет Oracle Instant Client «Basic» или «Basic Light» для архитектуры вашей операционной системы.

В качестве альтернативы можно использовать клиентские библиотеки, уже доступные в локально установленной базе данных, например в бесплатной версии Oracle AI Database Free .

Чтобы использовать python-oracledb в режиме Thick, необходимо вызвать oracledb.init_oracle_client()в приложении соответствующий запрос (см. раздел Включение режима Thick в python-oracledb) . Например:

import oracledb

oracledb.init_oracle_client()
В Linux не передавайте lib_dirпараметр init_oracle_client(). Библиотеки клиента Oracle в Linux должны быть указаны в системном пути поиска библиотек до запуска процесса Python.

2.4.2.1. ZIP-файлы Oracle Instant Client
Чтобы использовать режим python-oracledb Thick с zip-файлами Oracle Instant Client:

Загрузите zip-файл Oracle 23, 21, 19, 18, 12 или 11.2 «Basic» или «Basic Light», соответствующий вашей 64-битной или 32-битной архитектуре Python:

Linux 64-бит (x86-64)

Linux 32-бит (x86)

Linux Arm 64-бит (aarch64)

Oracle Instant Client версии 23 будет подключаться к Oracle Database 19 или более поздней версии. Oracle Instant Client версии 19 будет подключаться к Oracle Database 11.2 или более поздней версии.

Версии Oracle Database 23 и 19 являются версиями с долгосрочной поддержкой. Обратите внимание, что 32-разрядные клиенты недоступны ни на одной платформе для Oracle Database версии 23, однако вы можете использовать более старые 32-разрядные клиенты для подключения к этой версии базы данных.

Рекомендуется следить за последними обновлениями Oracle Instant Client желаемой вами основной версии.

Распакуйте пакет в отдельный каталог, доступный вашему приложению. Например:

mkdir -p /opt/oracle
cd /opt/oracle
unzip instantclient-basic-linux.x64-21.6.0.0.0.zip
Примечание. Ограничения ОС могут помешать открытию клиентских библиотек Oracle, установленных по небезопасным путям, например, из каталога пользователя. Возможно, вам потребуется выполнить установку в каталог, например /opt, или /usr/local.

Установите libaioпакет с помощью sudo или от имени пользователя root. Например:

sudo dnf install libaio
В некоторых дистрибутивах Linux этот пакет называется libaio1или libaio1t64.

Если вы libaio1t64установили Instant Client, но он по-прежнему не может его найти libaio.so.1, вам может потребоваться создать символическую ссылку с libaio.so.1t64на libaio.so.1(аналогично патчу здесь ).

При использовании Oracle Instant Client 19 в версиях Linux, таких как Oracle Linux 8, вам может потребоваться вручную установить libnslпакет, чтобы сделать libnsl.soдоступным:

sudo dnf install libnsl
Если на компьютере нет другого программного обеспечения Oracle, которое может быть затронуто, добавьте Instant Client в путь ссылки среды выполнения. Например, с помощью sudo или от имени пользователя root:

sudo sh -c "echo /opt/oracle/instantclient_21_6 > /etc/ld.so.conf.d/oracle-instantclient.conf"
sudo ldconfig
В качестве альтернативы можно задать переменную окружения, LD_LIBRARY_PATHсоответствующую каталогу версии Instant Client. Например:

export LD_LIBRARY_PATH=/opt/oracle/instantclient_21_6:$LD_LIBRARY_PATH
Убедитесь, что это установлено в каждой оболочке, вызывающей Python. Веб-серверы и другие демоны часто сбрасывают переменные окружения, поэтому ldconfigобычно предпочтительнее использовать .

Если вы используете дополнительные файлы конфигурации Oracle, такие как tnsnames.ora, sqlnet.ora, или oraaccess.xmlс Instant Client, поместите эти файлы в доступный каталог, например, в /opt/oracle/your_config_dir. Затем используйте:

import oracledb

oracledb.init_oracle_client(config_dir="/home/your_username/oracle/your_config_dir")
или задайте переменную окружения TNS_ADMINдля этого имени каталога.

В качестве альтернативы, поместите файлы в network/adminподкаталог Instant Client, например, в /opt/oracle/instantclient_21_6/network/admin. Это каталог конфигурации Oracle по умолчанию для исполняемых файлов, связанных с этим Instant Client.

Подайте oracledb.init_oracle_client()заявку, если она еще не использована.

2.4.2.2. Пакеты RPM для Oracle Instant Client
Чтобы использовать python-oracledb с RPM-пакетами Oracle Instant Client:

1a. Загрузите и установите Oracle Instant Client «Basic» или «Basic Light».

Загрузите и установите RPM-пакет Oracle 23, 21, 19, 18, 12 или 11.2 «Basic» или «Basic Light», соответствующий вашей архитектуре Python, с сайта:

Linux 64-бит (x86-64)

Linux Arm 64-бит (aarch64)

Установите загруженный RPM-пакет с помощью sudo или от имени пользователя root. Например:

sudo dnf install oracle-instantclient-basic-23.9.0.25.07-1.el9.x86_64.rpm
Dnf автоматически установит необходимые зависимости, такие как libaio.

Рекомендуется следить за последними обновлениями Oracle Instant Client желаемой вами основной версии.

Версии Oracle Database 23 и 19 являются версиями с долгосрочной поддержкой. Oracle Instant Client версии 23 будет подключаться к Oracle Database 19 или более поздней версии. Oracle Instant Client версии 19 будет подключаться к Oracle Database версии 11.2 или более поздней версии.

Примечание. 32-разрядные клиенты недоступны ни на одной платформе для Oracle Database версии 23, однако вы можете использовать более старые 32-разрядные клиенты для подключения к этой версии базы данных.

1б. Альтернативный вариант — установить Instant Client с сервера yum от Oracle.

См. инструкции по Oracle Database Instant Client для Oracle Linux . Репозитории:

Instant Client версии 23 на Oracle Linux 9

Instant Client 23 для Oracle Linux 9 (x86-64)

Instant Client 23 для Oracle Linux 9 (aarch64)

sudo dnf install oracle-instantclient-release-23ai-el9
sudo dnf install oracle-instantclient-basic
Instant Client версии 19 на Oracle Linux 9:

Instant Client 19 для Oracle Linux 9 (x86-64)

Instant Client 19 для Oracle Linux 9 (aarch64)

sudo dnf install oracle-instantclient-release-el9
sudo dnf install oracle-instantclient19.XX-basic
Instant Client версии 23 на Oracle Linux 8

Instant Client 23 для Oracle Linux 8 (x86-64)

Instant Client 23 для Oracle Linux 8 (aarch64)

sudo dnf install oracle-instantclient-release-23ai-el8
sudo dnf install oracle-instantclient-basic
Instant Client версии 19 на Oracle Linux 8:

Instant Client 19 для Oracle Linux 8 (x86-64)

Instant Client 19 для Oracle Linux 8 (aarch64)

sudo dnf install -y oracle-release-el8
sudo dnf install -y oracle-instantclient19.XX-basic
При использовании Oracle Instant Client 19 в версиях Linux, таких как Oracle Linux 8, вам может потребоваться вручную установить libnslпакет, чтобы сделать libnsl.soдоступным:

sudo dnf install libnsl
Для Instant Client 19 и более поздних версий путь поиска системной библиотеки настраивается автоматически во время установки.

Для более старых версий, если на компьютере нет другого программного обеспечения Oracle, которое может быть затронуто, добавьте Instant Client в путь ссылки среды выполнения. Например, с помощью sudo или от имени пользователя root:

sudo sh -c "echo /usr/lib/oracle/18.5/client64/lib > /etc/ld.so.conf.d/oracle-instantclient.conf"
sudo ldconfig
В качестве альтернативы, для версии 18 и более ранних версий, в каждой оболочке, работающей с Python, необходимо задать переменную окружения, LD_LIBRARY_PATHсоответствующую каталогу версии Instant Client. Например:

export LD_LIBRARY_PATH=/usr/lib/oracle/18.5/client64/lib:$LD_LIBRARY_PATH
Веб-серверы и другие демоны обычно сбрасывают переменные среды, поэтому ldconfigвместо этого предпочтительнее использовать .

Если вы используете дополнительные файлы конфигурации Oracle, такие как tnsnames.ora, sqlnet.oraили oraaccess.xmlс Instant Client, поместите их в доступный каталог, например /opt/oracle/your_config_dir, . Тогда код вашего приложения сможет использовать:

import oracledb

oracledb.init_oracle_client(config_dir="/opt/oracle/your_config_dir")
или вы можете задать переменную окружения TNS_ADMINдля этого имени каталога.

В качестве альтернативы, поместите файлы в network/adminподкаталог Instant Client, например, в /usr/lib/oracle/21/client64/lib/network/admin. Это каталог конфигурации Oracle по умолчанию для исполняемых файлов, связанных с этим Instant Client.

Подайте oracledb.init_oracle_client()заявку, если она еще не использована.

2.4.2.3 Локальная база данных или полный клиент Oracle
Приложения Python-oracledb могут использовать библиотеки Oracle Client 23, 21, 19, 18, 12 или 11.2 из локальной базы данных Oracle или полной установки Oracle Client (например, установленной с помощью графического установщика Oracle).

Библиотеки должны быть 32- или 64-разрядными, в соответствии с архитектурой вашего Python. Обратите внимание: 32-разрядные клиенты недоступны ни на одной платформе для Oracle Database версии 23, однако вы можете использовать более старые 32-разрядные клиенты для подключения к этой версии базы данных.

Установите необходимые переменные среды Oracle, такие как ORACLE_HOME, запустив скрипт среды Oracle. Например:

source /usr/local/bin/oraenv
Для Oracle Database Express Edition («XE») 11.2 выполните:

source /u01/app/oracle/product/11.2.0/xe/bin/oracle_env.sh
Дополнительные файлы конфигурации Oracle, такие как tnsnames.ora, sqlnet.ora, или oraaccess.xmlмогут быть помещены в $ORACLE_HOME/network/admin.

В качестве альтернативы, файлы конфигурации Oracle можно разместить в другом доступном каталоге. Затем укажите TNS_ADMINимя этого каталога в переменной окружения.

Подайте oracledb.init_oracle_client()заявку, если она еще не использована.

2.5. Установка python-oracledb на Windows
2.5.1. Установка python-oracledb
Используйте пакет pip Python для установки python-oracledb из репозитория пакетов Python PyPI :

python -m pip install oracledb --upgrade
Если вы используете прокси-сервер, воспользуйтесь этой --proxyопцией. Например:

python -m pip install oracledb --upgrade --proxy=http://proxy.example.com:80
Это позволит загрузить и установить предварительно скомпилированный двоичный файл, если он доступен для вашей архитектуры. Если предварительно скомпилированный двоичный файл недоступен, исходный код будет загружен, скомпилирован, а полученный двоичный файл будет установлен.

2.5.2. При необходимости установите Oracle Client
По умолчанию python-oracledb работает в тонком режиме, который подключается напрямую к Oracle Database, поэтому дополнительная установка не требуется. Однако для использования дополнительных функций, доступных в толстом режиме, необходимо установить библиотеки Oracle Client. Можно использовать Oracle Client версий 21, 19, 18, 12 и 11.2.

Если ваша база данных находится на удаленном компьютере, загрузите бесплатный пакет Oracle Instant Client «Basic» или «Basic Light» для архитектуры вашей операционной системы.

В качестве альтернативы можно использовать клиентские библиотеки, уже доступные в локально установленной базе данных, например в бесплатной версии Oracle Database Express Edition («XE») .

Чтобы использовать python-oracledb в режиме Thick, необходимо вызвать oracledb.init_oracle_client()в приложении соответствующий запрос (см. раздел Включение режима Thick в python-oracledb) . Например:

import oracledb

oracledb.init_oracle_client()
В Windows вы можете предпочесть передать lib_dirпараметр в вызове, как показано ниже.

2.5.2.1. ZIP-файлы Oracle Instant Client
Чтобы использовать python-oracledb в толстом режиме с zip-файлами Oracle Instant Client:

Загрузите zip-архив Oracle 21, 19, 18, 12 или 11.2 «Basic» или «Basic Light»: 64- или 32-разрядную версию , соответствующую вашей архитектуре Python. Обратите внимание, что 32-разрядные клиенты недоступны ни на одной платформе для Oracle Database версии 23, однако вы можете использовать более старые 32-разрядные клиенты для подключения к этой версии базы данных.

Рекомендуется последняя версия. Oracle Instant Client 19 будет подключаться к Oracle Database 11.2 или более поздней версии.

Распакуйте пакет в каталог, доступный вашему приложению. Например, распакуйте instantclient-basic-windows.x64-19.22.0.0.0dbru.zipв C:\oracle\instantclient_19_22.

Для работы библиотек Oracle Instant Client требуется распространяемый пакет Visual Studio с 64- или 32-разрядной архитектурой, соответствующий архитектуре Instant Client. Для каждой версии Instant Client требуется своя версия распространяемого пакета:

Для Instant Client 21 установите VS 2019 или более позднюю версию.

Для Instant Client 19 установите VS 2017

Для Instant Client 18 или 12.2 установите VS 2013

Для Instant Client 12.1 установите VS 2010

Для Instant Client 11.2 установите VS 2005 64-bit

2.5.2.1.1. Настройка Oracle Instant Client
Существует несколько альтернативных способов сообщить python-oracledb, где находятся библиотеки клиента Oracle, см. раздел Инициализация python-oracledb .

С помощью Oracle Instant Client вы можете использовать oracledb.init_oracle_client()в своем приложении, например:

import oracledb

oracledb.init_oracle_client(lib_dir=r"C:\oracle\instantclient_19_22")
Обратите внимание, что используется «сырая» строка, поскольку в пути встречаются обратные косые черты.

В качестве альтернативы добавьте каталог Oracle Instant Client в PATH переменную окружения. Этот каталог должен предшествовать PATHвсем остальным каталогам Oracle. Перезапустите все открытые окна командной строки.

Обновите свое приложение, вызвав init_oracle_client(), что включит режим python-oracledb Thick:

import oracledb

oracledb.init_oracle_client()
Другой способ установки PATH— использование пакетного файла, который устанавливает его перед выполнением Python, например:

REM mypy.bat
SET PATH=C:\oracle\instantclient_19_22;%PATH%
python %*
Вызывайте этот пакетный файл каждый раз, когда хотите запустить Python.

Обновите свое приложение, вызвав init_oracle_client(), что включит режим python-oracledb Thick:

import oracledb

oracledb.init_oracle_client()
Если вы используете дополнительные файлы конфигурации Oracle, такие как tnsnames.ora, sqlnet.ora, или oraaccess.xmlс Instant Client, поместите эти файлы в доступный каталог, например, в C:\oracle\your_config_dir. Затем используйте:

import oracledb

oracledb.init_oracle_client(lib_dir=r"C:\oracle\instantclient_19_22",
                            config_dir=r"C:\oracle\your_config_dir")
или задайте переменную окружения TNS_ADMINдля этого имени каталога.

В качестве альтернативы, поместите файлы в network\adminподкаталог Instant Client, например, в C:\oracle\instantclient_19_22\network\admin. Это каталог конфигурации Oracle по умолчанию для исполняемых файлов, связанных с этим Instant Client.

2.5.2.2 Локальная база данных или полный клиент Oracle
Приложения Python-oracledb в толстом режиме могут использовать библиотеки Oracle Client 21, 19, 18, 12 или 11.2 из локальной базы данных Oracle или полного клиента Oracle (например, установленного с помощью установщика графического интерфейса Oracle).

Библиотеки Oracle должны быть 32- или 64-разрядными, в соответствии с архитектурой вашего Python. Обратите внимание: 32-разрядные клиенты недоступны ни на одной платформе для Oracle Database версии 23, однако вы можете использовать более старые 32-разрядные клиенты для подключения к этой версии базы данных.

Установите переменную среды PATHтак, чтобы она включала путь, содержащий OCI.DLL, если он еще не установлен.

Перезапустите все открытые окна командной строки.

Дополнительные файлы конфигурации Oracle, такие как tnsnames.ora, sqlnet.ora, или oraaccess.xmlмогут быть размещены в network\adminподкаталоге установки программного обеспечения Oracle Database.

В качестве альтернативы передайте config_dirзначение , oracledb.init_oracle_client() как показано в предыдущем разделе, или укажите TNS_ADMINимя каталога.

Чтобы использовать python-oracledb в толстом режиме, необходимо вызвать oracledb.init_oracle_client()в своем приложении соответствующий раздел. См. Включение толстого режима python-oracledb .

import oracledb

oracledb.init_oracle_client()
2.6. Установка python-oracledb на macOS
Python-oracledb доступен как универсальный двоичный файл для Python 3.9 или более поздней версии на архитектурах Apple macOS Intel x86-64 и Apple macOS ARM64 (M1, M2, M3, M4).

2.6.1. Установка python-oracledb
Используйте пакет pip Python для установки python-oracledb из репозитория пакетов Python PyPI :

python -m pip install oracledb --upgrade
Эта --userопция может быть полезна, если у вас нет разрешения на запись в системные каталоги:

python -m pip install oracledb --upgrade --user
Если вы используете прокси-сервер, воспользуйтесь этой --proxyопцией. Например:

python -m pip install oracledb --upgrade --user --proxy=http://proxy.example.com:80
Для установки Python в систему вам может потребоваться использовать /usr/bin/python3 вместо python:

/usr/bin/python3 -m pip install oracledb --upgrade --user
2.6.2. При необходимости установите Oracle Client
По умолчанию python-oracledb работает в тонком режиме, который подключается напрямую к Oracle Database, поэтому дополнительная установка не требуется. Однако для использования дополнительных функций, доступных в толстом режиме, необходимо установить клиентские библиотеки Oracle.

Библиотеки можно получить из пакета Oracle Instant Client Basic или Basic Light . Ниже приведены шаги по установке Basic .

2.6.2.1. Мгновенная установка клиента по сценарию на macOS ARM64
Установку Instant Client можно выполнить с помощью скрипта. Откройте окно терминала и выполните:

cd $HOME/Downloads
curl -O https://download.oracle.com/otn_software/mac/instantclient/233023/instantclient-basic-macos.arm64-23.3.0.23.09.dmg
hdiutil mount instantclient-basic-macos.arm64-23.3.0.23.09.dmg
/Volumes/instantclient-basic-macos.arm64-23.3.0.23.09/install_ic.sh
hdiutil unmount /Volumes/instantclient-basic-macos.arm64-23.3.0.23.09
Обратите внимание, что вам следует использовать последнюю доступную версию DMG.

Если у вас смонтировано несколько DMG-пакетов Instant Client, вам достаточно запустить его только install_ic.shодин раз. Он скопирует все смонтированные DMG-пакеты Instant Client одновременно.

Каталог Instant Client будет выглядеть так $HOME/Downloads/instantclient_23_3: . Приложения могут не иметь доступа к этому Downloadsкаталогу, поэтому вам следует переместить Instant Client в удобное место.

2.6.2.2. Ручная установка клиента Instant Client на macOS ARM64
Загрузите последнюю версию пакета Instant Client Basic ARM64 DMG с сайта Oracle .

С помощью Finder дважды щелкните по DMG-файлу, чтобы смонтировать его.

Откройте окно терминала и запустите скрипт установки в смонтированном пакете, например, если вы загрузили версию 23.3:

/Volumes/instantclient-basic-macos.arm64-23.3.0.23.09/install_ic.sh
Каталог Instant Client будет выглядеть так $HOME/Downloads/instantclient_23_3: . Приложения могут не иметь доступа к этому Downloadsкаталогу, поэтому вам следует переместить Instant Client в удобное место.

Используя Finder, извлеките смонтированный пакет Instant Client.

Если у вас смонтировано несколько DMG-пакетов Instant Client, вам достаточно запустить его только install_ic.shодин раз. Он скопирует все смонтированные DMG-пакеты Instant Client одновременно.

2.6.2.3. Мгновенная установка клиента по сценарию на macOS Intel x86-64
Установку Instant Client можно выполнить с помощью скрипта. Откройте окно терминала и выполните:

cd $HOME/Downloads
curl -O https://download.oracle.com/otn_software/mac/instantclient/1916000/instantclient-basic-macos.x64-19.16.0.0.0dbru.dmg
hdiutil mount instantclient-basic-macos.x64-19.16.0.0.0dbru.dmg
/Volumes/instantclient-basic-macos.x64-19.16.0.0.0dbru/install_ic.sh
hdiutil unmount /Volumes/instantclient-basic-macos.x64-19.16.0.0.0dbru
Обратите внимание, что вам следует использовать последнюю доступную версию DMG.

Если у вас смонтировано несколько DMG-пакетов Instant Client, вам достаточно запустить его только install_ic.shодин раз. Он скопирует все смонтированные DMG-пакеты Instant Client одновременно.

Каталог Instant Client будет находиться в $HOME/Downloads/instantclient_19_16. Приложения могут не иметь доступа к этому Downloadsкаталогу, поэтому вам следует переместить Instant Client в удобное место.

2.6.2.4. Ручная установка Instant Client на macOS Intel x86-64
Загрузите последнюю версию пакета Instant Client Basic Intel 64-bit DMG с сайта Oracle .

С помощью Finder дважды щелкните по DMG-файлу, чтобы смонтировать его.

Откройте окно терминала и запустите скрипт установки в смонтированном пакете, например:

/Volumes/instantclient-basic-macos.x64-19.16.0.0.0dbru/install_ic.sh
Каталог Instant Client будет находиться в $HOME/Downloads/instantclient_19_16. Приложения могут не иметь доступа к этому Downloadsкаталогу, поэтому вам следует переместить Instant Client в удобное место.

Используя Finder, извлеките смонтированный пакет Instant Client.

Если у вас смонтировано несколько DMG-пакетов Instant Client, вам достаточно запустить его только install_ic.shодин раз. Он скопирует все смонтированные DMG-пакеты Instant Client одновременно.

2.6.3. Настройка Oracle Instant Client
Ваше приложение должно загружать установленные библиотеки Oracle Instant Client. При необходимости оно может указывать внешние файлы конфигурации.

Подайте заявку oracledb.init_oracle_client()по телефону:

import oracledb

oracledb.init_oracle_client(lib_dir="/Users/your_username/Downloads/instantclient_23_3")
Если вы используете дополнительные файлы конфигурации Oracle, такие как tnsnames.ora, sqlnet.ora, или oraaccess.xmlс Oracle Instant Client, поместите файлы в доступный каталог, например, в /Users/your_username/oracle/your_config_dir. Затем используйте:

import oracledb

oracledb.init_oracle_client(lib_dir="/Users/your_username/Downloads/instantclient_23_3",
                            config_dir="/Users/your_username/oracle/your_config_dir")
Или задайте переменную окружения TNS_ADMINдля этого имени каталога.

В качестве альтернативы, поместите файлы в network/adminподкаталог Oracle Instant Client, например, в /Users/your_username/Downloads/instantclient_23_3/network/admin. Это каталог конфигурации Oracle по умолчанию для исполняемых файлов, связанных с этим Instant Client.

2.7. Установка python-oracledb без доступа в Интернет
Чтобы установить python-oracledb на компьютер, не подключенный к Интернету, скачайте пакет python-oracledb wheel из репозитория пакетов Python PyPI . Используйте файл, соответствующий вашей операционной системе и версии Python. Перенесите этот файл на компьютер, не подключенный к Интернету, и установите его с помощью:

python -m pip install "<file_name>"
Вам также потребуется выполнить аналогичный шаг для установки необходимого пакета криптографии и его зависимостей.

Затем следуйте общим инструкциям по установке платформы python-oracledb, чтобы установить клиентские библиотеки Oracle. Это необходимо только в том случае, если вы планируете использовать режим python-oracledb Thick .

2.8. Установка python-oracledb без пакета Cryptography
Если пакет криптографии Python недоступен, python-oracledb всё равно можно установить, но использовать его можно только в режиме Thick. Попытка использовать режим Thin приведёт к ошибке .DPY-3016: python-oracledb thin mode cannot be used because the cryptography package is not installed

Чтобы использовать python-oracledb без пакета криптографии:

Установите python-oracledb, используя опцию pip --no-deps, например:

python -m pip install oracledb --no-deps
Затем необходимо установить клиентские библиотеки Oracle. См. предыдущие разделы.

Добавьте вызов в oracledb.init_oracle_client()свое приложение, см. Включение толстого режима python-oracledb .

2.9 Установка из исходного кода
Для платформ, на которых нет готовых двоичных файлов в PyPI , использование обычной команды загрузит исходный пакет python-oracledb, соберет и установит его.python -m pip install oracledb

В качестве альтернативы, чтобы создать собственные файлы пакета для установки, вы можете собрать и установить python-oracledb либо локально из исходного кода , либо с помощью предварительно предоставленного действия GitHub , которое собирает пакеты для всех архитектур и версий Python.

2.9.1. Локальная сборка пакета python-oracledb
Установите компилятор C, совместимый с C99.

Загрузите исходный код, используя один из следующих вариантов:

Вы можете клонировать исходный код с GitHub :

git clone --recurse-submodules https://github.com/oracle/python-oracledb.git
Кроме того, вы можете вручную загрузить исходный zip- файл с GitHub.

В этом случае вам также потребуется загрузить исходный zip-файл ODPI-C и поместить извлеченное содержимое в odpiподкаталог, например, в python-oracledb-main/src/oracledb/impl/thick/odpi.

В качестве альтернативы, клонируйте исходный код с opensource.oracle.com , который является зеркалом GitHub:

git clone --recurse-submodules https://opensource.oracle.com/git/oracle/python-oracledb.git
git checkout main
Альтернативно, исходный пакет python-oracledb можно вручную загрузить из PyPI.

Перейдите на страницу загрузки файлов PyPI python-oracledb , загрузите архив исходного пакета и распакуйте его.

Имея доступ к исходному коду, соберите пакет python-oracledb, выполнив:

cd python-oracledb               # the name may vary depending on the download
python -m pip install build --upgrade
# export PYO_COMPILE_ARGS='-g0'  # optionally set any compilation arguments
python -m build
В подкаталоге создаётся пакет python-oracledb wheel dist. Например, при использовании Python 3.12 в macOS у вас может быть файл dist/oracledb-3.1.0-cp312-cp312-macosx_14_0_arm64.whl.

Установите этот пакет:

python -m pip install dist/oracledb-3.1.0-cp312-cp312-macosx_14_0_arm64.whl
Пакет также можно установить на любой компьютер, имеющий ту же архитектуру и версию Python, что и машина сборки.

2.9.2 Сборка пакетов python-oracledb с использованием GitHub Actions
В репозитории GitHub python-oracledb имеется действие-компоновщик, которое использует инфраструктуру GitHub для сборки пакетов python-oracledb для всех архитектур и версий Python.

Создайте форк репозитория python-oracledb . Также создайте форк репозитория ODPI-C , сохранив имя по умолчанию.

В форке python-oracledb перейдите на вкладку «Действия» . Если вы используете «Действия» впервые, подтвердите их включение.https://github.com/<your name>/python-oracledb/actions/

В списке «Все рабочие процессы» слева выберите запись «Сборка пакетов python-oracledb».

Перейдите в раскрывающийся список «Запустить рабочий процесс», выберите ветвь, из которой будет выполняться сборка (например, «main»), и запустите рабочий процесс.

При необходимости отредактируйте список целевых пакетов в поле ввода и удалите те, которые вам не нужны. Например, удалите «Linux», если вам не нужны пакеты Linux.

После завершения сборки скачайте артефакт «python-oracledb-wheels», распакуйте его и установите версию, соответствующую вашей архитектуре и версии Python. Например, при использовании Python 3.12 в macOS установите:

python -m pip install oracledb-3.1.0-cp312-cp312-macosx_10_13_universal2.whl
2.10 Использование контейнеров python-oracledb
Dockerfiles, демонстрирующие установку Python и python-oracledb в Oracle Linux, доступны по адресу github.com/oracle/docker-images/tree/main/OracleLinuxDevelopers .

Контейнеры, созданные на основе этих Dockerfiles, можно извлечь из реестра контейнеров GitHub:

oraclelinux9-python

oraclelinux8-python

Например, вы можете извлечь контейнер для Python 3.12 в Oracle Linux 9, используя:

docker pull ghcr.io/oracle/oraclelinux9-python:3.12-oracledb
Или используйте его в Dockerfile следующим образом:

FROM ghcr.io/oracle/oraclelinux9-python:3.12-oracledb
Контейнеры для образцов

Имеется два контейнера python-oracledb с образцами, расположенными в /samples/containers .

2.11 Установка модулей централизованного поставщика конфигурации для python-oracledb
Чтобы использовать python-oracledb с поставщиком централизованной конфигурации , необходимо установить необходимые модули для предпочитаемого вами поставщика, как подробно описано ниже.

2.11.1 Установка модулей для поставщика централизованной конфигурации хранилища объектов OCI
Чтобы python-oracledb использовал поставщик конфигурации хранилища объектов Oracle Cloud Infrastructure (OCI) , необходимо установить пакет OCI , используя дополнительную [oci_config]зависимость:

python -m pip install oracledb[oci_config]
Информацию об использовании этого поставщика конфигурации с python-oracledb см . в разделе Использование централизованного поставщика конфигурации хранилища объектов OCI .

2.11.2 Установка модулей для поставщика централизованной конфигурации приложений Azure
Чтобы python-oracledb использовал поставщик конфигурации приложений Azure , необходимо установить пакеты Azure App Configuration , Azure Identity и Azure Key Vault Secrets с помощью дополнительной [azure_config]зависимости:

python -m pip install oracledb[azure_config]
Информацию об использовании этого поставщика конфигурации с python-oracledb см . в разделе Использование централизованного поставщика конфигурации приложений Azure .

2.12 Установка модулей аутентификации Cloud Native для python-oracledb
Чтобы использовать плагин аутентификации Cloud Native на базе python-oracledb, необходимо установить необходимые модули для предпочитаемого вами плагина, как описано ниже.

2.12.1 Установка модулей для плагина аутентификации OCI Cloud Native
Чтобы python-oracledb использовал плагин аутентификации OCI Cloud Native, необходимо установить пакет Python SDK для Oracle Cloud Infrastructure, используя [oci_auth] зависимость:

python -m pip install oracledb[oci_auth]
При необходимости ознакомьтесь с инструкциями по установке OCI SDK .

Дополнительные сведения об использовании плагина в python-oracledb см . в разделе OCI Cloud Native Authentication с плагином oci_tokens .

2.12.2 Установка модулей для плагина аутентификации Azure Cloud Native
Чтобы python-oracledb использовал подключаемый модуль аутентификации Azure Cloud Native, необходимо установить пакет библиотеки аутентификации Microsoft (MSAL) для Python, используя дополнительную [azure_auth]зависимость:

python -m pip install oracledb[azure_auth]
При необходимости ознакомьтесь с инструкциями по установке Microsoft MSAL .

Дополнительные сведения об использовании плагина в python-oracledb см . в статье Azure Cloud Native Authentication с подключаемым модулем azure_tokens .
