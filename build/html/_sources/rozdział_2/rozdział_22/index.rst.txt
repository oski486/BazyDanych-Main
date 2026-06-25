Konfiguracja bazy danych PostgreSQL
===================================

Wstęp
-----

Rozdział opisuje konfigurację połączenia z bazą danych **PostgreSQL** – dojrzałym, open-source'owym systemem zarządzania relacyjnymi bazami danych (RDBMS), który wyróżnia się pełną zgodnością ze standardem SQL oraz silnym naciskiem na rozszerzalność i niezawodność.

**Wymagania:**

* dostęp do instancji PostgreSQL,
* zmienne środowiskowe,
* narzędzia projektowe (np. ``psql``, ``pg_isready``).

Podstawowe parametry
--------------------

Każde połączenie z PostgreSQL wymaga:

* hosta i portu (domyślnie ``localhost`` i ``5432``),
* nazwy bazy danych,
* użytkownika i hasła.

Dane wrażliwe (np. hasła) przechowuj w **zmiennych środowiskowych** – to oddziela konfigurację od kodu.
Pamiętaj: zmienne środowiskowe nie są mechanizmem bezpieczeństwa; w produkcji uzupełnij je o dedykowane
zarządzanie sekretami (np. HashiCorp Vault, AWS Secrets Manager).

Lokalizacja i struktura katalogów PostgreSQL
--------------------------------------------

Pliki konfiguracyjne PostgreSQL znajdują się w katalogu danych (``PGDATA``).

**Główne pliki konfiguracyjne:**

* ``postgresql.conf`` – podstawowa konfiguracja serwera (port, pamięć, logi)
* ``pg_hba.conf`` – kontrola dostępu (kto i skąd może się łączyć)
* ``pg_ident.conf`` – mapowanie użytkowników systemowych na role PostgreSQL

**Domyślne lokalizacje:**

.. list-table:: Domyślne lokalizacje
   :header-rows: 1
   :widths: 30 70

   * - System
     - Katalog danych
   * - Linux (RPM/DEB)
     - ``/var/lib/pgsql/data/`` lub ``/var/lib/postgresql/*/main/``
   * - Linux (kompilacja źródłowa)
     - ``/usr/local/pgsql/data/``
   * - Windows
     - ``C:\Program Files\PostgreSQL\<version>\data\``
   * - Docker
     - ``/var/lib/postgresql/data/`` (w kontenerze)

**Zmiana katalogu danych:**

.. code-block:: bash

    # Inicjalizacja nowego katalogu
    initdb -D /ścieżka/do/katalogu

    # Uruchomienie z niestandardowym PGDATA
    pg_ctl -D /ścieżka/do/katalogu start

**Struktura katalogu danych:**

::

    $PGDATA/
    ├── postgresql.conf   # główny plik konfiguracyjny
    ├── pg_hba.conf       # polityka dostępu
    ├── pg_ident.conf     # mapowanie tożsamości
    ├── base/             # rzeczywiste bazy danych
    ├── global/           # tabele systemowe
    ├── log/              # logi serwera
    └── pg_wal/           # Write-Ahead Log (WAL)

**Sprawdzenie aktualnej lokalizacji:**

.. code-block:: sql

    SHOW data_directory;
    SHOW config_file;
    SHOW hba_file;

Środowiska i profile
--------------------

Przełączanie środowisk PostgreSQL (dev/test/prod) realizuj przez:

* zmienne środowiskowe (np. ``PGHOST``, ``PGPORT``, ``PGDATABASE``, ``PGUSER``, ``PGPASSWORD``),
* profile konfiguracyjne,
* osobne pliki ``.env`` dla każdego środowiska.

Automatyzacja
-------------

Zalecenia dla PostgreSQL:

* walidacja konfiguracji przy starcie za pomocą ``pg_isready``,
* skrypty sprawdzające kompletność zmiennych środowiskowych,
* integracja z CI/CD (np. GitHub Actions, GitLab CI).

Dokumentację generuj automatycznie (Sphinx).

Przykłady
---------

**PostgreSQL – zmienne środowiskowe (.env)** ::

    PGHOST=localhost
    PGPORT=5432
    PGDATABASE=aplikacja
    PGUSER=app_user
    PGPASSWORD=${DB_PASSWORD}

**PostgreSQL – URL połączenia** ::

    DATABASE_URL=postgresql://${PGUSER}:${PGPASSWORD}@${PGHOST}:${PGPORT}/${PGDATABASE}

**PostgreSQL – sprawdzenie połączenia** ::

    pg_isready -h ${PGHOST} -p ${PGPORT} -U ${PGUSER} -d ${PGDATABASE}

Najlepsze praktyki
------------------

#. Nie przechowuj sekretów w repozytorium.
#. Stosuj ``.env.example`` zamiast rzeczywistych danych.
#. Używaj standardowych zmiennych środowiskowych PostgreSQL (``PG*``).
#. Waliduj konfigurację przed uruchomieniem (``pg_isready``).
#. W środowiskach produkcyjnych używaj połączeń SSL/TLS (``sslmode=require``).

Podsumowanie
------------

Poprawna konfiguracja połączenia z PostgreSQL zwiększa bezpieczeństwo i przenośność aplikacji między środowiskami. Standardowe zmienne środowiskowe oraz narzędzia takie jak ``pg_isready`` ułatwiają automatyzację i walidację.



Szczegółowe omówienie plików konfiguracyjnych
---------------------------------------------

Po omówieniu podstawowych wymagań środowiskowych można przejść do szczegółowej konfiguracji PostgreSQL.

W kolejnych sekcjach przedstawiono najważniejsze parametry konfiguracyjne dostępne w plikach:

* ``postgresql.conf`` – konfiguracja działania serwera,
* ``pg_hba.conf`` – kontrola dostępu do bazy danych,
* ``pg_ident.conf`` – mapowanie użytkowników systemowych na role PostgreSQL.


postgresql.conf
---------------

Plik ``postgresql.conf`` jest głównym plikiem konfiguracyjnym klastra PostgreSQL. 
Zawiera ustawienia wpływające na działanie serwera, wydajność, bezpieczeństwo, logowanie oraz replikację.

Lokalizację aktywnego pliku można sprawdzić poleceniem:

.. code-block:: sql

    SHOW config_file;

Najważniejsze parametry
~~~~~~~~~~~~~~~~~~~~~~~

Połączenia sieciowe
^^^^^^^^^^^^^^^^^^^

``listen_addresses``
    Określa adresy IP, na których PostgreSQL nasłuchuje połączeń.

    Najczęstsze wartości:

    * ``localhost`` – tylko połączenia lokalne,
    * ``*`` – wszystkie interfejsy sieciowe,
    * lista adresów, np. ``'192.168.1.10,localhost'``.

``port``
    Port TCP używany przez PostgreSQL.

    Domyślna wartość:

    .. code-block:: ini

        port = 5432

``max_connections``
    Maksymalna liczba jednoczesnych połączeń z bazą danych.

    Zbyt wysoka wartość może zwiększać zużycie pamięci.

Pamięć i wydajność
^^^^^^^^^^^^^^^^^^

``shared_buffers``
    Ilość pamięci RAM używanej przez PostgreSQL do buforowania danych.

    Typowe wartości:

    * 128MB – środowiska testowe,
    * 25% pamięci RAM – środowiska produkcyjne.

``work_mem``
    Ilość pamięci wykorzystywanej przez operacje sortowania i zapytania.

    Parametr jest przydzielany dla pojedynczej operacji.

``maintenance_work_mem``
    Pamięć używana podczas operacji administracyjnych, np.:

    * ``VACUUM``,
    * ``CREATE INDEX``,
    * ``ALTER TABLE``.

``effective_cache_size``
    Szacowany rozmiar pamięci podręcznej dostępnej dla PostgreSQL.

    Parametr pomaga optymalizatorowi zapytań wybierać odpowiednie plany wykonania.

Logowanie
^^^^^^^^^^

``logging_collector``
    Włącza zapisywanie logów do plików.

    Dostępne wartości:

    * ``on``,
    * ``off``.

``log_directory``
    Katalog przechowywania logów.

``log_filename``
    Wzorzec nazw plików logów.

    Przykład:

    .. code-block:: ini

        log_filename = 'postgresql-%Y-%m-%d.log'

``log_min_duration_statement``
    Określa minimalny czas wykonania zapytania (w ms), po którym zostanie ono zapisane w logach.

    Przykład:

    .. code-block:: ini

        log_min_duration_statement = 1000

    Wartość ``1000`` oznacza logowanie zapytań trwających dłużej niż 1 sekundę.

Bezpieczeństwo
^^^^^^^^^^^^^^

``password_encryption``
    Algorytm szyfrowania haseł użytkowników.

    Zalecana wartość:

    .. code-block:: ini

        password_encryption = scram-sha-256

``ssl``
    Włącza obsługę połączeń SSL/TLS.

    Dostępne wartości:

    * ``on``,
    * ``off``.

``ssl_cert_file`` / ``ssl_key_file``
    Ścieżki do certyfikatu i klucza prywatnego SSL.

Replikacja i WAL
^^^^^^^^^^^^^^^^

``wal_level``
    Określa poziom szczegółowości Write-Ahead Log (WAL).

    Dostępne wartości:

    * ``minimal``,
    * ``replica``,
    * ``logical``.

``max_wal_size``
    Maksymalny rozmiar plików WAL przed wykonaniem checkpointu.

``archive_mode``
    Włącza archiwizację WAL.

``archive_command``
    Polecenie wykonywane podczas archiwizacji plików WAL.

Przykładowa konfiguracja
~~~~~~~~~~~~~~~~~~~~~~~~

.. code-block:: ini

    listen_addresses = '*'
    port = 5432
    max_connections = 200

    shared_buffers = 1GB
    work_mem = 16MB
    maintenance_work_mem = 256MB

    logging_collector = on
    log_directory = 'log'

    ssl = on
    password_encryption = scram-sha-256

    wal_level = replica
    max_wal_size = 2GB

Weryfikacja konfiguracji
~~~~~~~~~~~~~~~~~~~~~~~~

Po zmianie parametrów konfigurację można przeładować bez restartu serwera:

.. code-block:: bash

    SELECT pg_reload_conf();

Niektóre parametry wymagają pełnego restartu PostgreSQL.

Aktualne wartości parametrów można sprawdzić poleceniem:

.. code-block:: sql

    SHOW shared_buffers;
    SHOW wal_level;
    SHOW max_connections;


pg_hba.conf
-----------

Plik ``pg_hba.conf`` (Host-Based Authentication) odpowiada za kontrolę dostępu do serwera PostgreSQL.

Definiuje:

* kto może połączyć się z bazą danych,
* z jakiego adresu IP,
* do jakiej bazy danych,
* przy użyciu jakiej metody uwierzytelniania.

Lokalizację aktywnego pliku można sprawdzić poleceniem:

.. code-block:: sql

    SHOW hba_file;

Składnia wpisu
~~~~~~~~~~~~~~

Każdy wpis w ``pg_hba.conf`` ma następującą strukturę:

::

    TYPE    DATABASE    USER    ADDRESS    METHOD

Znaczenie pól:

``TYPE``
    Typ połączenia.

    Dostępne wartości:

    * ``local`` – połączenia lokalne (Unix socket),
    * ``host`` – połączenia TCP/IP,
    * ``hostssl`` – połączenia TCP/IP z SSL,
    * ``hostnossl`` – połączenia TCP/IP bez SSL.

``DATABASE``
    Nazwa bazy danych, do której użytkownik może się połączyć.

    Możliwe wartości:

    * konkretna nazwa bazy,
    * ``all`` – wszystkie bazy,
    * ``sameuser`` – baza o nazwie zgodnej z użytkownikiem.

``USER``
    Użytkownik lub rola PostgreSQL.

``ADDRESS``
    Adres IP lub zakres sieci w notacji CIDR.

    Przykłady:

    * ``127.0.0.1/32``,
    * ``192.168.1.0/24``,
    * ``0.0.0.0/0`` – wszystkie adresy.

``METHOD``
    Metoda uwierzytelniania użytkownika.

Najczęściej używane metody uwierzytelniania
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

``trust``
    Zezwala na połączenie bez uwierzytelniania.

    Zalecane wyłącznie w środowiskach testowych.

``reject``
    Odrzuca połączenie.

``md5``
    Uwierzytelnianie przy użyciu hasła MD5.

    Metoda starsza i mniej bezpieczna niż SCRAM.

``scram-sha-256``
    Nowoczesna metoda uwierzytelniania oparta na SCRAM.

    Zalecana dla nowych instalacji PostgreSQL.

``peer``
    Uwierzytelnianie na podstawie użytkownika systemowego.

    Najczęściej używane dla lokalnych połączeń Linux/Unix.

``ident``
    Mapowanie użytkownika systemowego przy użyciu ``pg_ident.conf``.

Przykładowa konfiguracja
~~~~~~~~~~~~~~~~~~~~~~~~

.. code-block:: ini

    # Połączenia lokalne
    local   all             postgres                        peer

    # Połączenia localhost IPv4
    host    all             all         127.0.0.1/32        scram-sha-256

    # Połączenia localhost IPv6
    host    all             all         ::1/128             scram-sha-256

    # Sieć lokalna
    host    aplikacja       app_user    192.168.1.0/24      scram-sha-256

Najlepsze praktyki
~~~~~~~~~~~~~~~~~~

#. Używaj ``scram-sha-256`` zamiast ``md5``.
#. Ograniczaj dostęp do konkretnych adresów IP.
#. Nie używaj ``trust`` w środowiskach produkcyjnych.
#. Stosuj ``hostssl`` dla połączeń zdalnych.
#. Po zmianie konfiguracji wykonuj reload konfiguracji:

.. code-block:: sql

    SELECT pg_reload_conf();


pg_ident.conf
-------------

Plik ``pg_ident.conf`` umożliwia mapowanie użytkowników systemowych na role PostgreSQL.

Najczęściej używany jest razem z metodami uwierzytelniania:

* ``peer``,
* ``ident``.

Lokalizację aktywnego pliku można sprawdzić poleceniem:

.. code-block:: sql

    SHOW ident_file;

Zastosowanie
~~~~~~~~~~~~

Mechanizm mapowania pozwala określić, który użytkownik systemowy może zostać powiązany z konkretną rolą PostgreSQL.

Jest to szczególnie przydatne:

* w środowiskach Linux/Unix,
* przy automatyzacji administracyjnej,
* dla lokalnych połączeń bez użycia haseł.

Składnia wpisu
~~~~~~~~~~~~~~

::

    MAPNAME    SYSTEM-USERNAME    PG-USERNAME

Znaczenie pól:

``MAPNAME``
    Nazwa mapowania wykorzystywana w ``pg_hba.conf``.

``SYSTEM-USERNAME``
    Użytkownik systemu operacyjnego.

``PG-USERNAME``
    Rola PostgreSQL, na którą użytkownik ma zostać zmapowany.

Przykładowa konfiguracja
~~~~~~~~~~~~~~~~~~~~~~~~

.. code-block:: ini

    # MAPNAME    SYSTEM-USERNAME    PG-USERNAME
    admin_map    postgres           postgres
    admin_map    deploy             db_admin
    app_map      appuser            aplikacja

Przykład użycia w ``pg_hba.conf``:

.. code-block:: ini

    local   all     all     peer map=admin_map

W powyższym przykładzie użytkownicy systemowi zdefiniowani w ``admin_map`` mogą logować się jako odpowiadające im role PostgreSQL.

Najlepsze praktyki
~~~~~~~~~~~~~~~~~~

#. Ograniczaj mapowania wyłącznie do wymaganych użytkowników.
#. Nie mapuj użytkowników systemowych na role superuser bez uzasadnienia.
#. Stosuj osobne role administracyjne i aplikacyjne.
#. Dokumentuj wszystkie mapowania użytkowników.
#. Regularnie przeglądaj konfigurację dostępu.



.. admonition:: Opracowanie
   :class: note

   **Autor:** Michał Kraus 
   **Przedmiot:** Bazy danych  
   **Data:** maj 2026

