# Rola testów automatycznych w procesie produkcji oprogramowania
Zestaw testów automatycznych robi znacznie więcej niż tylko pilnowanie, czy przypadkiem nie przekręciliśmy warunków w kodzie.

## Weryfikacja spójności wymagań biznesowych
Mamy sytuację: wpada nowy CR, do kodu pokrytego testami. Robimy zmiany (razem z testami), uruchamiamy testy całości - nasze testy przechodzą, ale coś innego się wysypuje. Testy wyglądają prawidłowo, CR wygląda jakbyśmy zrealizowali prawidłowo, ale dalej jest czerwono. Przyczyna: zmiana, jaką zażyczył sobie klient oznacza zmianę w innych, istniejących, procesach biznesowych - info zwrotne, które możemy odbić do biznesu, żeby wyjaśnił z klientem co oni tak w sumie tu chcą; problem wychodzi zanim klient się na to natknie i wrzuci nam SLA że "zepsuliśmy działającą rzecz" kiedy to ich CR spowodowała rozjechanie się wymagań.

![Typowy biznes](https://www.businessprocessincubator.com/wp-content/uploads/2014/06/adamdeane.files_.wordpress.com201406at-least-a8c3a5a9136298bb33a34668f51b828589746b0d.jpg)

## Regresja
Wprowadzamy zmiany w istniejących funkcjach pod CR lub jako bugfix - jaka jest szansa, że przy systemie wyglądającym jak popularne danie kuchni śródziemnomorskiej (i nie mam na myśli pizzy) nie zepsuje to czegoś pozornie niezwiązanego? Testy pełnią rolę siatki bezpieczeństwa - jeśli nasze zmiany psują coś co już istnieje i powinno działać, dostajemy info zwrotne od razu, bez czekania aż QA w momencie testowania regresji dla nowej wersji wróci do nas z informacją, że tak w sumie to gdzieś w ciągu ostatnich 3 miesięcy udało ci się popsuć tą jedną rzadko używaną funkcję, na której klientowi zależy - powodzenia w szukaniu co i jak to zepsuło.

![Regresja](https://i.pinimg.com/originals/87/1c/35/871c35af560933b8dc4a287b46b2e0a0.jpg)

## Weryfikacja założeń
Testy automatyczne sprawdzają, czy to co chcemy żeby nowy kod robił rzeczywiście jest robione, bez czekania aż zostaną odbite od QA. W praktyce oznacza to, że my pilnujemy żeby kod robił to co naszym zdaniem powinien robić, QA pilnuje czy nasze zrozumienie co kod powinien robić pokrywa się z tym co naobiecywaliśmy klientowi - w efekcie zwrotki z QA powinny w większym stopniu brzmieć "nie o to chodziło", niż "to nie działa". Co w praktyce sprowadza się do mniejszej ilości zwrotek od QA do nas, szczególnie w przypadku CR lub bugfixów - testy pilnują czy działanie systemu jest poprawne, więc jedyne zwrotki mogą dotyczyć tego, że działanie systemu nie jest prawidłowe.

![QA](https://www.testbytes.net/wp-content/uploads/2018/06/1.jpg)

## Skrócenie cyklu wersji
Obecny problem: nowa wersja wydawana jest raz na kilka miesięcy. To oznacza, że dostajemy CR, przygotowujemy zmiany, wrzucamy to do następnej wersji; przed wydaniem wersji QA bierze wszystkie zmiany i robi całościowe testy systemu, odbijając do nas problemy do tematów sprzed 3-4 miesięcy - tematów, w które trzeba się na nowo wgryzać i przypominać sobie jak to działało. Potem wersja trafia do klienta i klient też coś znajduje - w efekcie wraca fix do zmiany sprzed ponad pół roku, gdzie nikt już nie ma pojęcia o co tam w sumie chodziło.

Mając dobre pokrycie systemu testami (melodia przyszłości, ale bardzo realna), jeśli w praktyce będzie oznaczać znacznie mniejszą ilość regresji, możemy realnie myśleć o skróceniu cyklu życia wersji - może nawet docierając do punktu, w którym każdą CR oddajemy jako osobny release - my nie użeramy się ze zwrotkami po miesiącach, klient nie czeka na dodanie trzech przycisków pół roku, QA może się skupić na nowych zmianach i mniej czasu poświęcać na żmudne całościowe sprawdzanie systemu. Wszyscy wygrywają!

![Friday deploy](https://i.redd.it/961lqp4ed9i41.png)

## Skrócenie feedback loop w bieżącej pracy
W obecnej wersji naszego backend, build+testy (założenia: hot cache i zmiana w 1-2 plikach, czyli w trakcie bieżącej pracy) trwa na mojej maszynie około 35s, build+uruchomienie webapi to 65s, plus czas na załadowanie się webapi (kolejne 20s), plus czas na załadowanie frontu i przeklikanie nowej funkcji. Efekt: testy można wpiąć żeby wykonywały się po zbudowaniu, więc praca bieżąca idzie szybciej - napisz co myślisz że zadziała, odpal testy, zobacz wynik, popraw, i tak w kółko aż testy będą zielone. Dopiero wtedy trzeba się przemęczyć z webapi i rzeczywiście przeklikać zmiany - całość idzie szybciej i ma mniej gapienia się na paski postępu niż alternatywa z ręcznym testowaniem.

![Build time](https://www.meme-arsenal.com/memes/f8e3714ed0d4a19b65ebe61006ac9bde.jpg)

## Automatyczny deployment nowych wersji
To już bardzo melodia przyszłości, która dodatkowo wymaga kilku innych rzeczy (CD pipeline, pewny i bezpieczny rollback, automatyczny monitoring, feature toggle itd.), ale którą traktujemy jako potencjalną opcję dla nowych aplikacji greenfield. Scenariusz: wpada CR/bug report, programista robi fix, testy przechodzą, robiony jest commit, PR, zmiana trafia do głównego brancha. CI łapie zmiany, buduje i wystawia wersję, przebija zgłoszenie na testerów, testerzy testują i jak jest ok, odklikują że wersja jest ok. CD przejmuje zagadnienie i wrzuca nową wersję na produkcję, potem na sygnał biznesu op albo tester odklikuje w ustawieniach aplikacji włączenie zmiany dla klienta. W efekcie klient dostaje zmiany szybko, stopniowo, zgłoszenia ewentualnych problemów są na świeżo, a ops mogą skupić się na przygotowaniu i utrzymaniu CD + śledzeniu monitoringu aplikacji, zamiast ręcznie kopiować skrypty i pliki aplikacji między środowiskami.

![Zło](https://blog.empirix.com/wp-content/uploads/2019/04/dev-ops-problem-cat-meme.jpg)
