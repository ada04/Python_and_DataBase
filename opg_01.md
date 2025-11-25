## 1. Introduction to the Python Driver for Oracle Database
   
The python-oracledb driver is an open source Python module that enables access to Oracle Database with no extra libraries needed. The module is built with Cython for safety and speed. It is lightweight and high-performance. It is stable, well tested, and has comprehensive documentation. The module is maintained by Oracle.

The module conforms to the Python Database API v2.0 Specification with a considerable number of additions and a couple of minor exclusions. It is used by many Python frameworks, SQL generators, ORMs, and libraries.

Python-oracledb has a rich feature set which is easy to use. It gives you control over SQL and PL/SQL statement execution; for working with data frames; for fast data ingestion; for calling NoSQL-style document APIs; for message queueing; for receiving database notifications; and for starting and stopping the database. It also has high availability and security features. Synchronous and concurrent coding styles are supported. Database operations can optionally be pipelined.

The module is available from standard package repositories including PyPI, conda-forge, and yum.oracle.com. The source code is hosted at github.com/oracle/python-oracledb.

This module is currently tested with Python 3.9, 3.10, 3.11, 3.12, 3.13, and 3.14 against Oracle Database version 23, 21, 19, 18, 12, and 11.2. Previous versions of python-oracledb supported older Python versions.

Changes in python-oracledb releases can be found in the release notes.

The python-oracledb driver is the renamed, successor to cx_Oracle. The cx_Oracle driver is obsolete and should not be used for new development. For upgrade information, see Upgrading from cx_Oracle 8.3 to python-oracledb.

### 1.1. Getting Started
See Quick Start python-oracledb Installation.

Runnable examples are in the GitHub samples directory. A tutorial Python and Oracle Database Tutorial: The New Wave of Scripting is also available.

### 1.2. Architecture
Python-oracledb is a ‘Thin’ driver with an optional ‘Thick’ mode enabled by an application setting.

#### 1.2.1. python-oracledb Thin Mode Architecture
By default, python-oracledb allows connecting directly to Oracle Database 12.1 or later. This Thin mode does not need Oracle Client libraries.

architecture of the python-oracledb driver in Thin mode
Fig. 1.1 Architecture of the python-oracledb driver in Thin mode

The figure shows the architecture of python-oracledb. Users interact with a Python application, for example by making web requests. The application program makes calls to python-oracledb functions. The connection from python-oracledb Thin mode to Oracle Database is established directly by python-oracledb over the Oracle Net protocol. The database can be on the same machine as Python, or it can be remote.

The behavior of Oracle Net can optionally be configured with application settings, or by using a tnsnames.ora file, see Optional Oracle Net Configuration Files.

#### 1.2.2. python-oracledb Thick Mode Architecture
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
