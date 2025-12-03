# Модуль для подключения к Oracle Database из Python
## 1. Введение
   
Python-oracledb -  это модуль Python с [открытым исходным кодом](https://python-oracledb.readthedocs.io/en/latest/license.html#license), обеспечивающий доступ к базе данных Oracle без необходимости использования дополнительных библиотек. Модуль разработан на Cython для обеспечения безопасности и скорости. Он лёгкий и высокопроизводительный. Он стабилен, хорошо протестирован и имеет подробную документацию. 

Модуль соответствует [спецификации Python Database API v2.0](https://www.python.org/dev/peps/pep-0249/) со значительным количеством дополнений и несколькими незначительными исключениями. Он используется многими фреймворками Python, SQL-генераторами, ORM-объектами и библиотеками.

Python-oracledb обладает богатым набором функций, которые просты в использовании. Он позволяет управлять выполнением операторов SQL и PL/SQL, работать с фреймами данных, быстро получать данные, вызывать document API в стиле NoSQL, организовывать очереди сообщений, получать уведомления базы данных, а также запускать и останавливать базу данных. Он также обладает функциями обеспечения высокой доступности и безопасности. Поддерживаются синхронный и параллельный стили кодирования. Операции с базой данных могут быть конвейерными.

Модуль доступен в стандартных репозиториях пакетов, включая [PyPI](https://pypi.org/project/oracledb/), [conda-forge](https://anaconda.org/conda-forge/oracledb), and [yum.oracle.com](https://yum.oracle.com/oracle-linux-python.html). Исходный код размещён на [github.com/oracle/python-oracledb](https://github.com/oracle/python-oracledb).

В настоящее время этот модуль тестируется с Python 3.9, 3.10, 3.11, 3.12, 3.13 и 3.14 и Oracle Database версий 23, 21, 19, 18, 12 и 11.2. Предыдущие версии python-oracledb поддерживали более старые версии Python.

!_! Changes in python-oracledb releases can be found in the release notes.

Модуль python-oracledb ранее назывался cx_Oracle. cx_Oracle устарел и не рекомендуется для использования в новых проектах. Информация об обновлении см. в разделе [Обновление с cx_Oracle 8.3 до python-oracledb](https://python-oracledb.readthedocs.io/en/latest/user_guide/appendix_c.html#upgrading83) .

### 1.1. <a id="odb_1_1">Начало работы</a>
!_! See Quick Start python-oracledb Installation.

Примеры для запуска находятся в [каталоге примеров на GitHub](https://github.com/oracle/python-oracledb/tree/main/samples). Также доступен обучающий [курс «Python и Oracle Database: The New Wave of Scripting»](https://github.com/oracle/python-oracledb/tree/main/samples).

### 1.2. Архитектура
Python-oracledb по умолчанию подключается к СУБД Oracle в режиме "тонкого" клиента, но можно в настройках изменить на режим "толстого" клиента.

#### 1.2.1. Подключение в режиме "тонкого" клиента python-oracledb
По умолчанию python-oracledb позволяет подключаться напрямую к Oracle Database 12.1 и более поздним версиям в режиме "тонкого" не требующего установки клиентских библиотек Oracle.

![](https://python-oracledb.readthedocs.io/en/latest/_images/python-oracledb-thin-arch.png)
Рсиунок 1.1 Архитектура работы python-oracledb в режиме "тонкого" клиента.

На рисунке показана архитектура работы python-oracledb в режиме "тонкого" клиента. Пользователи взаимодействуют с приложением Python, например, отправляя веб-запросы. Прикладная программа вызывает функции python-oracledb. Подключение python-oracledb в режиме "тонкого" клиента к Oracle Database осуществляется непосредственно python-oracledb по протоколу Oracle Net. База данных может располагаться на той же машине, что и Python, или быть удалённой.

Поведение Oracle Net можно дополнительно настроить с помощью параметров приложения или с помощью файла tnsnames.ora, см. [Optional Oracle Net Configuration Files](https://python-oracledb.readthedocs.io/en/latest/user_guide/initialization.html#optnetfiles).

#### 1.2.2. Подключение в режиме "толстого" клиента python-oracledb
Python-oracledb считается работающим в режиме «толстого» (Thick) клиента, когда при подключении к Oracle использует клиентские библиотеки (необходим установленный клиент Oracle). Этот режим включается во время выполнения скрипта приложения, загружая библиотеки (см. раздел [«Включение толстого режима python-oracledb»](https://python-oracledb.readthedocs.io/en/latest/user_guide/initialization.html#enablingthick)). Это предоставляет ряд [дополнительных функций](https://python-oracledb.readthedocs.io/en/latest/user_guide/appendix_a.html#featuresummary). В зависимости от версии клиентских библиотек Oracle, этот режим python-oracledb может подключаться к Oracle Database 9.2 или более поздней версии.

![architecture of the python-oracledb driver in Thick mode](https://python-oracledb.readthedocs.io/en/latest/_images/python-oracledb-thick-arch.png)
Рисунок 1.2 Архитектура подключения python-oracledb в режиме "толстого" клиента

На рисунке показана архитектура python-oracledb в режиме Thick mode. Пользователи взаимодействуют с приложением Python, например, отправляя веб-запросы. Прикладная программа вызывает функции python-oracledb. Внутренне python-oracledb динамически загружает клиентские библиотеки Oracle. Подключения из режима Thick в python-oracledb к базе данных Oracle устанавливаются клиентскими библиотеками Oracle по протоколу Oracle Net. База данных может располагаться на том же компьютере, что и Python, или быть удалённой.

Для использования режима Thick python-oracledb необходимо отдельно установить клиентские библиотеки Oracle (см. раздел [Установка python-oracledb](https://python-oracledb.readthedocs.io/en/latest/user_guide/installation.html#installation)). Библиотеки могут быть из установки [Oracle Instant Client](https://www.oracle.com/database/technologies/instant-client.html), из полной установки Oracle Client (например, установленной с помощью графического установщика Oracle) или даже из установки Oracle Database (если Python работает на том же компьютере, что и база данных). Стандартная совместимость клиент-серверных версий Oracle позволяет подключаться как к старым, так и к новым базам данных из разных версий клиентской библиотеки Oracle.

Некоторые функции клиентских библиотек Oracle можно настроить с помощью файла oraaccess.xml, например, для включения автоматической настройки кэша операторов. См. раздел [Дополнительный файл конфигурации клиента Oracle](https://python-oracledb.readthedocs.io/en/latest/user_guide/initialization.html#optclientfiles).

Поведение Oracle Net можно дополнительно настроить с помощью таких файлов, как tnsnames.ora и sqlnet.ora, например, для включения [сетевого шифрования](https://python-oracledb.readthedocs.io/en/latest/user_guide/connection_handling.html#netencrypt). См. [раздел Дополнительные файлы конфигурации Oracle Net](https://python-oracledb.readthedocs.io/en/latest/user_guide/initialization.html#optnetfiles).

Переменные среды Oracle, заданные до того, как python-oracledb впервые создаст соединение с базой данных, могут повлиять на поведение python-oracledb в режиме Thick. См. [раздел Переменные среды Oracle для python-oracledb](https://python-oracledb.readthedocs.io/en/latest/user_guide/initialization.html#envset).

### 1.3. Feature Highlights of python-oracledb
Основные возможности python-oracledb:
 * Простая установка из PyPI и других репозиториев
 * Поддержка нескольких версий Oracle Database
 * Поддерживает [спецификацию Python Database API v2.0](https://www.python.org/dev/peps/pep-0249/) со значительным количеством дополнений и несколькими исключениями.
 * Работает с распространенными фреймворками и ORM
 * Выполнение операторов SQL и PL/SQL
 * Расширенная поддержка типов данных Oracle, включая JSON, VECTOR, большие объекты (CLOB и BLOB) и привязку объектов SQL
 * Управление соединениями, включая пул соединений
 * Функции высокой доступности Oracle Database
 * Полное использование инфраструктуры Oracle Network Service, включая шифрование сетевого трафика

Дополнительные сведения см. [в Приложении A: Функции базы данных Oracle, поддерживаемые python-oracledb](https://python-oracledb.readthedocs.io/en/latest/user_guide/appendix_a.html#featuresummary).

| -:- | -:- | -:- |
|[Оглавление](odb_00.md)|    |[Установка python-oracledb](odb_02.md)|

