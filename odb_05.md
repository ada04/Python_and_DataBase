# В разработке

# 5. Варианты аутентификации

Аутентификация позволяет получить доступ к Oracle Database только авторизованным пользователям после успешной проверки их личности. В этом разделе подробно описаны различные варианты аутентификации Oracle Database, поддерживаемые в python-oracledb.

Библиотеки клиента Oracle, используемые режимом python-oracledb Thick, могут поддерживать дополнительные параметры аутентификации, которые настраиваются независимо от драйвера.

5.1 Аутентификация базы данных
Аутентификация в базе данных — это самый простой метод аутентификации, позволяющий пользователям подключаться к Oracle Database, используя действительное имя пользователя и связанный с ним пароль. Oracle Database сверяет имя пользователя и пароль, указанные в методе подключения python-oracledb, с информацией, хранящейся в базе данных. Подробнее см. в разделе «Аутентификация пользователей в базе данных» .

Отдельные и объединённые соединения можно создавать в тонком и толстом режимах python-oracledb с использованием аутентификации базы данных. Это можно сделать, указав имя пользователя базы данных и соответствующий пароль впараметрахuser, , ,или . Пример:passwordoracledb.connect()oracledb.create_pool()oracledb.connect_async()oracledb.create_pool_async()

import oracledb
import getpass

userpwd = getpass.getpass("Enter password: ")

connection = oracledb.connect(user="hr", password=userpwd,
                              dsn="dbhost.example.com/orclpdb")
5.2. Аутентификация через прокси-сервер
Аутентификация через прокси-сервер позволяет пользователю («пользователю сеанса») подключаться к Oracle Database, используя учетные данные «пользователя сеанса». Операторы будут выполняться от имени пользователя сеанса. Аутентификация через прокси-сервер обычно используется в трехуровневых приложениях, где один пользователь владеет схемой, а несколько конечных пользователей получают доступ к данным. Подробнее об аутентификации через прокси-сервер см. в документации Oracle .

Альтернативой использованию прокси-пользователей является установка Connection.client_identifierпосле подключения и использование его значения в операторах и в базе данных, например, для мониторинга .

В следующих примерах прокси-серверов используются эти схемы. Схеме mysessionuserпредоставлен доступ к использованию пароля myproxyuser:

CREATE USER myproxyuser IDENTIFIED BY myproxyuserpw;
GRANT CREATE SESSION TO myproxyuser;

CREATE USER mysessionuser IDENTIFIED BY itdoesntmatter;
GRANT CREATE SESSION TO mysessionuser;

ALTER USER mysessionuser GRANT CONNECT THROUGH myproxyuser;
После подключения к базе данных можно использовать следующий запрос для отображения сеанса и пользователей прокси:

SELECT SYS_CONTEXT('USERENV', 'PROXY_USER'),
       SYS_CONTEXT('USERENV', 'SESSION_USER')
FROM DUAL;
Примеры автономного подключения:

# Basic Authentication without a proxy
connection = oracledb.connect(user="myproxyuser", password="myproxyuserpw",
                              dsn="dbhost.example.com/orclpdb")
# PROXY_USER:   None
# SESSION_USER: MYPROXYUSER

# Basic Authentication with a proxy
connection = oracledb.connect(user="myproxyuser[mysessionuser]", password="myproxyuserpw",
                              dsn="dbhost.example.com/orclpdb")
# PROXY_USER:   MYPROXYUSER
# SESSION_USER: MYSESSIONUSER
Примеры объединенных соединений:

# Basic Authentication without a proxy
pool = oracledb.create_pool(user="myproxyuser", password="myproxyuserpw",
                            dsn="dbhost.example.com/orclpdb")
connection = pool.acquire()
# PROXY_USER:   None
# SESSION_USER: MYPROXYUSER

# Basic Authentication with proxy
pool = oracledb.create_pool(user="myproxyuser[mysessionuser]", password="myproxyuserpw",
                            dsn="dbhost.example.com/orclpdb",
                            homogeneous=False)

connection = pool.acquire()
# PROXY_USER:   MYPROXYUSER
# SESSION_USER: MYSESSIONUSER
Обратите внимание на использование неоднородного пула в примере выше. Это необходимо в данном сценарии.

5.3 Внешняя аутентификация
Вместо хранения имени пользователя и пароля базы данных в скриптах Python или переменных окружения, доступ к базе данных можно аутентифицировать с помощью внешней системы. Внешняя аутентификация позволяет приложениям проверять доступ пользователя с помощью внешнего хранилища паролей (например, Oracle Wallet ), операционной системы или внешней службы аутентификации.

Примечание

Подключение к Oracle Database с использованием внешней аутентификации поддерживается только в режиме python-oracledb Thick. См. раздел Включение режима python-oracledb Thick .

5.3.1 Использование Oracle Wallet для внешней аутентификации
Ниже представлен обзор использования Oracle Wallet. Кошельки следует хранить в безопасном месте. Управлять кошельками можно с помощью Oracle Wallet Manager .

В этом примере кошелек создается для myuserсхемы в каталоге /home/oracle/wallet_dir. mkstoreКоманда доступна из полной установки Oracle Client или Oracle Database. Если администратор баз данных предоставил вам кошелек, перейдите к шагу 3.

Сначала создайте новый кошелек как oracleпользователь:

mkstore -wrl "/home/oracle/wallet_dir" -create
Будет запрошен новый пароль для кошелька.

Создайте запись для имени пользователя и пароля базы данных, которые в настоящее время жёстко заданы в ваших скриптах Python. Используйте любой из способов, представленных ниже. Они запросят пароль кошелька, заданный на первом шаге.

Метод 1. Использование строки Easy Connect :

mkstore -wrl "/home/oracle/wallet_dir" -createCredential dbhost.example.com/orclpdb myuser myuserpw
Метод 2. Использование идентификатора имени соединения :

mkstore -wrl "/home/oracle/wallet_dir" -createCredential mynetalias myuser myuserpw
Ключ псевдонима, mynetaliasследующий сразу за -createCredentialопцией, будет именем подключения, которое будет использоваться в скриптах Python. Если ваше приложение подключается к нескольким пользователям базы данных, вы можете создать запись в кошельке с разными именами подключения для каждого из них.

Вы можете просмотреть вновь созданные учетные данные с помощью:

mkstore -wrl "/home/oracle/wallet_dir" -listCredential
Пропустите этот шаг, если кошелёк был создан с помощью строки Easy Connect. В противном случае добавьте запись в файл tnsnames.ora для имени подключения следующим образом:

mynetalias =
    (DESCRIPTION =
        (ADDRESS = (PROTOCOL = TCP)(HOST = dbhost.example.com)(PORT = 1521))
        (CONNECT_DATA =
            (SERVER = DEDICATED)
            (SERVICE_NAME = orclpdb)
        )
    )
Файл использует описание вашей существующей базы данных и задает псевдоним имени подключения mynetalias, который является идентификатором, используемым при добавлении записи кошелька.

Добавьте следующую запись о местоположении кошелька в файл sqlnet.ora , используя тот, DIRECTORYв котором вы создали кошелек:

WALLET_LOCATION =
    (SOURCE =
        (METHOD = FILE)
        (METHOD_DATA =
            (DIRECTORY = /home/oracle/wallet_dir)
        )
    )
SQLNET.WALLET_OVERRIDE = TRUE
Полные настройки и значения смотрите в документации Oracle.

Убедитесь, что файлы конфигурации находятся в расположении по умолчанию или TNS_ADMIN настроен на каталог, содержащий их. См. раздел Дополнительные файлы конфигурации Oracle Net .

Если кошелек Oracle настроен и доступен для чтения, ваши скрипты смогут подключаться к базе данных Oracle с помощью:

Автономные соединения путем установки externalauthпараметра в значение True в oracledb.connect():

connection = oracledb.connect(externalauth=True, dsn="mynetalias")
Или объединить соединения в пул, установив externalauthпараметр в значение True в oracledb.create_pool(). Кроме того, в режиме python-oracledb Thick необходимо установить homogeneousпараметр в значение False , как показано ниже, поскольку разнородные пулы можно использовать только с внешней аутентификацией:

pool = oracledb.create_pool(externalauth=True, homogeneous=False,
                            dsn="mynetalias")
pool.acquire()
Используемый dsnв oracledb.connect()и oracledb.create_pool()должен совпадать с используемым в кошельке.

После подключения запрос:

SELECT SYS_CONTEXT('USERENV', 'SESSION_USER') FROM DUAL;
покажет:

MYUSER
Примечание

Кошельки также используются для настройки соединений по протоколу TLS (Transport Layer Security). Если вы используете такой кошелек, вам могут потребоваться имя пользователя и пароль базы данных для вызовов oracledb.connect()и oracledb.create_pool()вызовов.

Внешняя аутентификация и прокси-аутентификация

В следующих примерах показана аутентификация внешнего кошелька в сочетании с аутентификацией через прокси . В этих примерах используется конфигурация кошелька, описанная выше, с добавлением разрешения другому пользователю:

ALTER USER mysessionuser GRANT CONNECT THROUGH myuser;
После подключения вы можете проверить, с кем находится пользователь сеанса:

SELECT SYS_CONTEXT('USERENV', 'PROXY_USER'),
       SYS_CONTEXT('USERENV', 'SESSION_USER')
FROM DUAL;
Пример автономного подключения:

# External Authentication with proxy
connection = oracledb.connect(user="[mysessionuser]", dsn="mynetalias")
# PROXY_USER:   MYUSER
# SESSION_USER: MYSESSIONUSER
Вы также можете установить externalauthпараметр на True в автономных соединениях:

# External Authentication with proxy when externalauth is set to True
connection = oracledb.connect(user="[mysessionuser]", dsn="mynetalias",
                              externalauth=True)
# PROXY_USER:   MYUSER
# SESSION_USER: MYSESSIONUSER
Пример пула соединений:

# External Authentication with proxy
pool = oracledb.create_pool(externalauth=True, homogeneous=False,
                            dsn="mynetalias")
pool.acquire(user="[mysessionuser]")
# PROXY_USER:   MYUSER
# SESSION_USER: MYSESSIONUSER
Следующее использование не поддерживается:

pool = oracledb.create_pool(user="[mysessionuser]", externalauth=True,
                            homogeneous=False, dsn="mynetalias")
pool.acquire()
5.3.2 Аутентификация операционной системы
Благодаря аутентификации операционной системы Oracle позволяет выполнять аутентификацию пользователей средствами операционной системы. Ниже представлен обзор реализации аутентификации ОС в Linux.

Войдите в систему на компьютере. В командах, используемых в этом разделе, предполагается, что имя пользователя операционной системы — «oracle».

Войдите в SQL*Plus как пользователь SYSTEM и проверьте значение параметра OS_AUTHENT_PREFIX:

SQL> SHOW PARAMETER os_authent_prefix

NAME                                 TYPE        VALUE
------------------------------------ ----------- ------------------------------
os_authent_prefix                    string      ops$
Создайте пользователя базы данных Oracle, используя имя, os_authent_prefixопределенное на шаге 2, и имя пользователя операционной системы:

CREATE USER ops$oracle IDENTIFIED EXTERNALLY;
GRANT CONNECT, RESOURCE TO ops$oracle;
В Python подключение осуществляется с помощью следующего кода:

connection = oracledb.connect(dsn="mynetalias")
Ваш сеансовый пользователь будет OPS$ORACLE.

Если ваша база данных находится не на том же компьютере, что и Python, вы можете выполнить тестирование, установив параметр конфигурации базы данных remote_os_authent=true. Будьте осторожны, поскольку это небезопасно.

Дополнительные сведения об аутентификации операционной системы см. в Руководстве по безопасности Oracle AI Database .

5.4 Аутентификация на основе токенов
Аутентификация на основе токенов позволяет пользователям подключаться к базе данных с помощью зашифрованного токена аутентификации без необходимости ввода имени пользователя и пароля. Для успешного подключения токен аутентификации должен быть действительным и не просроченным. Пользователи, уже подключившиеся к базе данных, смогут продолжить работу после истечения срока действия своего токена, но не смогут подключиться повторно без получения нового токена.

python-oracledb поддерживает два метода аутентификации: Open Authorization (OAuth 2.0) и Oracle Cloud Infrastructure (OCI) Identity and Access Management (IAM) . Эти методы аутентификации могут использовать облачную аутентификацию с поддержкой Azure SDK или OCI SDK для генерации токенов доступа и подключения к Oracle Database. Кроме того, эти методы могут использовать скрипт Python, содержащий класс для генерации токенов доступа для подключения к Oracle Database.

5.4.1 Аутентификация на основе токенов OCI IAM
Oracle Cloud Infrastructure (OCI) Identity and Access Management (IAM) предоставляет пользователям централизованную систему аутентификации и авторизации в базе данных. Используя этот метод аутентификации, пользователи могут использовать токен доступа к базе данных, выдаваемый OCI IAM, для аутентификации в автономной базе данных Oracle. Драйвер python-oracledb поддерживает как «тонкий», так и «толстый» режимы аутентификации на основе токенов OCI IAM.

При использовании python-oracledb в толстом режиме необходимы клиентские библиотеки Oracle версии 19.14 (или более поздней) или 21.5 (или более поздней).

Отдельные и объединённые в пул соединения можно создавать в режимах python-oracledb «Толстый» и «Тонкий» с использованием аутентификации на основе токенов OCI IAM. Это можно сделать с помощью класса, например, примера класса TokenHandlerIAM , или с помощью плагина аутентификации OCI Cloud Native Authentication Plugin (oci_tokens) python-oracledb . Токены можно указать с помощью параметра подключения, представленного в python-oracledb 1.1. Пользователи более ранних версий python-oracledb также могут использовать строки подключения для аутентификации на основе токенов OCI IAM .

5.4.1.1 Генерация и извлечение токенов OCI IAM
Токены аутентификации можно сгенерировать с помощью плагина oci_tokens python-oracledb .

Альтернативно, токены аутентификации можно сгенерировать путем выполнения команды интерфейса командной строки Oracle Cloud Infrastructure (OCI-CLI).

oci iam db-token get
.oci/db-tokenВ Linux в вашем домашнем каталоге будет создана папка, содержащая файлы токена и закрытого ключа, необходимые для python-oracledb.

Пример создания токена IAM

Здесь в качестве примера мы используем скрипт Python для автоматизации процесса генерации и чтения токенов OCI IAM.

import os

import oracledb

class TokenHandlerIAM:

    def __init__(self,
                 dir_name="dir_name",
                 command="oci iam db-token get"):
        self.dir_name = dir_name
        self.command = command
        self.token = None
        self.private_key = None

    def __call__(self, refresh):
        if refresh:
            if os.system(self.command) != 0:
                raise Exception("token command failed!")
        if self.token is None or refresh:
            self.read_token_info()
        return (self.token, self.private_key)

    def read_token_info(self):
        token_file_name = os.path.join(self.dir_name, "token")
        pkey_file_name = os.path.join(self.dir_name, "oci_db_key.pem")
        with open(token_file_name) as f:
            self.token = f.read().strip()
        with open(pkey_file_name) as f:
            if oracledb.is_thin_mode():
                self.private_key = f.read().strip()
            else:
                lines = [s for s in f.read().strip().split("\n")
                         if s not in ('-----BEGIN PRIVATE KEY-----',
                                      '-----END PRIVATE KEY-----')]
                self.private_key = "".join(lines)
Класс TokenHandlerIAM использует вызываемый объект для генерации и чтения токенов OCI IAM. При первом вызове вызываемого объекта в классе TokenHandlerIAM для создания отдельного соединения или пула параметр refreshимеет значение False , что позволяет вызываемому объекту при необходимости возвращать кэшированный токен. Затем из этого токена извлекается дата истечения срока действия и сравнивается с текущей датой. Если срок действия токена не истёк, он будет использован напрямую. Если срок действия токена истёк, вызываемый объект вызывается второй раз с refresh параметром True .

Определенный здесь класс TokenHandlerIAM используется в примерах, показанных в разделе Создание соединения с токенами доступа OCI IAM .

5.4.1.2 Создание соединения с использованием токенов доступа OCI IAM
Для аутентификации на основе токенов OCI IAM с использованием такого класса, как пример класса TokenHandlerIAM , access_tokenнеобходимо указать параметр подключения. Этот параметр должен представлять собой кортеж из двух элементов (или вызываемый объект, возвращающий кортеж из двух элементов), содержащий токен и закрытый ключ. В приведенных ниже примерах параметру access_tokenприсваивается вызываемый объект.

В примерах, представленных в последующих разделах, класс TokenHandlerIAM используется для генерации токенов OCI IAM для подключения к автономной базе данных Oracle Autonomous Database с использованием протокола Mutual TLS (mTLS). См. раздел Подключение к автономным базам данных Oracle Cloud Autonomous Databases .

Автономные соединения в тонком режиме с использованием токенов OCI IAM

При использовании такого класса, как TokenHandlerIAM, для генерации токенов OCI IAM для подключения к Oracle Autonomous Database в тонком режиме необходимо явно задать access_tokenпараметр connect(), а также любые необходимые параметры config_dir, wallet_locationи wallet_password. Например:

connection = oracledb.connect(
    access_token=TokenHandlerIAM(),
    dsn=mydb_low,
    config_dir="path_to_unzipped_wallet",
    wallet_location="location_of_pem_file",
    wallet_password=wp)
Пулы соединений в тонком режиме с использованием токенов OCI IAM

При использовании такого класса, как TokenHandlerIAM, для генерации токенов OCI IAM для подключения к Oracle Autonomous Database в тонком режиме необходимо явно задать access_tokenпараметр create_pool(), а также любые необходимые параметры , и . config_dirПараметр wallet_locationдолжен быть равен True (значение по умолчанию). Например:wallet_passwordhomogeneous

connection = oracledb.create_pool(
    access_token=TokenHandlerIAM(),
    homogeneous=True, # must always be True for connection pools
    dsn=mydb_low,
    config_dir="path_to_unzipped_wallet",
    wallet_location="location_of_pem_file",
    wallet_password=wp
    min=1, max=5, increment=2)
Обратите внимание, что access_tokenпараметр должен быть установлен как вызываемый объект. Это полезно, когда пул подключений необходимо расширить и создать новые подключения, но срок действия текущего токена истёк. В таком случае вызываемый объект должен возвращать строку, указывающую новый действительный токен доступа.

Автономные соединения в толстом режиме с использованием токенов OCI IAM

При использовании такого класса, как TokenHandlerIAM, для генерации токенов OCI IAM для подключения к автономной базе данных Oracle в режиме Thick необходимо явно задать параметры access_tokenи . Например:externalAuthconnect()

connection = oracledb.connect(
    access_token=TokenHandlerIAM(),
    externalauth=True, # must always be True in Thick mode
    dsn=mydb_low)
Пулы соединений в толстом режиме с использованием токенов OCI IAM

При использовании такого класса, как TokenHandlerIAM, для генерации токенов OCI IAM для подключения к Oracle Autonomous Database в режиме Thick необходимо явно задать параметры access_tokenи . Параметр должен быть равен True (значение по умолчанию). Например:externalauthoracledb.create_pool()homogeneous

pool = oracledb.create_pool(
    access_token=TokenHandlerIAM(),
    externalauth=True, # must always be True in Thick mode
    homogeneous=True,  # must always be True for connection pools
    dsn=mydb_low, min=1, max=5, increment=2)
Обратите внимание, что access_tokenпараметр должен быть установлен как вызываемый объект. Это полезно, когда пул подключений необходимо расширить и создать новые подключения, но срок действия текущего токена истёк. В таком случае вызываемый объект должен возвращать строку, указывающую новый действительный токен доступа.

5.4.1.3. Строки подключения аутентификации на основе токенов OCI IAM
Строка подключения, используемая python-oracledb, может указывать каталог, где находятся файлы токена и закрытого ключа. Этот синтаксис применим и к более старым версиям python-oracledb. Однако рекомендуется использовать параметры подключения, представленные в python-oracledb 1.1. См. раздел « Аутентификация на основе токенов OCI IAM» .

Примечание

Строки подключения для аутентификации на основе токенов OCI IAM поддерживаются только в режиме python-oracledb Thick. См. раздел Включение режима python-oracledb Thick .

Интерфейс командной строки Oracle Cloud Infrastructure (OCI-CLI) можно использовать извне для получения токенов и закрытых ключей из OCI IAM, например, с помощью команды OCI-CLI.oci iam db-token get

TOKEN_AUTHПри использовании синтаксиса строки подключения необходимо задать параметр Oracle Net . Кроме того, PROTOCOLпараметр должен быть tcps и SSL_SERVER_DN_MATCHдолжен быть ON.

Вы можете задать его TOKEN_AUTH=OCI_TOKENв sqlnet.oraфайле. Кроме того, вы можете указать его в дескрипторе подключения , например, при использовании файла tnsnames.ora :

db_alias =
    (DESCRIPTION =
        (ADDRESS=(PROTOCOL=TCPS)(PORT=1522)(HOST=xxx.oraclecloud.com))
        (CONNECT_DATA=(SERVICE_NAME=xxx.adb.oraclecloud.com))
        (SECURITY =
            (SSL_SERVER_CERT_DN="CN=xxx.oraclecloud.com, \
             O=Oracle Corporation,L=Redwood City,ST=California,C=US")
            (TOKEN_AUTH=OCI_TOKEN)
        )
    )
Расположение токена и закрытого ключа по умолчанию совпадает с расположением по умолчанию, куда инструмент OCI-CLI выполняет запись. Например, ~/.oci/db-token/в Linux.

Если файлы токена и закрытого ключа не находятся в расположении по умолчанию, то их каталог необходимо указать с помощью TOKEN_LOCATIONпараметра в файле sqlnet.ora или в дескрипторе подключения , например, при использовании файла tnsnames.ora :

db_alias =
    (DESCRIPTION =
        (ADDRESS=(PROTOCOL=TCPS)(PORT=1522)(HOST=xxx.oraclecloud.com))
        (CONNECT_DATA=(SERVICE_NAME=xxx.adb.oraclecloud.com))
        (SECURITY =
            (SSL_SERVER_CERT_DN="CN=xxx.oraclecloud.com, \
             O=Oracle Corporation,L=Redwood City,ST=California,C=US")
            (TOKEN_AUTH=OCI_TOKEN)
            (TOKEN_LOCATION="/path/to/token/folder")
        )
    )
Значения TOKEN_AUTHи TOKEN_LOCATIONв строке подключения имеют приоритет над sqlnet.oraнастройками.

Пример автономного подключения:

connection = oracledb.connect(dsn=db_alias, externalauth=True)
Пример пула соединений:

pool = oracledb.create_pool(dsn=db_alias, externalauth=True,
                            homogeneous=False, min=1, max=2, increment=1)

connection = pool.acquire()
5.4.1.4. Собственная аутентификация OCI Cloud с помощью плагина oci_tokens
Благодаря облачной аутентификации плагин oci_tokens python-oracledb может автоматически генерировать и обновлять токены OCI IAM при необходимости с поддержкой комплекта средств разработки программного обеспечения (SDK) Oracle Cloud Infrastructure (OCI) .

Плагин oci_tokens можно импортировать следующим образом:

import oracledb.plugins.oci_tokens
Плагин имеет зависимость от пакета Python, который необходимо установить отдельно перед использованием плагина. См. раздел Установка модулей для плагина аутентификации OCI Cloud Native .

Плагин oci_tokensопределяет и регистрирует функцию- перехват параметра , которая использует параметр соединения, extra_auth_paramsпереданный в oracledb.connect(), oracledb.create_pool(), oracledb.connect_async(), или oracledb.create_pool_async(). Используя значения этого параметра, функция-перехват устанавливает access_tokenпараметр объекта ConnectParams в вызываемый объект, который генерирует токен OCI IAM. Затем Python-oracledb получает и использует токен для прозрачного завершения вызовов соединения или создания пула.

Для подключения OCI Cloud Native Authentication и создания пула extra_auth_paramsпараметр должен быть словарем с ключами, как показано в следующей таблице.

Таблица 5.1. Ключи конфигурации аутентификации OCI Cloud Native
Ключ

Описание

Обязательно или необязательно

auth_type

Тип аутентификации. Значение должно быть строкой «ConfigFileAuthentication», «InstancePrincipal», «SecurityToken», «SecurityTokenSimple» или «SimpleAuthentication».


При аутентификации по конфигурационному файлу необходимо указать местоположение конфигурационного файла, содержащего необходимую информацию. По умолчанию этот файл находится по адресу /home/username/.oci/config , если при настройке OCI IAM не указано иное местоположение.


Благодаря аутентификации субъекта-экземпляра вычислительные экземпляры OCI могут быть авторизованы для доступа к сервисам Oracle Cloud, таким как Oracle Autonomous Database. Приложения Python-oracledb, работающие на таком вычислительном экземпляре, автоматически аутентифицируются, устраняя необходимость предоставления учётных данных пользователя базы данных. Этот метод аутентификации будет работать только на вычислительных экземплярах, к которым доступны внутренние сетевые конечные точки. См. раздел «Аутентификация субъекта-экземпляра» .


При аутентификации с помощью токена безопасности или сеансового токена аутентификация выполняется с использованием параметра security_token_file , указанного в файле конфигурации. По умолчанию этот файл находится по адресу /home/username/.oci/config , если при настройке OCI IAM не указано иное расположение. Также необходимо указать профиль , содержащий параметр security_token_file .


При простой аутентификации с помощью токена безопасности или простой аутентификации на основе токена сеанса аутентификация выполняется с использованием параметра security_token_file . Отдельные параметры конфигурации можно задать во время выполнения.


При простой аутентификации отдельные параметры конфигурации могут быть предоставлены во время выполнения.


Дополнительную информацию см. в разделе Методы аутентификации OCI SDK .


Необходимый

user

Идентификатор Oracle Cloud Identifier (OCID) пользователя, вызывающего API. Например, ocid1.user.oc1..<unique_ID> .


Этот параметр можно указать, если значение ключа auth_type— «SimpleAuthentication». Он не требуется, если auth_typeзначение — «SecurityToken» или «SecurityTokenSimple».


Необходимый

key_file

Полный путь и имя файла закрытого ключа.


Этот параметр можно указать, если значение ключа auth_type— «SimpleAuthentication».


Необходимый

fingerprint

Отпечаток пальца, связанный с открытым ключом, который был добавлен к этому пользователю.


Этот параметр можно указать, если значение ключа auth_type— «SimpleAuthentication».


Необходимый

tenancy

Идентификатор OCID вашей аренды. Например, ocid1.tenancy.oc1..<уникальный_ID> .


Этот параметр можно указать, если значение ключа auth_type— «SimpleAuthentication».


Необходимый

region

Регион Oracle Cloud Infrastructure. Например, ap-mumbai-1 .


Этот параметр можно указать, если значение ключа auth_type— «SimpleAuthentication».


Необходимый

profile

Имя профиля конфигурации для загрузки.


Можно создать несколько профилей, каждый с уникальными значениями необходимых параметров. Если не указано иное, используется профиль по умолчанию.


Этот параметр можно указать, если значение ключа auth_typeравно «SimpleAuthentication» или «ConfigFileAuthentication». Если он не указан при использовании «ConfigFileAuthentication», используется значение по умолчанию.


Необходимый

file_location

Расположение файла конфигурации. Значение по умолчанию: ~/.oci/config .


Этот параметр можно указать, если значение ключа auth_typeравно «ConfigFileAuthentication».


Необязательный

scope

Этот параметр идентифицирует все базы данных в облачной аренде аутентифицированного пользователя. Значение по умолчанию — urn:oracle:db::id::* .


Область, которая разрешает доступ ко всем базам данных в отсеке, имеет формат urn:oracle:db::id::<compartment-ocid> , например, urn:oracle:db::id::ocid1.compartment.oc1..xxxxxxxx .


Область, разрешающая доступ к одной базе данных в отсеке, имеет формат urn:oracle:db::id::<compartment-ocid>::<database-ocid> , например, urn:oracle:db::id::ocid1.compartment.oc1..xxxxxx::ocid1.autonomousdatabase.oc1.phx.xxxxxx .


Этот параметр можно указать, если значение ключа auth_typeравно «SimpleAuthentication», «ConfigFileAuthentication» или «InstancePrincipal».


Необязательный

Все ключи и значения, кроме , auth_typeиспользуются вызовами API OCI SDK в плагине. Реализацию плагина можно посмотреть в plugins/oci_tokens.py .

Информацию о параметрах конфигурации, специфичных для OCI, см. в OCI SDK .

В примерах в последующих разделах используется плагин oci_tokens для генерации токенов OCI IAM для подключения к автономной базе данных Oracle Autonomous Database с использованием протокола Mutual TLS (mTLS). См. раздел Подключение к автономным базам данных Oracle Cloud Autonomous Databases .

Автономные соединения в тонком режиме с использованием токенов OCI IAM

При использовании плагина oci_tokens для генерации токенов OCI IAM для подключения к Oracle Autonomous Database в тонком режиме необходимо явно задать extra_auth_paramsпараметр connect(), а также любые необходимые параметры config_dir, wallet_locationи wallet_password. Например:

import oracledb.plugins.oci_tokens

token_based_auth = {                             # OCI specific configuration
    "auth_type": "ConfigFileAuthentication",     # parameters to be set when using
    "profile": <profile>,                        # the oci_tokens plugin with
    "file_location": <filelocation>,             # configuration file authentication
}

connection = oracledb.connect(
    dsn=mydb_low,
    config_dir="path_to_unzipped_wallet",
    wallet_location="location_of_pem_file",
    wallet_password=wp,
    extra_auth_params=token_based_auth)
Пулы соединений в тонком режиме с использованием токенов OCI IAM

При использовании плагина oci_tokens для генерации токенов OCI IAM для подключения к Oracle Autonomous Database в тонком режиме необходимо явно задать extra_auth_paramsпараметр create_pool(), а также любые необходимые параметры , и . config_dirПараметр wallet_locationдолжен быть равен True (значение по умолчанию). Например:wallet_passwordhomogeneous

import oracledb.plugins.oci_tokens

token_based_auth = {
    "auth_type": "SimpleAuthentication", # OCI specific configuration
    "user": <user>,                      # parameters to be set when using
    "key_file": <key_file>,              # the oci_tokens plugin with
    "fingerprint": <fingerprint>,        # simple authentication
    "tenancy": <tenancy>,
    "region": <region>,
    "profile": <profile>
}

connection = oracledb.create_pool(
    dsn=mydb_low,
    config_dir="path_to_unzipped_wallet",
    homogeneous=true,           # must always be True for connection pools
    wallet_location="location_of_pem_file",
    wallet_password=wp,
    extra_auth_params=token_based_auth)
Автономные соединения в толстом режиме с использованием токенов OCI IAM

При использовании плагина oci_tokens для генерации токенов OCI IAM для подключения к автономной базе данных Oracle в режиме Thick необходимо явно задать параметры externalauthи . Например:extra_auth_paramsoracledb.connect()

import oracledb.plugins.oci_tokens

token_based_auth = {
    "auth_type": "SimpleAuthentication", # OCI specific configuration
    "user": <user>,                      # parameters to be set when using
    "key_file": <key_file>,              # the oci_tokens plugin with
    "fingerprint": <fingerprint>,        # simple authentication
    "tenancy": <tenancy>,
    "region": <region>,
    "profile": <profile>
}
connection = oracledb.connect(
    externalauth=True,
    dsn=mydb_low,
    extra_auth_params=token_based_auth)
Пулы соединений в толстом режиме с использованием токенов OCI IAM

При использовании плагина oci_tokens для генерации токенов OCI IAM для подключения к автономной базе данных Oracle в режиме Thick необходимо явно задать параметры extra_auth_paramsи . Параметр должен быть равен True (значение по умолчанию). Например:externalauthcreate_pool()homogeneous

import oracledb.plugins.oci_tokens

token_based_auth = {                             # OCI specific configuration
    "auth_type": "ConfigFileAuthentication",     # parameters to be set when using
    "profile": <profile>,                        # the oci_tokens plugin with
    "file_location": <filelocation>,             # configuration file authentication
}

connection = oracledb.create_pool(
    externalauth=True, # must always be True in Thick mode
    homogeneous=True,  # must always be True for connection pools
    dsn=mydb_low,
    extra_auth_params=token_based_auth)
5.4.2 Аутентификация на основе токенов OAuth 2.0
Пользователями Oracle Cloud Infrastructure (OCI) можно централизованно управлять через службу Microsoft Entra ID (ранее Microsoft Azure Active Directory). Аутентификация на основе токенов Open Authorization (OAuth 2.0) позволяет пользователям проходить аутентификацию в Oracle Database с помощью токенов Entra ID OAuth2. Убедитесь, что у вас есть учетная запись Microsoft Azure, а ваша база данных Oracle зарегистрирована в Microsoft Entra ID. Подробнее см. в разделе « Настройка базы данных Oracle для интеграции с Microsoft Entra ID» . Драйвер python-oracledb поддерживает как «тонкий», так и «толстый» режимы аутентификации на основе токенов OAuth 2.0.

При использовании python-oracledb в толстом режиме необходимы клиентские библиотеки Oracle версии 19.15 (или более поздней) или 21.7 (или более поздней).

Отдельные и объединённые в пул соединения можно создавать в режимах python-oracledb «Толстый» и «Тонкий» с использованием аутентификации на основе токенов OAuth 2.0. Это можно сделать с помощью класса, например, TokenHandlerOAuth , или с помощью подключаемого модуля аутентификации Azure Cloud Native Authentication (azure_tokens) python-oracledb . Токены можно указать с помощью параметра подключения, представленного в python-oracledb 1.1. Пользователи более ранних версий python-oracledb также могут использовать строки подключения аутентификации на основе токенов OAuth 2.0 .

5.4.2.1. Генерация и извлечение токенов OAuth2
Существуют разные способы получения токенов OAuth2 для Entra ID. Для генерации токенов можно использовать плагин azure_tokens из python-oracledb . Некоторые другие способы получения токенов OAuth2 подробно описаны в разделе «Примеры получения токенов OAuth2 для Entra ID» . Вы также можете получить токены OAuth2 для Entra ID с помощью клиентской библиотеки Azure Identity для Python .

Пример создания токена OAuth2

Пример автоматизации процесса генерации и чтения токенов Entra ID OAuth2:

import json
import os

import oracledb
import requests

class TokenHandlerOAuth:

    def __init__(self,
                 file_name="cached_token_file_name",
                 api_key="api_key",
                 client_id="client_id",
                 client_secret="client_secret"):
        self.token = None
        self.file_name = file_name
        self.url = \
            f"https://login.microsoftonline.com/{api_key}/oauth2/v2.0/token"
        self.scope = \
            f"https://oracledevelopment.onmicrosoft.com/{client_id}/.default"
        if os.path.exists(file_name):
            with open(file_name) as f:
                self.token = f.read().strip()
        self.api_key = api_key
        self.client_id = client_id
        self.client_secret = client_secret

    def __call__(self, refresh):
        if self.token is None or refresh:
            post_data = dict(client_id=self.client_id,
                             grant_type="client_credentials",
                             scope=self.scope,
                             client_secret=self.client_secret)
            r = requests.post(url=self.url, data=post_data)
            result = json.loads(r.text)
            self.token = result["access_token"]
            with open(self.file_name, "w") as f:
                f.write(self.token)
        return self.token
Класс TokenHandlerOAuth использует вызываемый объект для генерации и чтения токенов OAuth2. При первом вызове вызываемого объекта в классе TokenHandlerOAuth для создания отдельного соединения или пула параметр refreshпринимает значение False, что позволяет вызываемому объекту при необходимости возвращать кэшированный токен. Затем из этого токена извлекается дата истечения срока действия и сравнивается с текущей датой. Если срок действия токена не истёк, он будет использован напрямую. Если срок действия токена истёк, вызываемый объект вызывается второй раз с refresh параметром True .

Определенный здесь класс TokenHandlerOAuth используется в примерах, показанных в разделе Создание соединения с токенами доступа OAuth2 .

Пример использования команды Curl

Альтернативный способ генерации токенов см. в разделе Использование команды curl .

5.4.2.2 Создание соединения с токенами доступа OAuth2
Для аутентификации на основе токенов OAuth 2.0 с использованием класса, например, класса TokenHandlerOAuth , access_tokenнеобходимо указать параметр подключения. Этот параметр должен быть строкой (или вызываемым объектом, возвращающим строку), указывающей токен Entra ID OAuth2. В приведенных ниже примерах параметру access_tokenприсвоено значение вызываемого объекта.

В примерах, представленных в последующих разделах, класс TokenHandlerOAuth используется для генерации токенов OAuth2 для подключения к автономной базе данных Oracle Autonomous Database с использованием протокола Mutual TLS (mTLS). См. раздел Подключение к автономным базам данных Oracle Cloud Autonomous Databases .

Автономные соединения в тонком режиме с использованием токенов OAuth2

При использовании такого класса, как TokenHandlerOAuth, для генерации токенов OAuth2 для подключения к Oracle Autonomous Database в тонком режиме необходимо явно задать access_token, а также любые необходимые параметры config_dir, wallet_location, и . Например:wallet_passwordconnect()

connection = oracledb.connect(
    access_token=TokenHandlerOAuth(),
    dsn=mydb_low,
    config_dir="path_to_unzipped_wallet",
    wallet_location="location_of_pem_file",
    wallet_password=wp)
Пулы соединений в тонком режиме с использованием токенов OAuth2

При использовании такого класса, как TokenHandlerOAuth, для генерации токенов OAuth2 для подключения к Oracle Autonomous Database в тонком режиме необходимо явно задать access_tokenпараметр create_pool(), а также любые необходимые параметры , и . config_dirПараметр wallet_locationдолжен быть равен True (значение по умолчанию). Например:wallet_passwordhomogeneous

connection = oracledb.create_pool(
    access_token=TokenHandlerOAuth(),
    homogeneous=True, # must always be True for connection pools
    dsn=mydb_low,
    config_dir="path_to_unzipped_wallet",
    wallet_location="location_of_pem_file",
    wallet_password=wp
    min=1, max=5, increment=2)
Обратите внимание, что access_tokenпараметр должен быть установлен как вызываемый объект. Это полезно, когда пул подключений необходимо расширить и создать новые подключения, но срок действия текущего токена истёк. В таком случае вызываемый объект должен возвращать строку, указывающую новый действительный токен Entra ID OAuth2.

Автономные соединения в режиме «Толстый режим» с использованием токенов OAuth2

При использовании такого класса, как TokenHandlerOAuth, для генерации токенов OAuth2 для подключения к Oracle Autonomous Database в режиме Thick необходимо явно задать параметры access_tokenи . Например:externalAuthconnect()

connection = oracledb.connect(
    access_token=TokenHandlerOAuth(),
    externalauth=True, # must always be True in Thick mode
    dsn=mydb_low)
Пулы соединений в толстом режиме с использованием токенов OAuth2

При использовании такого класса, как TokenHandlerOAuth, для генерации токенов OAuth2 для подключения к Oracle Autonomous Database в режиме Thick необходимо явно задать параметры access_tokenи . Параметр должен иметь значение True (что является его значением по умолчанию). Например:externalauthcreate_pool()homogeneous

pool = oracledb.create_pool(
    access_token=TokenHandlerOAuth(),
    externalauth=True, # must always be True in Thick mode
    homogeneous=True,  # must always be True for connection pools
    dsn=mydb_low, min=1, max=5, increment=2)
Обратите внимание, что access_tokenпараметр должен быть установлен как вызываемый объект. Это полезно, когда пул подключений необходимо расширить и создать новые подключения, но срок действия текущего токена истёк. В таком случае вызываемый объект должен возвращать строку, указывающую новый действительный токен Entra ID OAuth2.

5.4.2.3. Строки подключения аутентификации на основе токенов OAuth 2.0
Строка подключения, используемая python-oracledb, может указывать каталог, где находится файл токена. Этот синтаксис применим и к более старым версиям python-oracledb. Однако рекомендуется использовать параметры подключения, представленные в python-oracledb 1.1. См. раздел « Аутентификация на основе токенов OAuth 2.0» .

Примечание

Строки подключения для аутентификации на основе токенов OAuth 2.0 поддерживаются только в режиме python-oracledb Thick. См. раздел Включение режима python-oracledb Thick .

Существуют различные способы получения токенов OAuth2 Entra ID. Некоторые из них подробно описаны в разделе «Примеры получения токенов OAuth2 Entra ID» . Вы также можете получить токены OAuth2 Entra ID с помощью клиентской библиотеки Azure Identity для Python .

Пример использования команды Curl

Здесь в качестве примера мы используем Curl с потоком учетных данных владельца ресурса (ROPC), то есть команда curlиспользуется против API Entra ID для получения токена Entra ID OAuth2:

curl -X POST -H 'Content-Type: application/x-www-form-urlencoded'
https://login.microsoftonline.com/your_tenant_id/oauth2/v2.0/token
-d 'client_id=your_client_id'
-d 'grant_type=client_credentials'
-d 'scope=https://oracledevelopment.onmicrosoft.com/your_client_id/.default'
-d 'client_secret=your_client_secret'
Эта команда генерирует JSON-ответ с типом токена, сроком действия и значениями токена доступа. JSON-ответ необходимо проанализировать, чтобы записать и сохранить в файл только токен доступа. Вы можете сохранить сгенерированное значение access_tokenв файл и указать TOKEN_LOCATIONрасположение файла с токенами. Пример генерации токенов см. в классе TokenHandlerOAuth .

При использовании синтаксиса строки подключения необходимо задать параметры Oracle Net TOKEN_AUTHи . Кроме того, параметр должен быть и должен быть .TOKEN_LOCATIONPROTOCOLtcpsSSL_SERVER_DN_MATCHON

Вы можете установить TOKEN_AUTH=OAUTH. В этом случае местоположение по умолчанию не задано, поэтому необходимо выбрать TOKEN_LOCATIONодно из следующих значений:

Каталог, в этом случае необходимо создать файл с именем token, содержащий значение токена.

Полное имя файла. В этом случае необходимо указать полный путь к файлу, содержащему значение токена.

Вы можете либо установить TOKEN_AUTHего TOKEN_LOCATIONв файле sqlnet.ora , либо указать его внутри дескриптора подключения , например, при использовании файла tnsnames.ora :

db_alias =
    (DESCRIPTION =
        (ADDRESS=(PROTOCOL=TCPS)(PORT=1522)(HOST=xxx.oraclecloud.com))
        (CONNECT_DATA=(SERVICE_NAME=xxx.adb.oraclecloud.com))
        (SECURITY =
            (SSL_SERVER_CERT_DN="CN=xxx.oraclecloud.com, \
             O=Oracle Corporation,L=Redwood City,ST=California,C=US")
            (TOKEN_AUTH=OAUTH)
            (TOKEN_LOCATION="/home/user1/mytokens/oauthtoken")
        )
    )
Значения TOKEN_AUTHи TOKEN_LOCATIONв строке подключения имеют приоритет над sqlnet.oraнастройками.

Пример автономного подключения:

connection = oracledb.connect(dsn=db_alias, externalauth=True)
Пример пула соединений:

pool = oracledb.create_pool(dsn=db_alias, externalauth=True,
                            homogeneous=False, min=1, max=2, increment=1)

connection = pool.acquire()
5.4.2.4. Собственная аутентификация в облаке Azure с помощью плагина azure_tokens
Благодаря облачной аутентификации плагин azure_tokens python-oracledb может автоматически генерировать и обновлять токены OAuth2 при необходимости с поддержкой библиотеки аутентификации Microsoft (MSAL) .

Плагин azure_tokens можно импортировать следующим образом:

import oracledb.plugins.azure_tokens
Плагин имеет зависимость от пакета Python, который необходимо установить отдельно, прежде чем его можно будет использовать. См. раздел Установка модулей для подключаемого модуля аутентификации Azure Cloud Native .

Плагин azure_tokensопределяет и регистрирует функцию- перехват параметра , которая использует параметр подключения, extra_auth_paramsпереданный в oracledb.connect(), oracledb.create_pool(), oracledb.connect_async(), или oracledb.create_pool_async(). Используя значения этого параметра, функция-перехват устанавливает access_tokenпараметр объекта ConnectParams в вызываемый объект, который генерирует токен OAuth2. Затем Python-oracledb получает и использует токен для прозрачного завершения вызовов подключения или создания пула.

Для подключения аутентификации на основе токенов OAuth 2.0 и создания пула extra_auth_paramsпараметр должен быть словарем с ключами, как показано в следующей таблице.

Таблица 5.2. Ключи конфигурации аутентификации Azure Cloud Native
Ключ

Описание

Обязательно или необязательно

auth_type

Тип аутентификации.


Это должна быть строка «AzureServicePrincipal». Этот тип позволяет плагину получать токены доступа субъекта-службы Azure через поток учётных данных клиента.


Необходимый

authority

Этот параметр должен быть задан как строка в формате URI с идентификатором арендатора, например .https://{identity provider instance}/{tenantId}


tenantId — это арендатор каталога, с которым работает приложение, в формате GUID или доменного имени.


Необходимый

client_id

Идентификатор приложения, присвоенный вашему приложению.


Эту информацию можно найти на портале, где была зарегистрирована заявка.


Необходимый

client_credential

Секретный ключ клиента, сгенерированный для вашего приложения на портале регистрации приложений.

Необходимый

scopes

Этот параметр представляет собой значение области действия запроса.


Это должен быть идентификатор ресурса (URI идентификатора приложения) нужного ресурса с суффиксом «.default». Например, https://{uri}/clientID/.default.


Необходимый

Все ключи и значения, кроме , auth_typeиспользуются вызовами API библиотеки аутентификации Microsoft (MSAL) в плагине. Реализацию плагина можно посмотреть в plugins/azure_tokens.py .

Информацию о параметрах конфигурации, специфичных для Azure, см. в MSAL .

В примерах в последующих разделах используется плагин azure_tokens для генерации токенов OAuth2 для подключения к Oracle Autonomous Database с использованием протокола Mutual TLS (mTLS). См. раздел Подключение к Oracle Cloud Autonomous Databases .

Автономные соединения в тонком режиме с использованием токенов OAuth2

При использовании плагина azure_tokens для генерации токенов OAuth2 для подключения к Oracle Autonomous Database в тонком режиме необходимо явно задать extra_auth_paramsпараметр , а также все необходимые параметры config_dir, wallet_location, и . Например:wallet_passwordconnect()

import oracledb.plugins.azure_tokens

token_based_auth = {
    "auth_type": "AzureServicePrincipal", # Azure specific configuration
    "authority": <authority>,             # parameters to be set when using
    "client_id": <client_id>,             # the azure_tokens plugin
    "client_credential": <client_credential>,
    "scopes": <scopes>
}

connection = oracledb.connect(
    dsn=mydb_low,
    config_dir="path_to_unzipped_wallet",
    wallet_location="location_of_pem_file",
    wallet_password=wp,
    extra_auth_params=token_based_auth)
Пулы соединений в тонком режиме с использованием токенов OAuth2

При использовании плагина azure_tokens для генерации токенов OAuth2 для подключения к Oracle Autonomous Database в тонком режиме необходимо явно задать extra_auth_paramsпараметр create_pool(), а также любые необходимые параметры , и . config_dirПараметр wallet_locationдолжен быть равен True (значение по умолчанию). Например:wallet_passwordhomogeneous

import oracledb.plugins.azure_tokens

token_based_auth = {
    "auth_type": "AzureServicePrincipal", # Azure specific configuration
    "authority": <authority>,             # parameters to be set when using
    "client_id": <client_id>,             # the azure_tokens plugin
    "client_credential": <client_credential>,
    "scopes": <scopes>
}

connection = oracledb.create_pool(
    dsn=mydb_low,
    config_dir="path_to_unzipped_wallet",
    homogeneous=true,          # must always be True for connection pools
    wallet_location="location_of_pem_file",
    wallet_password=wp,
    extra_auth_params=token_based_auth)
Автономные соединения в режиме «Толстый режим» с использованием токенов OAuth2

При использовании плагина azure_tokens для генерации токенов OAuth2 для подключения к Oracle Autonomous Database в режиме Thick необходимо явно задать параметры extra_auth_paramsи . Например:externalauthconnect()

import oracledb.plugins.azure_tokens

token_based_auth = {
    "auth_type": "AzureServicePrincipal", # Azure specific configuration
    "authority": <authority>,             # parameters to be set when using
    "client_id": <client_id>,             # the azure_tokens plugin
    "client_credential": <client_credential>,
    "scopes": <scopes>
}

connection = oracledb.connect(
    externalauth=True,  # must always be True in Thick mode
    dsn=mydb_low,
    extra_auth_params=token_based_auth)
Пулы соединений в толстом режиме с использованием токенов OAuth2

При использовании плагина azure_tokens для генерации токенов OAuth2 для подключения к Oracle Autonomous Database в режиме Thick необходимо явно задать параметры extra_auth_paramsи . Параметр должен быть равен True (значение по умолчанию). Например:externalauthcreate_pool()homogeneous

import oracledb.plugins.azure_tokens

token_based_auth = {
    "auth_type": "AzureServicePrincipal", # Azure specific configuration
    "authority": <authority>,             # parameters to be set when using
    "client_id": <client_id>,             # the azure_tokens plugin
    "client_credential": <client_credential>,
    "scopes": <scopes>
}

connection = oracledb.create_pool(
    externalauth=True, # must always be True in Thick mode
    homogeneous=True,  # must always be True for connection pools
    dsn=mydb_low,
    extra_auth_params=token_based_auth)
5.5 Аутентификация субъекта экземпляра
Благодаря аутентификации субъекта-экземпляра вычислительные экземпляры Oracle Cloud Infrastructure (OCI) могут быть авторизованы для доступа к сервисам Oracle Cloud, таким как Oracle Autonomous Database. Приложения Python-oracledb, работающие на таком вычислительном экземпляре, не требуют предоставления учётных данных пользователя базы данных.

Каждый вычислительный экземпляр ведёт себя как отдельный тип принципала управления идентификацией и доступом (IAM), то есть каждый вычислительный экземпляр имеет уникальный идентификатор в виде цифрового сертификата, управляемого OCI. При использовании аутентификации принципала экземпляра вычислительный экземпляр проходит аутентификацию в OCI IAM, используя этот идентификатор, и получает кратковременный токен. Этот токен затем используется для доступа к сервисам Oracle Cloud без хранения и управления какими-либо секретными данными в вашем приложении.

В примере ниже показано, как подключиться к автономной базе данных Oracle Autonomous Database с использованием аутентификации Instance Principal. Для этого используйте плагин oci_tokens из python-oracledb , который предустановлен вместе с oracledbмодулем.

Шаг 1: Создание вычислительного экземпляра OCI

Вычислительный экземпляр OCI — это виртуальная машина, работающая в среде OCI и предоставляющая вычислительные ресурсы для вашего приложения. Этот вычислительный экземпляр будет использоваться для аутентификации доступа к сервисам Oracle Cloud при использовании аутентификации субъекта экземпляра.

Чтобы создать вычислительный экземпляр OCI, ознакомьтесь с инструкциями в разделе Создание экземпляра документации Oracle Cloud Infrastructure.

Дополнительную информацию о вычислительных экземплярах OCI см. в разделе Вызов служб из вычислительного экземпляра .

Шаг 2: Установите OCI CLI на свой вычислительный экземпляр

Интерфейс командной строки OCI (CLI) , который можно использовать самостоятельно или вместе с консолью Oracle Cloud для выполнения задач OCI.

Чтобы установить OCI CLI на вычислительный экземпляр, см. инструкции по установке в разделе Установка CLI документации Oracle Cloud Infrastructure.

Шаг 3: Создайте динамическую группу

Динамическая группа используется для определения правил группировки вычислительных экземпляров, которым требуется доступ.

Чтобы создать динамическую группу с помощью консоли Oracle Cloud, ознакомьтесь с инструкциями в разделе Создание динамической группы в документации Oracle Cloud Infrastructure.

Шаг 4: Создайте политику IAM

Политика IAM используется для предоставления динамической группе разрешения на доступ к необходимым сервисам OCI, таким как автономная база данных Oracle. Если область действия не задана, политика должна применяться к указанному арендатору.

Чтобы создать политику IAM с помощью консоли Oracle Cloud, ознакомьтесь с инструкциями в разделе Создание политики IAM в документации по Oracle Cloud Infrastructure.

Шаг 5: Сопоставьте субъекта экземпляра с пользователем базы данных Oracle

Необходимо сопоставить субъекта экземпляра с пользователем базы данных Oracle. Подробнее см. в разделе Доступ к базе данных с использованием субъекта экземпляра .

Также убедитесь, что в Oracle ADB включена внешняя аутентификация, а параметр Oracle Database IDENTITY_PROVIDER_TYPEустановлен на OCI_IAM . Инструкции см. в разделе Включение IAM-аутентификации в ADB .

Шаг 6: Разверните приложение на вычислительном экземпляре

Чтобы использовать аутентификацию Instance Principal, задайте её extra_auth_paramsпри создании отдельного подключения или пула подключений. Заданная политика IAM должна разрешать доступ в соответствии с указанной областью действия. Сведения о ключах этого extra_auth_paramsпараметра см. в разделе «Ключи конфигурации аутентификации OCI Cloud Native» .

Пример подключения с использованием Instance Principal:

import oracledb
import oracledb.plugins.oci_tokens

token_based_auth = {
  "auth_type": "InstancePrincipal"
}

connection = oracledb.connect(
    dsn=mydb_low,
    extra_auth_params=token_based_auth
)
5.6 Методы аутентификации для поставщиков централизованной конфигурации
Для доступа к централизованному поставщику конфигурации вам может потребоваться предоставить методы аутентификации. В этом разделе подробно описаны методы аутентификации для следующих централизованных поставщиков конфигурации:

Поставщик централизованной конфигурации хранилища объектов OCI

Поставщик централизованной конфигурации приложений Azure

5.6.1 Методы аутентификации поставщика конфигурации хранилища объектов OCI
Для доступа к централизованному поставщику конфигурации OCI Object Storage можно использовать метод аутентификации Oracle Cloud Infrastructure (OCI). Метод аутентификации можно задать в <option>=<value>параметре строки подключения к OCI Object Storage . В зависимости от выбранного метода аутентификации необходимо также задать соответствующие параметры аутентификации в строке подключения.

Вы можете указать один из перечисленных ниже методов аутентификации.

Аутентификация на основе API-ключа

Аутентификация в OCI выполняется с использованием значений, связанных с ключами API. Это метод аутентификации по умолчанию. Обратите внимание, что этот метод используется, когда значение аутентификации не задано или параметру присвоено значение OCI_DEFAULT .

Необязательные параметры аутентификации, которые можно задать для этого метода: OCI_PROFILE , OCI_TENANCY , OCI_USER , OCI_FINGERPRINT , OCI_KEY_FILE и OCI_PASS_PHRASE . Эти параметры аутентификации также можно задать в файле конфигурации аутентификации OCI, который может храниться в расположении по умолчанию ~/.oci/config , ~/.oraclebmc/config или в расположении, указанном переменной среды OCI_CONFIG_FILE . См. раздел Параметры аутентификации для объектного хранилища Oracle Cloud Infrastructure (OCI) .

Аутентификация субъекта экземпляра

Аутентификация в OCI выполняется с использованием учётных данных экземпляра виртуальной машины, работающей в OCI. Чтобы использовать этот метод, установите значение параметра OCI_INSTANCE_PRINCIPAL . Для этого метода нет дополнительных параметров аутентификации, которые можно задать.

Аутентификация принципала ресурса

Аутентификация в OCI выполняется с использованием принципалов ресурсов OCI. Для использования этого метода необходимо установить значение параметра OCI_RESOURCE_PRINCIPAL. Для этого метода нет дополнительных параметров аутентификации, которые можно задать.

Более подробную информацию об этих методах аутентификации см. в разделе Методы аутентификации OCI .

5.6.2 Методы аутентификации поставщика конфигурации приложений Azure
Для доступа к централизованному поставщику конфигурации приложения Azure можно использовать метод аутентификации Microsoft Azure. Метод аутентификации можно задать в параметре <option>=<value>строки подключения приложения Azure . В зависимости от выбранного метода аутентификации необходимо также задать соответствующие параметры аутентификации в строке подключения.

Учетные данные Azure по умолчанию

Аутентификация в Azure App Configuration выполняется как субъект-служба (с использованием секрета клиента или сертификата клиента) или как управляемое удостоверение в зависимости от заданных параметров. Этот метод аутентификации также поддерживает чтение параметров как переменных среды. Это метод аутентификации по умолчанию. Он используется, когда значение аутентификации не задано или параметру присвоено значение AZURE_DEFAULT .

Для этого параметра можно задать следующие необязательные параметры: AZURE_CLIENT_ID , AZURE_CLIENT_SECRET , AZURE_CLIENT_CERTIFICATE_PATH , AZURE_TENANT_ID и AZURE_MANAGED_IDENTITY_CLIENT_ID . Дополнительные сведения об этих параметрах см. в разделе Параметры аутентификации для хранилища конфигураций приложений Azure .

Принципал обслуживания с клиентской тайной

Аутентификация в службе конфигурации приложений Azure выполняется с использованием секретного ключа клиента. Для использования этого метода необходимо установить значение параметра AZURE_SERVICE_PRINCIPAL . Обязательные параметры, которые необходимо задать для этого параметра, включают AZURE_SERVICE_PRINCIPAL , AZURE_CLIENT_ID , AZURE_CLIENT_SECRET и AZURE_TENANT_ID . Дополнительные сведения об этих параметрах см. в статье Параметры аутентификации для хранилища конфигураций приложений Azure .

Принципал обслуживания с сертификатом клиента

Аутентификация в службе конфигурации приложений Azure выполняется с использованием клиентского сертификата. Для использования этого метода необходимо установить значение параметра AZURE_SERVICE_PRINCIPAL . Обязательные параметры, которые необходимо задать для этого параметра: AZURE_SERVICE_PRINCIPAL , AZURE_CLIENT_ID , AZURE_CLIENT_CERTIFICATE_PATH и AZURE_TENANT_ID . Дополнительные сведения об этих параметрах см. в статье Параметры аутентификации для хранилища конфигураций приложений Azure .

Обратите внимание, что метод аутентификации «Служба-принципал с клиентским сертификатом» переопределяет метод аутентификации «Служба-принципал с клиентским секретом».

Управляемая идентификация

Аутентификация в Azure App Configuration выполняется с использованием управляемого удостоверения или управляемых учетных данных удостоверения пользователя. Для использования этого метода необходимо установить значение параметра AZURE_MANAGED_IDENTITY . Если вы хотите использовать назначенное пользователем управляемое удостоверение для аутентификации, необходимо указать обязательный параметр AZURE_MANAGED_IDENTITY_CLIENT_ID . Дополнительные сведения об этих параметрах см. в статье Параметры аутентификации для хранилища конфигураций приложений Azure .
