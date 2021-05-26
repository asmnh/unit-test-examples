# Unit testing - konspekt

1. Po co nam testy
- velocity
- łapanie bugów
- YAGNI - tylko potrzebny kod
- bugfixing + regresja
- pilnowanie zmian + kontraktów
- możliwy refactor
- war story - cache dla workload w WB2.2 - scope test, change, green, refactor
2. TDD - krótkie wyjaśnienie
- najpierw testy, potem kod
- nowy kod krokami
- red-green-refactor loop
- testujemy interfejsy, nie implementacje
3. Unit testy - zasady:
- stałe in/out
- hardkodowane stałe
- "unit" - jeden test = jedna rzecz
- odcięty od świata zewnętrznego
- automatyczny
- jednoznaczny, single-path
4. Co testować
- nowe funkcje
- bugfixy
- zmiany do istniejących funkcji
- NIE: baza danych
- NIE: komunikacja/sieć
- NIE: end-to-end
- TAK: helpery do testów
5. DEMO: TDD dla nowej funkcji - C#, AG: normalizeDriver robi uppercase
6. DEMO: TDD dla bugfixa - JS, gira - ujemny port traktować jak brak portu
7. Jak testować
- arrange/act/assert
- mocki
- setup/teardown
- gdzie i jak zacząć
8. Jak my robimy testy:
- nazwenictwo testów, projekty testowe
  - C# - opcje
- framework
  - MSTest vs xUnit vs nUnit -> MSTest
  - angular: standard
- mocki: Moq
