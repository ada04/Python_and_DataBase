# Модуль для подключения к Oracle Database из Python
## 1. Введение
   
Python-oracledb -  это модуль Python с [открытым исходным кодом](https://python-oracledb.readthedocs.io/en/latest/license.html#license), обеспечивающий доступ к базе данных Oracle без необходимости использования дополнительных библиотек. Модуль разработан на Cython для обеспечения безопасности и скорости. Он лёгкий и высокопроизводительный. Он стабилен, хорошо протестирован и имеет подробную документацию. 

Модуль соответствует [спецификации Python Database API v2.0](https://www.python.org/dev/peps/pep-0249/) со значительным количеством дополнений и несколькими незначительными исключениями. Он используется многими фреймворками Python, SQL-генераторами, ORM-объектами и библиотеками.

Python-oracledb обладает богатым набором функций, которые просты в использовании. Он позволяет управлять выполнением операторов SQL и PL/SQL, работать с фреймами данных, быстро получать данные, вызывать document API в стиле NoSQL, организовывать очереди сообщений, получать уведомления базы данных, а также запускать и останавливать базу данных. Он также обладает функциями обеспечения высокой доступности и безопасности. Поддерживаются синхронный и параллельный стили кодирования. Операции с базой данных могут быть конвейерными.

Модуль доступен в стандартных репозиториях пакетов, включая [PyPI](https://pypi.org/project/oracledb/), [conda-forge](https://anaconda.org/conda-forge/oracledb), and [yum.oracle.com](https://yum.oracle.com/oracle-linux-python.html). Исходный код размещён на [github.com/oracle/python-oracledb](https://github.com/oracle/python-oracledb).

В настоящее время этот модуль тестируется с Python 3.9, 3.10, 3.11, 3.12, 3.13 и 3.14 и Oracle Database версий 23, 21, 19, 18, 12 и 11.2. Предыдущие версии python-oracledb поддерживали более старые версии Python.

!_! Changes in python-oracledb releases can be found in the release notes.

Модуль python-oracledb ранее назывался cx_Oracle. cx_Oracle устарел и не рекомендуется для использования в новых проектах. Информация об обновлении см. в разделе [Обновление с cx_Oracle 8.3 до python-oracledb](https://python-oracledb.readthedocs.io/en/latest/user_guide/appendix_c.html#upgrading83) .

### 1.1. Начало работы
See Quick Start python-oracledb Installation.

Runnable examples are in the GitHub samples directory. A tutorial Python and Oracle Database Tutorial: The New Wave of Scripting is also available.

### 1.2. Архитектура
Python-oracledb is a ‘Thin’ driver with an optional ‘Thick’ mode enabled by an application setting.

#### 1.2.1. Режим тонкого клиента python-oracledb
By default, python-oracledb allows connecting directly to Oracle Database 12.1 or later. This Thin mode does not need Oracle Client libraries.

![](https://python-oracledb.readthedocs.io/en/latest/_images/python-oracledb-thin-arch.png)
Fig. 1.1 Architecture of the python-oracledb driver in Thin mode

The figure shows the architecture of python-oracledb. Users interact with a Python application, for example by making web requests. The application program makes calls to python-oracledb functions. The connection from python-oracledb Thin mode to Oracle Database is established directly by python-oracledb over the Oracle Net protocol. The database can be on the same machine as Python, or it can be remote.

The behavior of Oracle Net can optionally be configured with application settings, or by using a tnsnames.ora file, see Optional Oracle Net Configuration Files.

#### 1.2.2. Режим толстого клиента python-oracledb
Python-oracledb is said to be in ‘Thick’ mode when it links with Oracle Client libraries. An application script runtime option enables this mode by loading the libraries, see Enabling python-oracledb Thick mode. This gives you some additional functionality. Depending on the version of the Oracle Client libraries, this mode of python-oracledb can connect to Oracle Database 9.2 or later.

architecture of the python-oracledb driver in Thick mode
Fig. 1.2 Architecture of the python-oracledb driver in Thick mode

The figure shows the architecture of the python-oracledb Thick mode. Users interact with a Python application, for example by making web requests. The application program makes calls to python-oracledb functions. Internally, python-oracledb dynamically loads Oracle Client libraries. Connections from python-oracledb Thick mode to Oracle Database are established by the Oracle Client libraries over the Oracle Net protocol. The database can be on the same machine as Python, or it can be remote.

To use python-oracledb Thick mode, the Oracle Client libraries must be installed separately, see Installing python-oracledb. The libraries can be from an installation of Oracle Instant Client, from a full Oracle Client installation (such as installed by Oracle’s GUI installer), or even from an Oracle Database installation (if Python is running on the same machine as the database). Oracle’s standard client-server version interoperability allows connection to both older and newer databases from different Oracle Client library versions.

Some behaviors of the Oracle Client libraries can optionally be configured with an oraaccess.xml file, for example to enable auto-tuning of a statement cache. See Optional Oracle Client Configuration File.

The behavior of Oracle Net can optionally be configured with files such as tnsnames.ora and sqlnet.ora, for example to enable network encryption. See Optional Oracle Net Configuration Files.

Oracle environment variables that are set before python-oracledb first creates a database connection may affect python-oracledb Thick mode behavior. See Oracle Environment Variables for python-oracledb.

### 1.3. Feature Highlights of python-oracledb
The python-oracledb feature highlights are:

Easy installation from PyPI and other repositories

Support for multiple Oracle Database versions

Supports the Python Database API v2.0 Specification with a considerable number of additions and a couple of exclusions

Works with common frameworks and ORMs

Execution of SQL and PL/SQL statements

Extensive Oracle data type support, including JSON, VECTOR, large objects (CLOB and BLOB) and binding of SQL objects

Connection management, including connection pooling

Oracle Database High Availability features

Full use of Oracle Network Service infrastructure, including encrypted network traffic

See Appendix A: Oracle Database Features Supported by python-oracledb for more information.
