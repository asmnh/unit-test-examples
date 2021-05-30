# Testy automatyczne
Temat testów automatycznych w dużym legacy systemie z minimalnym pokryciem testami to mały koszmar - na szczęście da się to w miarę sensownie ogarnąć.

Poniżej linki do osobnych dokumentów/stron, z informacją o tym jak ugryźć temat testowania istniejącego, działającego i raczej dużego systemu. Nie od razu da się zrobić wszystko idealnie, dotarcie do przyzwoitego poziomu pokrycia kodu testami zajmie dużo czasu (miesiące, może nawet lata), ale też celem nie jest "100% pokrycia kodu testami" dla samych testów, a ułatwienie przyszłej pracy nad kodem.

## Po co testy
Przede wszystkim, testy automatyczne są po to żeby można było szybciej wprowadzać zmiany, z mniejszym ryzykiem wprowadzenia nowych błędów do istniejących funkcji.

Nawet odrobina testów pełni rolę siatki bezpieczeństwa, pilnującej czy zmiany, jakie dodajemy nie psują oczekiwanych istniejących funkcji systemu. Mając automat pilnujący tego, co już działa i istnieje - nawet jeśli częściowo - co pozwala nam dodać nową funkcję bez zwracania szczególnej uwagi na jej otoczenie, używając testów do sprawdzenia czy w ramach dodawania zmian do istniejącego systemu nie zepsuło się coś, co do tej pory działało.

Dodatkowo, dobre pokrycie testami oszczędza czas potrzebny na ręczną weryfikację poprawności działania całości - jeśli wiemy że jakiś fragment działa i ten fragment ma przyzwoite pokrycie w testach, jeśli testy przechodzą możemy być w miarę pewni (na pewno bardziej pewni niż gdybyśmy próbowali sprawdzić wszystkie możliwe kombinacje ręcznie), że wszystko dalej działa prawidłowo.

Więcej: [Miejsce testów w procesie wprowadzania zmian](rola_testow.md)

## Jak testować
Przede wszystkim: pisać testy. Więcej testów jest lepsze niż mniej, minimum testów jest lepsze niż brak testów. Jak długo testy sprawdzają właściwe rzeczy (założenia i oczekiwania do danej funkcji), spełniają swoją rolę, nawet jeśli tylko częściowo.

Dodawanie testów do istniejącego kodu jest bolesne, trudne i czasochłonne, próba opakowania wszystkiego w kompletny i pełny zestaw testów jest niemożliwa, więc - zamiast tego - dodać tyle testów, ile jesteśmy w stanie, na ile rozsądnie mamy czasu. Nawet jeśli nie pokryjemy wszystkich możliwych ścieżek wykonania, przetestowanie przynajmniej podstawowych (standardowych) ścieżek, plus może specyficznych sytuacji wyjątkowych - jeśli są dla procesu istotne - jest w zupełności wystarczające. Takie testy uzupełni się później, w miarę potrzeb i przy okazji. Więcej: [Testowanie kodu legacy](procedury/legacy.md).

W przypadku, gdy pojawia się zgłoszenie błędu: najlepiej zacząć od napisania testu, który ten błąd weryfikuje. Jeśli rozumiemy przyczynę błędu, taki test wykaże problematyczne zachowanie i nie przejdzie z objawami odpowiadającymi zgłoszeniu; jeśli tak się nie dzieje to znaczy, że błędu nie rozumiemy i trzeba wracać do diagnozy. Po zrobieniu takiego testu zostaje tylko zmiana kodu w taki sposób, żeby ten test zaczął przechodzić (i żaden inny test się nie wyłożył) - błąd poprawiony, plus dodatkowo mamy test na wypadek regresji. Więcej: [Testy automatyczne przy poprawianiu błędów](procedury/bugfix.md).

Dla nowych projektów (ładna nazwa - "greenfield") - nie widzę uzasadnienia, żeby testów *nie* pisać. Jeśli projekt ma być używany więcej niż jednorazowo, jako ad-hoc tymczasowe rozwiązanie, co do którego mamy **gwarancję**, że go nie utrzymujemy (czytaj: deploy i kasujemy kod) - po początkowym etapie wdrożenia trzeba to będzie utrzymywać. I o ile testy mogą sprawić, że przygotowanie takiego projektu zajmie trochę więcej czasu, ten czas szybko się zwróci na zmianach i poprawkach, jakie do projektu będą się pojawiać. Z moich dotychczasowych obserwacji, jeden-dwa cykle zmian (jedna-dwie wersje po początkowej) to czas, po jakim początkowa różnica czasu się wyrównuje, dalej jest już tylko szybciej - o ile dyscyplina pisania testów jest utrzymana. Zachęcam do stosowania się do [zasad TDD](tdd.md) - ten sposób zapewnia dobre pokrycie kodu testami, jednocześnie nie pisząc więcej kodu niż jest to potrzebne. Więcej na temat greenfield: [Testy w nowych projektach](procedury/greenfield.md).

## Co testować
W idealnym świecie - wszystko. W praktyce - co tylko da się przetestować w rozsądnym czasie, biorąc pod uwagę korzyści z testów i czas potrzebny na ich przygotowanie. 

Testy zawsze i bez wyjątku piszemy do:
- biznesowych zachowań systemu (oczekiwana funkcjonalność systemu) - czyli jeśli dany kawałek kodu realizuje jakieś zachowanie biznesowe, dla tego zachowania biznesowego mamy test
- poprawy błędów w systemie, funkcjonalnych i niefunkcjonalnych - każdy poprawiony błąd powinien mieć test, który weryfikuje poprawność patcha (test nie przejdzie na wersji kodu przed poprawką, przejdzie na wersji kodu po poprawce)
- istotnych założeń biznesowych niebędących bezpośrednio częścią funkcjonalności - weryfikacja uprawnień, dostępu do obiektów
- trudnych do zrozumienia fragmentów kodu - jeśli ze sposobu działania kodu nie jest oczywiste, jaki jest cel tego fragmentu, powinny być dodane testy pokazujące użycie (cel istnienia) tego fragmentu
- kompletnie nowe systemy/funkcjonalności/greenfield - gdzie nie ma istniejących fragmentów utrudniających przygotowanie testów

Przypadki, które powinny mieć testy, jeśli jest na to czas i nie wpadają w kategorie powyżej:
- elementy niefunkcjonalne procesów biznesowych - np. zapis historii zmian, prawidłowa kolejność operacji
- obsługa błędów systemu i systemów zewnętrznych - np. reakcja na błąd transakcji, brak połączenia z bazą danych, brak pliku który powinien istnieć
- przypadki szczególne dla procesów biznesowych - np. warunkowane ustawieniami wykonanie operacji, różne sposoby konfiguracji tej samej operacji, obsługa różnych źródeł danych, sytuacje wyjątkowe
- pozornie oczywiste fragmenty kodu - jeśli coś wydaje się być zrozumiałe i "nie do zepsucia", test w takiej sytuacji pełni głównie rolę bezpiecznika na przyszłość

Przypadki, których nie testujemy w ramach zestawu testów automatycznych (mają osobny zestaw testów w ramach testów E2E):
- komunikacja z systemami zewnętrznymi - systemy klienta
- komunikacja międzysystemowa - analogicznie, obejmuje komunikację między niezależnymi systemami, gdzie obie końcówki utrzymujemy my
- operacje na bazie danych (dla kodu legacy, z uwagi na wymaganie rzeczywistej bazy danych w znanym i określonym stanie)
- operacje na systemie plików i strukturach chmurowych (kolejki, blob storage) - analogicznie do bazy danych, konieczność posiadania rzeczywistego środowiska sprawia, że czas wykonania testów nie wyrównuje korzyści z posiadania testów
- długotrwałe testy wydajnościowe i testy obciążeniowe - cały zestaw testów automatycznych powinien się wykonywać szybko, czasochłonne testy wyciągamy na zewnątrz i uruchamiamy osobno, żeby nie przeszkadzały w bieżącej pracy

Do istniejącego kodu dodajemy testy zawsze, gdy:
- poprawiamy błąd w istniejącym kodzie: test ma za zadanie wykazać błąd, który dopiero potem poprawiamy
- rozbudowujemy funkcjonalność: w pierwszej kolejności piszemy test do podstawowego procesu (zabezpieczenie, żeby upewnić się że nie zepsujemy istniejącego procesu), następnie test sprawdzający działanie funkcjonalności po rozbudowie, po zaimplementowaniu zmian oba testy powinny przechodzić
- zmieniamy istniejące zachowanie: tutaj idealna ścieżka to: test sprawdzający obecną funkcjonalność (zielono), modyfikacja testu pod nowe założenia (czerwono), modyfikacja kodu pod nowe założenia (zielono); można pominąć pierwszy krok, szczególnie jeśli zmiana funkcjonalności jest zakresowo duża (kompletnie wywraca założenia dotychczasowego kodu)
- polegamy na specyficznym zachowaniu jednej części kodu we fragmencie, który dodajemy/zmieniamy: test sprawdza czy nasze założenia są prawidłowe, a poźniej chroni przed przypadkowymi efektami ubocznymi wynikającymi z przyszłych zmian, tu warto w komentarzu obok testu dodać że wskazana część systemu polega na tym zachowaniu

Nowy kod piszemy od razu z testami, szczególnie jeśli jest pozbawiony istotnych zależności.

## FAQ

### Skąd mam wziąć czas na testy?
Testy to część czasu potrzebnego na development - dostosuj wyceny tak, żeby czas na pisanie testów się znalazł. Na początku dodawanie testów będzie czasochłonne i bolesne - wszyscy mają tego świadomość - ale w miarę coraz lepszego pokrycia systemu testami i wprawy w pisaniu testów, czas potrzebny na testy będzie systematycznie spadał, a jednocześnie czas na implementację zmian będzie krótszy - mniej sprawdzania i pilnowania czy zmiany nie wpływają na coś innego, proporcjonalnie do pokrycia zachowań systemu testami. Rzeczy, których nie przetestujemy przed oddaniem klientowi wracają do nas jako zgłoszenia bugów i SLA, a za poprawianie bugów klient nie płaci; testy służą temu, żeby początkowo rozwiązywać problemy zanim znajdzie je klient, a docelowo zredukować czas (czyli też koszt) wprowadzania zmian dla wszystkich.

### Nie wystarczy że testuję swoje zmiany ręcznie?
Masz na myśli: testujesz wszystkie potencjalnie powiązane elementy systemu dla każdej swojej zmiany, oraz swoje zmiany za każdym razem kiedy ktoś (ty lub ktoś inny z zespołu) wprowadza swoje zmiany w kodzie? To strasznie dużo klikania. :) Testy uruchamia każdy, w ramach codziennej pracy, na każdej wersji kodu, ciągle - więc jeśli jakiś problem wyjdzie dopiero w przyszłości, przy okazji wprowadzania zmian, automatyczne testy mają większą szansę to wykazać, niż liczenie na wystarczająco kompletne sprawdzenie manualne.

### Testy to nie jest zadanie QA?
Dbanie o jakość systemu to zadanie wszystkich, nie tylko QA. Testerzy mają swoją część do sprawdzenia - weryfikacja czy zmiany są zgodne z wymaganiami, sprawdzenie całościowego działania systemu i dodatkowa weryfikacja zachowań wynikających z niekompletnego pokrycia kodu testami. Automatyczne testy zdejmują z QA wyłapywanie regresji i podstawowych błędów w działaniu systemu - tego typu problemy powinny znacznie częściej wychodzić na etapie developmentu, jako sygnał zwrotny z zestawu testów. Przy dobrym zestawie testów jedyne błędy, jakie QA będzie do nas odbijać powinny wynikać albo z błędnej interpretacji wymagań (zachowanie systemu niezgodne z wymaganiami), albo dotyczyć zakresu nieobjętego testami automatycznymi (np. całościowe procesy dotykające kilku systemów, weryfikacja konfiguracji).

### Co jeśli bardzo nie chcę pisać testów?
Patrz: pytanie numer 2 - alternatywa to ręczne testowanie całego systemu przy każdej zmianie, oraz testowanie wszystkich swoich zmian kiedy ktokolwiek inny wprowadza swoje zmiany. I to wszystko w czasie przewidzianym na pisanie testów automatycznych. Ambitnie. :)

### Ale ja nie umiem w testy!
Pisania testów można się nauczyć. Instrukcja pisania testów dla [kodu legacy](procedury/legacy.md), [bugfixów](procedury/bugfix.md) i [projektów greenfield](procedury/greenfield.md) zawiera opis jak przygotowywać testy w tych przypadkach. Jeśli potrzebujesz bezpiecznej wprawki w pisaniu testów - znajdź fragment systemu, który ma jasne i jednoznaczne zachowanie i nie ma testów, napisz testy weryfikujące poprawność zachowania tego fragmentu i wystaw PR na mnie - przejrzę, dołożę uwagi, podpowiem jak pewne rzeczy można rozwiązać lepiej. Poza tym, zostaje praktyka, pisanie testów to skill, w którym potrzeba trochę wprawy żeby całość szła sprawnie. Dodatkowo, włącz uruchamianie testów w momencie gdy zakończy się build - w ten sposób od razu skorzystasz z informacji zwrotnej z testów pisanych przez inne osoby, jeśli coś się rozjedzie, będzie o tym sygnał zanim zaczniesz testować aplikację ręcznie.

### Co z testami repozytoriów, bazy danych, integracji itd?
To osobny temat - testy E2E działające na osobnym, testowym środowisku nie wchodzą w zakres testów, które trzymamy razem z projektem i każdy uruchamia w ramach bieżącej pracy. Problemy są dwa: potrzeba zapewnienia odpowiedniego środowiska dla takich testów, oraz czas ich wykonania - zestaw testów automatycznych w projekcie generalnie nie powinien się wykonywać dłużej niż 40-60s, wszystko powyżej lepiej wyłączyć na zewnątrz i testować w ramach osobnego procesu. Na chwilę obecną, ustalenie jak testy E2E dokładnie będą wyglądać, jak będą utrzymywane i aktualizowane wisi w powietrzu, jak tylko pojawią się konkrety, pojawi się instrukcja jak z tym pracować.

### Dlaczego dokładasz mi pracy?
Nie dokładam, oszczędzam - w dalszej perspektywie przynajmniej. :) Wprowadzanie nawet drastycznych zmian w dobrze pokrytym testami kodzie jest szybkie - napisz kilka prostych testów sprawdzających nowe funkcje, uruchom (powinno być czerwono w nowych testach), zmień kod tak żeby testy były zielone, profit. Realizowanie nowych zadań jest szybsze, QA rzadziej musi odbijać błędy więc przechodzi przez swoją część testowania szybciej, do nas wraca mniej uwag z QA, klient dostaje nową wersję szybciej i z mniejszymi bólami.
