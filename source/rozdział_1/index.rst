========================
Rozdział 1: Wprowadzenie
========================

:Autor: Oskar Mąka

Niniejszy rozdział otwiera sprawozdanie końcowe z przedmiotu Bazy Danych, realizowanego w semestrze letnim roku akademickiego 2025/2026 w ramach studiów na kierunku informatyka techniczna.

* **Link do repozytorium głównego:** `https://github.com/oski486/BazyDanych-Main`
* **Link do repozytorium tematycznego:** `https://github.com/oski486/BazyDanych-Subject`
* **Link do repozytorium rozdziału 3:** `https://github.com/oski486/BD_PlanBazyDanych_zad1`
* **Link do repozytorium rozdziału 4:** `https://github.com/oski486/BD_WprowadzanieBD/tree/main`
* **Link do repozytorium rozdziału 5:** `https://github.com/oski486/BD_Zapytania`

Wprowadzenie tematyczne
=======================

Dokumentacja ta prezentuje całościowe podejście do inżynierii systemów bazodanowych – począwszy od fundamentów teoretycznych, poprzez planowanie architektury, aż po fizyczne wdrożenie i analityczną weryfikację. W ośmiu podrozdziałach części przeglądowej zgromadzono niezbędną wiedzę z zakresu budowy systemów zarządzania bazami danych (DBMS), rygorystycznych zasad normalizacji oraz standardów i możliwości oferowanych przez język SQL, co stanowi podstawę dla późniejszych decyzji inżynierskich.

Praktyczny wymiar projektu skupia się na stworzeniu autorskiego systemu transakcyjnego dla sklepu internetowego dystrybuującego sprzęt elektroniczny. Analizą objęto kluczowe procesy biznesowe w sektorze e-commerce: zarządzanie kontami klientów, ewidencję i kategoryzację asortymentu, a także cykl życia zamówień, płatności oraz obsługę recenzji produktowych. Etap projektowy zaowocował powstaniem modelu konceptualnego, który następnie przekształcono w zoptymalizowany model logiczny, bezwzględnie spełniający reguły trzeciej postaci normalnej (3NF). Wymagało to zastosowania precyzyjnych rozwiązań architektonicznych, takich jak klucze sztuczne dla relacji wielokrotnych (koszyk zamówieniowy) czy mechanizmy utrwalania cen historycznych w momencie zakupu. Ostatecznie opracowano dwa niezależne modele fizyczne, dopasowane do możliwości i ograniczeń silników PostgreSQL oraz SQLite.

Finałowe części raportu szczegółowo dokumentują proces implementacji struktur w środowiskach docelowych, z uwzględnieniem lokalnych i chmurowych instancji PostgreSQL oraz akademickiej platformy JupyterHub. Istotnym elementem projektu była optymalizacja procesu ładowania danych – w tym celu opracowano skrypty w języku Python, które dzięki technikom wsadowego wprowadzania danych (*batch insert*) i ręcznemu zarządzaniu transakcjami, wydajnie zasilają tabele wygenerowanymi rekordami testowymi. Stabilność i poprawność całego systemu zweryfikowano poprzez implementację złożonych zapytań SQL, opartych na wielokrotnych złączeniach, grupowaniu, podzapytaniach i funkcjach agregujących. Skrypty odpytujące zostały przetestowane równolegle na obu silnikach relacyjnych, a ich kod źródłowy zintegrowano z narzędziem Sphinx w celu automatycznego wygenerowania finalnej dokumentacji technicznej.
