=========================================
Wydajność, skalowanie i replikacja danych
=========================================

+---------+--------------------------+
| Autorzy:| 1. Olaf Chomicki         |
|         | 2. Konrad Machowski      |
|         | 3. Wiktor Wydrzyński     |
+---------+--------------------------+

Wstęp
=====

W erze gwałtownego wzrostu ilości generowanych informacji, stabilne działanie systemów bazodanowych stało się fundamentem nowoczesnych aplikacji. Wydajność, skalowanie oraz replikacja to trzy ściśle powiązane ze sobą filary, które decydują o tym, czy baza danych będzie w stanie obsłużyć rosnącą liczbę użytkowników oraz operacji w jednostce czasu. Zrozumienie różnic oraz synergii między tymi pojęciami jest kluczowe dla każdego architekta oprogramowania [1]_.

Podczas gdy wydajność skupia się na optymalizacji istniejących zasobów w celu jak najszybszego przetwarzania pojedynczych żądań, skalowanie i replikacja odpowiadają na wyzwania związane z ograniczeniami sprzętowymi oraz koniecznością zapewnienia ciągłości działania systemu w przypadku awarii.


Wydajność baz danych
====================

Wydajność (Performance) odnosi się do szybkości, z jaką system bazodanowy reaguje na zapytania i transakcje. Na poziomie pojedynczej instancji kluczowe znaczenie ma optymalizacja kodu SQL, poprawne projektowanie indeksów oraz unikanie kosztownych operacji, takich jak pełne skanowanie tabel (Full Table Scan). Optymalizator bazy danych stara się dobrać najlepszą ścieżkę wykonania zapytania, jednak jego skuteczność zależy od aktualności statystyk oraz struktury danych [2]_.

Po stronie sprzętowej wąskimi gardłami najczęściej stają się operacje wejścia/wyjścia (I/O) dysków, dostępna pamięć RAM (służąca jako bufor dla często odczytywanych stron danych) oraz moc procesora. Wyczerpanie tych zasobów bezpośrednio przekłada się na wzrost opóźnień (latency) i spadek przepustowości (throughput).


Skalowanie: Pionowe vs. Poziome
===============================

Gdy optymalizacja zapytań przestaje wystarczać, system musi zostać poddany skalowaniu. Skalowanie pionowe (Scaling Up) polega na zwiększaniu zasobów pojedynczej maszyny – dodaniu szybszych procesorów, większej ilości pamięci RAM lub wydajniejszych dysków SSD. Jest to najprostsze podejście, niepoprawiające jednak architektury aplikacji, ale ma swoje fizyczne i ekonomiczne granice (prawo malejących przychodów oraz limit techniczny serwera) [3]_.

Skalowanie poziome (Scaling Out) polega na dodawaniu kolejnych maszyn do klastra. W kontekście baz danych jest to zadanie znacznie trudniejsze niż w przypadku bezstanowych aplikacji webowych, ponieważ wymaga mechanizmów dystrybucji danych. Jedną z głównych metod skalowania poziomego zapisu jest sharding, czyli fizyczny podział bazy na niezależne węzły.


Replikacja danych i wysoka dostępność
=====================================

Replikacja to proces kopiowania danych z jednego serwera bazy danych (węzła głównego/Primary) na jeden lub więcej serwerów pomocniczych (węzłów potomnych/Replica). Głównym celem replikacji jest zapewnienie wysokiej dostępności (High Availability) – w razie awarii serwera głównego, jedna z replik może przejąć jego funkcję, minimalizując czas przestoju systemu [4]_.

Dodatkowo replikacja pozwala na skalowanie operacji odczytu (Read Scalability). Aplikacja może kierować zapytania modyfikujące dane (INSERT, UPDATE) do węzła Primary, natomiast ciężkie zapytania raportowe i odczyty rozpraszać na repliki. Replikacja może odbywać się synchronicznie (gwarantuje spójność danych, ale zwalnia zapisy) lub asynchronicznie (szybsza, ale niesie ryzyko utraty ostatnich transakcji w przypadku nagłej awarii).


Wyzwania i kompromisy architektoniczne
======================================

Budowanie systemów wysoce skalowalnych i zreplikowanych wiąże się z koniecznością akceptacji kompromisów, co formalnie opisuje twierdzenie CAP (Consistency, Availability, Partition Tolerance). Mówi ono, że w rozproszonym systemie komputerowym można jednocześnie zapewnić tylko dwie z trzech wymienionych cech [5]_. 

W praktyce systemy bazodanowe często muszą wybierać pomiędzy silną spójnością danych (wszyscy użytkownicy widzą dokładnie to samo w tym samym momencie) a wysoką dostępnością i niskimi opóźnieniami, co doprowadziło do popularyzacji modeli spójności ostatecznej (Eventual Consistency) w bazach typu NoSQL.


Podsumowanie
============

Wydajność, skalowanie i replikacja nie są niezależnymi wyspami – to naczynia połączone. Dobrze zoptymalizowana pod kątem wydajności baza danych odwleka w czasie moment, w którym konieczne stanie się kosztowne skalowanie. Z kolei właściwie wdrożona replikacja nie tylko zabezpiecza system przed utratą danych, ale stanowi naturalny krok w kierunku poziomego skalowania odczytów. Architektura systemu musi być stale dostosowywana do dynamicznie zmieniających się wymagań biznesowych oraz skali operacji.

.. [1] "Projektowanie systemów rozproszonych" – Podstawy skalowalności baz danych.
.. [2] "Optymalizacja zapytań i indeksowanie w relacyjnych bazach danych".
.. [3] Analiza kosztów i barier technologicznych w skalowaniu pionowym.
.. [4] Mechanizmy replikacji synchronicznej i asynchronicznej w systemach High Availability.
.. [5] Twierdzenie CAP a praktyczne implementacje nowoczesnych magazynów danych.
