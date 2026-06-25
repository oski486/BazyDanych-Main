========================================================================
Rozdział 4: Implementacja fizyczna i zasilanie bazy danych
========================================================================

:Autorzy:
    1. Oskar Mąka
    2. Daniel Szokało
    3. Dawid Szokało
:Kierunek: Informatyka Techniczna, Politechnika Wrocławska
:Przedmiot: Bazy Danych / Projekt Laboratoryjny

Wprowadzenie
============
Niniejszy rozdział stanowi czwartą część sprawozdania laboratoryjnego i dokumentuje etap przejścia od abstrakcyjnego modelu logicznego w trzeciej postaci normalnej (3NF) do pełnej implementacji fizycznej struktur relacyjnych. Prace wdrożeniowe zrealizowano z wykorzystaniem dwóch odmiennych architektonicznie silników zarządzania bazami danych: PostgreSQL (wersja serwerowa) oraz SQLite (wersja plikowa). 

Głównym obiektem implementacji jest system transakcyjny dla sklepu internetowego z elektroniką użytkowną. W rozdziale tym opisano proces powołania struktur schematu (Zadanie 1) oraz stworzenie dedykowanego mechanizmu automatycznego zasilania baz danych za pomocą zoptymalizowanego mechanizmu wsadowego (Zadanie 2).

Definicja fizyczna bazy danych (Zadanie 1)
==========================================
Na podstawie modeli logicznych przygotowano kompletne skrypty DDL (Data Definition Language) definiujące tabele, unikalne indeksy, klucze główne oraz powiązania referencyjne. Zgodnie z wymaganiami laboratoryjnymi, definicja fizyczna bazy została pomyślnie wprowadzona i przetestowana w trzech niezależnych środowiskach:

1. **Zdalny internetowy serwer PostgreSQL** – instancja chmurowa wykorzystywana do testowania połączeń rozproszonych.
2. **Lokalny serwer PostgreSQL** – instancja uruchomiona bezpośrednio w środowisku laboratoryjnym na Politechnice Wrocławskiej.
3. **Plikowa baza SQLite** – wdrożona na zdalnym serwerze akademickim za pośrednictwem platformy JupyterHub.

Kod źródłowy skryptów wdrożeniowych oraz pliki konfiguracyjne zostały umieszczone w dedykowanym repozytorium podprojektu. Link do zasobów zamieszczono poniżej:
* **Link do repozytorium kodu:** `https://github.com/oski486/BD_WprowadzanieBD/tree/main`

Skrypt dla środowiska PostgreSQL
--------------------------------
Środowisko PostgreSQL udostępnia zaawansowany aparat deklaratywny, co pozwoliło na ścisłe typowanie kolumn oraz egzekwowanie więzów integralności na poziomie jądra bazy danych. W skrypcie zastosowano typ ``SERIAL`` dla kluczy sztucznych, typ ``NUMERIC(10,2)`` zapobiegający utracie precyzji zmiennoprzecinkowej przy cenach urządzeń, oraz warunki ``CHECK`` sprawdzające logikę biznesową.

Poniższy fragment prezentuje fizyczną definicję kluczowych tabel systemu:

.. code-block:: sql

    -- Skrypt DDL dla środowiska PostgreSQL (Fragment)
    
    CREATE TABLE Kategorie (
        ID_Kategorii SERIAL PRIMARY KEY,
        Nazwa_kategorii VARCHAR(50) NOT NULL
    );

    CREATE TABLE Producenci (
        ID_Producenta SERIAL PRIMARY KEY,
        Nazwa_producenta VARCHAR(100) NOT NULL
    );

    CREATE TABLE Produkty (
        ID_Produktu SERIAL PRIMARY KEY,
        ID_Kategorii INTEGER NOT NULL REFERENCES Kategorie(ID_Kategorii),
        ID_Producenta INTEGER NOT NULL REFERENCES Producenci(ID_Producenta),
        Nazwa VARCHAR(200) NOT NULL,
        Opis TEXT,
        Cena_aktualna NUMERIC(10,2) NOT NULL CHECK (Cena_aktualna > 0),
        Stan_magazynowy INTEGER NOT NULL DEFAULT 0,
        Gwarancja_miesiace SMALLINT CHECK (Gwarancja_miesiace >= 0)
    );

    CREATE TABLE Pozycje_Zamowienia (
        ID_Pozycji SERIAL PRIMARY KEY,
        ID_Zamowienia INTEGER NOT NULL REFERENCES Zamowienia(ID_Zamowienia),
        ID_Produktu INTEGER NOT NULL REFERENCES Produkty(ID_Produktu),
        Ilosc INTEGER NOT NULL CHECK (Ilosc > 0),
        Cena_historyczna NUMERIC(10,2) NOT NULL CHECK (Cena_historyczna > 0)
    );

    CREATE TABLE Opinie (
        ID_Opinii SERIAL PRIMARY KEY,
        ID_Pozycji INTEGER NOT NULL REFERENCES Pozycje_Zamowienia(ID_Pozycji) ON DELETE CASCADE,
        Ocena SMALLINT NOT NULL CHECK (Ocena BETWEEN 1 AND 5),
        Komentarz TEXT
    );

*Komentarz techniczny:* Zastosowanie klauzuli ``REFERENCES`` gwarantuje zachowanie pełnej spójności referencyjnej. Użycie ``ON DELETE CASCADE`` przy tabeli opinii automatycznie czyści oceny powiązane z usuwanymi pozycjami zamówień, chroniąc bazę przed powstaniem rekordów sierocych.

Skrypt dla środowiska SQLite
----------------------------
W silniku SQLite struktura fizyczna została dostosowana do uproszczonego modelu typowania. Ponieważ SQLite nie posiada dedykowanego typu dla wartości finansowych o stałej precyzji ani dla dat, ceny zamapowano jako typ ``REAL``, a daty jako tekst (ISO 8601). Klucze obce zdefiniowano w sposób jawny na poziomie deklaracji tabel.

.. code-block:: sql

    -- Skrypt DDL dla środowiska SQLite (Fragment)
    
    CREATE TABLE Produkty (
        ID_Produktu INTEGER PRIMARY KEY AUTOINCREMENT,
        ID_Kategorii INTEGER NOT NULL,
        ID_Producenta INTEGER NOT NULL,
        Nazwa TEXT NOT NULL,
        Opis TEXT,
        Cena_aktualna REAL NOT NULL CHECK (Cena_aktualna > 0),
        Stan_magazynowy INTEGER NOT NULL DEFAULT 0,
        Gwarancja_miesiace INTEGER,
        FOREIGN KEY(ID_Kategorii) REFERENCES Kategorie(ID_Kategorii),
        FOREIGN KEY(ID_Producenta) REFERENCES Producenci(ID_Producenta)
    );

    CREATE TABLE Pozycje_Zamowienia (
        ID_Pozycji INTEGER PRIMARY KEY AUTOINCREMENT,
        ID_Zamowienia INTEGER NOT NULL,
        ID_Produktu INTEGER NOT NULL,
        Ilosc INTEGER NOT NULL CHECK (Ilosc > 0),
        Cena_historyczna REAL NOT NULL,
        FOREIGN KEY(ID_Zamowienia) REFERENCES Zamowienia(ID_Zamowienia),
        FOREIGN KEY(ID_Produktu) REFERENCES Produkty(ID_Produktu)
    );

*Komentarz techniczny:* W SQLite automatyczna sekwencja klucza realizowana jest poprzez unikalną konstrukcję ``INTEGER PRIMARY KEY AUTOINCREMENT``. Ze względu na to, że silnik domyślnie nie wymusza więzów kluczy obcych, aplikacja kliencka każdorazowo przy otwarciu sesji wykonuje instrukcję ``PRAGMA foreign_keys = ON;``.

Mechanizm zasilania bazy danych (Zadanie 2)
===========================================
Po poprawnym powołaniu struktur fizycznych, bazy należało zasilić danymi demonstracyjnymi w celach weryfikacji poprawności działania indeksów oraz zapytań SQL. Dane wyjściowe zostały rozbudowane programowo i przygotowane w formie plików wsadowych.

Formatowanie danych (Pliki CSV)
-------------------------------
Surowe dane przygotowano w płaskich plikach w formacie CSV (Comma-Separated Values). Reprezentuje to uniwersalne i elastyczne podejście do przechowywania danych niezależnie od docelowego silnika RDBMS. Poniżej przedstawiono fragment wygenerowanego pliku zawierającego katalog produktów:

.. code-block:: text

    ID_Kategorii,ID_Producenta,Nazwa,Opis,Cena_aktualna,Stan_magazynowy,Gwarancja_miesiace
    1,1,"Sprzęt testowy Model 1","Standardowy opis wyposażenia",1540.50,12,24
    1,3,"Sprzęt testowy Model 2","Standardowy opis wyposażenia",4200.00,45,24
    2,2,"Sprzęt testowy Model 3","Standardowy opis wyposażenia",899.99,20,12
    3,1,"Sprzęt testowy Model 4","Standardowy opis wyposażenia",350.00,15,36

Implementacja mechanizmu importu
--------------------------------
Zasilanie bazy zrealizowano przy pomocy automatycznego skryptu napisanego w języku Python. Dobór mechanizmów importu podyktowany był ograniczeniami środowiskowymi oraz wymogami wydajnościowymi:

1. **Dla środowiska PostgreSQL:** Polecenie ``COPY`` w czystym SQL wymaga uprawnień superużytkownika (administratora) i fizycznego dostępu do plików na dysku serwera. W warunkach pracy zdalnej z instancjami chmurowymi oraz laboratoryjnymi uprawnienia te są zablokowane. Z tego powodu zaimplementowano mechanizm wsadowy oparty na technice **Multi-Value Batch INSERT**. Wykorzystano zoptymalizowaną metodę ``execute_values()`` z modułu ``psycopg2.extras`` (lub sparametryzowane zapytanie wielowierszowe). Pakuje ono setki wierszy z pamięci programu w jedno zapytanie, co redukuje narzut komunikacji sieciowej (network round-trips) i drastycznie przyspiesza proces zasilania na standardowych uprawnieniach użytkownika studenckiego.
2. **Dla środowiska SQLite:** Zastosowano metodę ``executemany()`` z wbudowanego pakietu ``sqlite3``. Aby zoptymalizować czas operacji wejścia/wyjścia (I/O), proces ładowania ujęto w **jedną zwartą transakcję** (ręczne wywołanie ``conn.commit()`` na samym końcu). Zapobiega to ciągłemu blokowaniu pliku bazy na dysku przy każdym pojedynczym rekordzie.

Poniżej zaprezentowano produkcyjną architekturę skryptu aplikacyjnego odpowiedzialnego za automatyczny masowy import struktur:

.. code-block:: python

    import sqlite3
    import random
    import os

    def zasil_baze_sqlite():
        print("Rozpoczęto generowanie danych i import wsadowy do SQLite...")
        
        # Przygotowanie bezpiecznych danych w pamięci
        imiona = ['Jan', 'Anna', 'Piotr', 'Katarzyna', 'Michał', 'Zofia', 'Oskar', 'Kamil']
        nazwiska = ['Kowalski', 'Nowak', 'Wiśniewski', 'Wójcik', 'Kowalczyk', 'Kamiński']
        miasta = ['Wrocław', 'Warszawa', 'Kraków', 'Poznań', 'Gdańsk']

        klienci = []
        for i in range(1, 101):
            klienci.append((
                random.choice(imiona), random.choice(nazwiska),
                f"user{i}_{random.randint(100,999)}@student.pwr.edu.pl",
                f"{random.randint(500, 800)}-{random.randint(100, 999)}-{random.randint(100, 999)}",
                random.choice(miasta), "Wybrzeże Wyspiańskiego 27", "50-370"
            ))

        kategorie = [("Laptopy",), ("Smartfony",), ("Podzespoły PC",)]
        producenci = [("Dell",), ("Apple",), ("Lenovo",)]

        produkty = []
        for i in range(1, 51):
            produkty.append((
                random.randint(1, 3), random.randint(1, 3),
                f"Sprzęt testowy Model {i}", "Standardowy opis wyposażenia",
                round(random.uniform(100.0, 7000.0), 2), random.randint(5, 50), 24
            ))

        # Reset pliku bazy w celu uniknięcia konfliktów "table already exists"
        if os.path.exists('sklep_elektronika.db'):
            os.remove('sklep_elektronika.db')

        conn = sqlite3.connect('sklep_elektronika.db')
        cur = conn.cursor()

        # Wczytanie struktury fizycznej DDL
        with open('schema_sqlite.sql', 'r', encoding='utf-8') as f:
            cur.executescript(f.read())

        # Wprowadzenie danych w ramach JEDNEJ transakcji transakcyjnej (Batch Ingestion)
        cur.executemany("INSERT INTO Klienci (Imie, Nazwisko, Email, Telefon, Miasto, Ulica, Kod_Pocztowy) VALUES (?, ?, ?, ?, ?, ?, ?)", klienci)
        cur.executemany("INSERT INTO Kategorie (Nazwa_kategorii) VALUES (?)", kategorie)
        cur.executemany("INSERT INTO Producenci (Nazwa_producenta) VALUES (?)", producenci)
        cur.executemany("INSERT INTO Produkty (ID_Kategorii, ID_Producenta, Nazwa, Opis, Cena_aktualna, Stan_magazynowy, Gwarancja_miesiace) VALUES (?, ?, ?, ?, ?, ?, ?)", produkty)

        conn.commit()
        conn.close()
        print("Zakończono sukcesem! Baza SQLite została pomyślnie zasilona.")

Podsumowanie
============
Niniejszy rozdział podsumowuje kluczowy etap wdrożenia fizycznego relacyjnej bazy danych sklepu komputerowego. Z powodzeniem zrealizowano mapowanie założeń teoretycznych i modeli 3NF na działające instancje systemów DBMS. 

W sekcji poświęconej **Definicji fizycznej bazy danych (Zadanie 1)** pomyślnie zaimplementowano skrypty struktur dla SQLite oraz PostgreSQL, testując je zarówno na serwerach internetowych (JupyterHub, chmura), jak i w lokalnej sieci laboratorium Politechniki Wrocławskiej. Działanie to udowodniło elastyczność struktury i pozwoliło zidentyfikować różnice architektoniczne pomiędzy analizowanymi silnikami bazodanowymi.

W ramach **Mechanizmu zasilania bazy danych (Zadanie 2)** zaimplementowano automatyzację ładowania danych testowych za pomocą skryptów w języku Python. Zastosowanie technik wsadowych (Batch Inserting) oraz optymalizacja operacji transakcyjnych pozwoliły na osiągnięcie wysokiej wydajności zasilania masowego przy jednoczesnym zachowaniu pełnej integralności referencyjnej danych i pracy na standardowych uprawnieniach użytkownika końcowego. Zaprojektowane rozwiązanie stanowi kompletne środowisko, w pełni przygotowane do dalszych testów optymalizacyjnych i analitycznych zapytaniami SQL.
