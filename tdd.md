# Test Driven Development
TDD to podejście do tworzenia kodu z testami, sprowadzające się do założenia: najpierw napisz test sprawdzający to co chcesz zrobić, a dopiero potem dodaj kod realizujący daną funkcjonalność, używając testu do sprawdzenia kiedy ta funkcjonalność działa.

Jako główny cel, TDD stawia sobie usprawnienie wprowadzania zmian w istniejącym systemie i dodawania nowych funkcji - zaczynając od testów bez implementacji, interfejs jest przygotowany z perspektywy tego co dana funkcja ma robić (bez wchodzenia w szczegóły jak to jest realizowane), a określając wymagania dotyczące nowych zmian jako testy jest wyraźny sygnał kiedy implementacja zmian jest kompletna (testy przechodzą).

## Red-green-refactor loop
Główną praktyczną koncepcją stojącą za TDD jest red-green-refactor loop. Sprowadza to fazę dodawania nowych funkcji/zmian w kodzie do trzech kroków:
1. Red: napisz test, który sprawdza zachowanie systemu, które dopiero chcesz zaimplementować. Dodaj absolutne minimum kodu po stronie systemu (puste implementacje, interfejsy itd.) żeby całość się kompilowała. Nowo napisany test nie przechodzi (stąd nazwa "red").
2. Green: zaimplementuj zachowanie w oparciu o to, co jest sprawdzane w testach. Na tym etapie jedyny cel - testy mają przechodzić; bez zwracania uwagi na strukturę, czytelność i ułożenie kodu. Efekt końcowy: wszystkie testy są zielone (stąd "green").
3. Refactor: przeorganizuj napisany w kroku 2. kod tak, aby miał czytelną i dającą się utrzymać strukturę, uporządkuj to, co napisałeś. Testy pilnują, żeby w trakcie refactor nie zepsuć czegoś, co już działało.

Cykl tych trzech kroków powtarza się dla każdej kolejnej zmiany, wymagania lub funkcjonalności. Czasem (jeśli implementujemy funkcjonalność z serią wymagań) można zdecydować się wyłączyć refactor na koniec, natomiast ta faza *nie* jest pomijalna, inaczej nowy kod szybko stanie się nieutrzymywalny pomimo testów, a refactor kiedy nie jesteś na świeżo z kodem zawsze zajmuje więcej czasu i powoduje więcej problemów.

## Kiedy TDD sprawdza się najlepiej
1. Dodajemy nową funkcjonalność. Nowe rzeczy zazwyczaj mamy zdefiniowane jako "co ma się dziać", jakieś user story albo scenariusz użycia - takie wymagania łatwo jest jednoznacznie przełożyć na test, bez konieczności zastanawiania się jeszcze jak dokładnie to zaimplementować.
2. Poprawiamy błąd. W tym wypadku test ma za zadanie odtworzenie błędu w istniejącym systemie, poprawa błędu sprawia że test jest zielony, w tego typu sytuacji refactor czasem nie jest konieczny (poprawa błędu to też dobra okazja żeby spojrzeć na kod istniejącej funkcjonalności i zobaczyć czy da się go trochę poprawić).
3. Dodajemy specyficzne zachowanie do istniejącej funkcjonalności (przypadek szczególny itd). Sytuacja analogiczna do poprawy błędu - dodajemy test sprawdzający brakującą funkcjonalność, zmieniamy kod tak aby test przechodził, robimy refactor jeśli to konieczne.
4. Zmieniany kod ma pokrycie w testach. TDD jest ryzykowne, jeśli nie mamy testów, które - dla fragmentu, który będziemy dotykać - sprawdzają poprawność pozostałych oczekiwanych zachowań, często konieczne jest dopisanie dodatkowych testów do istniejących funkcjonalności jako dodatkowa siatka bezpieczeństwa.

## Typowe problemy z TDD
1. W pierwszej fazie (red) test przechodzi bez zmian w kodzie. To oznacza jedną z dwóch rzeczy: albo dana funkcjonalność już istnieje (rzadziej) i nie trzeba wprowadzać żadnych zmian, albo w testach jest błąd (częściej). Jeśli test prawidłowo odzwierciedla oczekiwane zachowanie, a jesteśmy przekonani, że istniejący kod nie realizuje tego co zamierzamy dodać, prawidłowy test zawsze powinien być czerwony.
2. Mała zmiana wymaga aktualizacji 231231241 testów. Zazwyczaj to oznacza, że testy nie sprawdzają oczekiwanych zachowań, a zamiast tego testują implementację tych zachowań - oznacza to, że istniejące testy są do poprawy.
3. Z kodu po kilku cyklach robi się śmietnik. Pomijasz refactor albo forma testów (znowu: testowanie implementacji, nie zachowań) ogranicza to, jaki refactor możesz zrobić bez dotykania testów - teraz najlepsze co możesz zrobić to posprzątać istniejący kod, plus może przejrzeć i poprawić zbyt szczegółowe testy, na jakość testów i testowanego kodu trzeba będzie zwracać uwagę ciągle.
4. Napisanie testu wymaga zamockowania 789418694 zależności. To zazwyczaj znaczy, że dany klocek robi za dużo, czyli wiesz na czym będzie polegał refactor - na razie zamockuj co musisz i upewnij się że po twoich zmianach wszystko działa, potem możesz ten kod ułożyć inaczej (np. rozbić na mniejsze klocki).
