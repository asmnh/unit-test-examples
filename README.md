# Unit Testing
Unit testy to kod, którego jedynym zadaniem jest sprawdzanie, czy nasz kod robi to, co chcemy żeby robił.

Docelowo: TDD, 80%+ test coverage dla wszystkiego, testy integracyjne i end-to-end dla każdego buildu.

Soon: próby z TDD, przyzwoity test coverage dla nowych i zmienianych kawałków kodu, początki testów integracyjnych.

Teraz: mamy jakieś testy, chyba. :)

## Po co pisać testy
1. Przyspiesza pisanie kodu. Kodu będzie więcej (to zupełnie normalne, że testy zajmują więcej, niż testowany kod), ale sprawdzanie poprawności tego co napisaliśmy, wprowadzanie zmian i debugowanie jest znacznie szybsze. Kod testów jest zawsze prosty, oczywisty i jednoznaczny - pisze się to szybko.
2. Mniej bugów, mniej wraca od testerów. Testerzy mogą skupić się na sprawdzaniu ręcznym tego, czego nie da się sprawdzić automatem, a my dostajemy mniej zwrotek bo jakiś drobiazg się nie spina.
3. Nie piszemy niepotrzebnego kodu. Przy dobrych testach, wszystkie ścieżki są sprawdzone, więc pisząc kod pod testy mamy tylko tyle, ile jest potrzebne.
4. Szukanie i poprawianie bugów. Każdy zgłoszony bug dostaje test, który go sprawdza - test się wykłada na obecnym kodzie - poprawiasz kod - test przechodzi - kod leci do repo; jak coś w przyszłości się popsuje, test się wywali i od razu wiadomo co się dzieje.
5. Wiemy od razu, kiedy zmiany w istniejącym kodzie psują coś co działa. Jeśli zmiana psuje test - test się wykłada, trzeba albo poprawić kod, albo zmienić założenia (zmienić test).
6. Refactoring - z kodem mającym dobre pokrycie w testach można zrobić cokolwiek i od razu będzie wiadomo czy po refactor wszystko działa, czy tylko zdaje się działać.
7. Testy są dokumentacją funkcjonalną kodu - w sposób jednoznaczny, weryfikowalny i wykonywalny opisują *co* nasz kod robi, bez wchodzenia zbyt głęboko w to *jak* to jest realizowane; dobrze napisane testy są jednocześnie przykładem użycia danego fragmentu kodu, oraz opisem danej funkcji.

## Test Driven Development
Brzmi mądrze, ale zasada jest prosta - najpierw test(y), potem kod.

Testy: co kod, który będziemy pisać ma robić (jaki ma być efekt). Implementacja: jak to ma się dziać.

Siadając do kawałka kodu (nowa funkcjonalność, bugfix) wiemy co chcemy osiągnąć, niekoniecznie musimy wiedzieć jak to osiągnąć.

Pisząc testy przed kodem, mamy jasny sygnał kiedy skończyliśmy - nie ma ryzyka przekombinowania i robienia rzeczy niepotrzebnych, których nie będzie się dało sprawdzić ręcznie, a których ktoś potem może użyć w innym miejscu (zakładając, że działają).

### Przyrostowe pisanie kodu
TDD ładnie wpisuje się w przyrostowe pisanie kodu - napisz test, napisz minimum kodu, który go realizuje, napisz kolejny test, zmień/dodaj kod żeby nowy test przechodził, napisz jeszcze jeden test... itd.

Nie ma sytuacji, gdzie piszemy ścianę kodu, a dopiero potem to sprawdzamy mając nadzieję, że o wszystkim pamiętaliśmy, nic nie przeoczyliśmy i wszystko uwzględnimy przy testowaniu; dodatkowo drobne proste błędy (przeoczenia, odwrócone warunki logiczne, brak lub niepoprawne parametry itd) wychodzą o wiele szybciej i o wiele łatwiej. Ten sam argument co przy testowaniu ręcznym dotyczy pisania testów po napisaniu kodu - zamiast sprawdzać założenia, możemy sprawdzać implementację, a wtedy wiemy tylko tyle, że to co napisaliśmy zachowuje się jak napisaliśmy, a nie czy to co napisaliśmy robi to co ma robić.

### Red-green-refactor loop
TDD sprowadza się do trzech kroków:

1. Napisz test, który będzie weryfikował oczekiwane zachowanie danej metody/obiektu/grupy metod. Uruchomiony, taki test powinien nie przejść (red) - jeśli przechodzi, to znaczy że test ma błąd, skoro nowego zachowania jeszcze nie zaimplementowaliśmy.
2. Napisz kod, który sprawia, że test przechodzi. Czyli: zaimplementuj zachowanie/zmianę. Jedyne oczekiwanie: testy mają przechodzić (nie robimy więcej niż to, czego oczekują testy). Implementacja jest skończona, kiedy test jest zielony.
3. Jeśli kod, który wyszedł wygląda jak koszmar, refactor - wyciąganie fragmentów do metod prywatnych, reuse, przeniesienie stałych inline do stałych w jakimś spójnym miejscu.

Red-green-refactor zapewnia, że (prawie) całość pisanego kodu będzie miała pokrycie w testach, więc jakakolwiek zmiana w testowanym kodzie, która psuje założenia/kontrakt, da od razu sygnał przy uruchomieniu testów.

### Testujemy interfejsy, nie implementacje
Kod testowy powinien zawsze być pisany pod interfejs - czyli w ramach testu określamy jakie mamy wejście (stan systemu i parametry), oraz jakiego efektu oczekujemy po wykonaniu testu. Nie testujemy implementacji - nie piszemy testów sprawdzających, czy nasza implementacja się wykonuje (to jest bez sensu i prowadzi do tego, że mamy test typu "czy implementacja się zmieniła"), zamiast tego układamy testy tak, żeby sprawdzały czy w systemie dzieje się to, co ma się dziać.

Przykład złego testu (sprawdza implementację):

```csharp
[TestMethod]
public void UpsertBulbulator_UpdateIfBulbulatorExists()
{
    bulbulatorRepositoryMock.HasValue(new Bulbulator(){ Id = 335, FooBar = true });
    
    var subject = new BulbulatorLogic(bulbulatorRepository);
    subject.UpsertBulbulator(new Bulbulator(){ Id = 335, FooBar = false });

    // sprawdza czy aktualizowany w repo obiekt to ten sam, który został przekazany w parametrze
    // wysypie się, jeśli metoda będzie pobierać obiekt z repo i aktualizować w nim pola
    bulbulatorRepository.Verify(repo => repo.Update(newBulbulator), Times.Once);
}
```

```csharp
[TestMethod]
// poprawiona nazwa metody testowej - sprawdzamy aktualizację tylko jednego pola
public void UpsertBulbulator_UpdateFooBarIfBulbulatorExists()
{
    bulbulatorRepositoryMock.HasValue(new Bulbulator(){ Id = 335, FooBar = true });
    
    var subject = new BulbulatorLogic(bulbulatorRepository);
    subject.UpsertBulbulator(new Bulbulator(){ Id = 335, FooBar = false });

    // lepiej - nie ma znaczenia czy obiekt jest ten sam czy nowy, sprawdzamy oczekiwane zachowanie
    bulbulatorRepository.Verify(repo => repo.Update(It.Is<Bulbulator>(b => b.Id == 335 && b.FooBar == false)), Times.Once);
}
```

W tej kwestii TDD nam sporo pomaga - możemy zacząć pisanie testu od przygotowania interfejsu (bez parametrów, z `throw new NotImplementedException`/`throw new Error` jako implementacją, lub wręcz bez implementacji), napisać test opisujący pożądaną funkcjonalność (1-2 testy, dla najbardziej oczywistych warunków), uruchomić testy żeby upewnić się że test działa, kompiluje się i - jako że nie mamy jeszcze implementacji - wywala w prawidłowy sposób; a dopiero potem zaimplementować tyle kodu, żeby te testy przechodziły. W ten sposób testy zawsze są pisane pod interfejs (czyli pod używającego tej metody/klasy/funkcji/modułu), a nie pod sam testowany fragment.

## Czym jest unit test
Testów jest kilka rodzajów, unit testy charakteryzują się pewnymi specyficznymi cechami, o których dobrze pamiętać zastanawiając się czy coś powinno być testem i jak taki test powinien wyglądać.

### Unit test ma jasno określone, stałe wejście i stałe wyjście.
Nie używamy `Random`, nie używamy `DateTime.Now`, nie polegamy na systemowym locale, strefie czasowej, języku, fazie Księżyca, zawartości bazy danych, aktualnym stanie systemu czy okresie godowym świetlików. Dotyczy to też zmiennych statycznych (jeśli jakieś są, out - jeśli się nie da: komentarz przy zmiennej, `find all references`, `lock` na każde użycie i założyć `lock` w teście, gdzie nadpisujemy wartość zmiennej przed i przywracamy ją po), czy stanu samego runtime (locale dla bieżącego wątku, obecna strefa czasowa). Szczególna uwaga na locale - od czasu do czasu puszczam nasze testy na systemie "nie z tej Ziemi" (arabskie locale, język koreański, z układem tekstu RTL, losowa strefa czasowa) żeby upewnić się że wszystko jest ok, ale nie polegałbym na tym za bardzo. Sporą część wychwycimy mając część runnerów/osób z angielskim, a część z polskim locale w Windows.

### Stałe wejście i wyjście oznacza stałe 
Nie odwołujemy się do stałych w kodzie, nie pobieramy stałych z bazy; jeśli oczekujemy że funkcja zmieni status obiektu na status o ID=8, wpisujemy `8` do testu - jeśli zmienią nam się ID statusów, to psujący się test powinien dać nam o tym info. To samo dotyczy enumów - jeśli enum ma zdefiniowane wartości liczbowe, test powinien mieć `(MyEnum)3` zamiast `MyEnum.ThirdValue`; jeśli przekazujemy enum jako string - `Enum.Parse<MyEnum>("FunValue")` zamiast `MyEnum.FunValue`. Dla enumów, które są używane *tylko* jako enum (nie są parsowane/serializowane i nie mają przypisanych wartości liczbowych) używanie enumów jest ok (a nawet wskazane - tu jedynym identyfikatorem jest identyfikator enuma).

### Jeden unit test sprawdza jedno zachowanie jednej funkcjonalności 
Zazwyczaj oznacza to "jeden `assert` na test", w niektórych sytuacjach kilka asercji jest ok o ile sprawdzają one logicznie jedną rzecz - np. porównujemy wartości pól zwróconego obiektu z oczekiwanymi. Jeśli mamy do sprawdzenia kilka funkcjonalności lub kilka wariantów funkcjonalności - to są osobne testy, które będą się różniły tylko przekazanym parametrem i asercją, to jest ok, Ctrl+C + Ctrl+V pomagają. Testowanie dla kilku różnych wartości jest ok tylko, jeśli te wartości są logicznie tym samym i dotyczą dokładnie tego samego testowanego przypadku - np. dla "funkcja wstawia wartość domyślną jeśli nie dostanie wartości tekstowej" możemy zrobić jeden test (w C# użyć `[DataTestMethod]` i `[DataRow]`) gdzie identyczną logikę sprawdzamy dla `null`, `""`, `" "` i `"\t"` - gdzie null, pusty string i string mający tylko białe znaki oznaczają "brak wartości".

### Test nie zależy od świata zewnętrznego 
Po części wspomniane w [Jasno określone wejście i wyjście](#unit_test_ma_jasno_określone_stałe_wejście_i_wyjście). Uwaga na zmienne statyczne (statyczny modyfikowalny stan obiektu) i współdzielony dostęp do zasobów; każdy unit test powinien być kompletnie niezależny od innych testów. VS wykonuje testy równolegle i w różnej kolejności, więc testy z takimi zależnościami będą (nie "mogą", tylko "będą" - mamy takie "unit" testy w kilku miejscach przy raportach, które działają odpalone pojedynczo a kładą się odpalone wszystkie naraz - ServiceStack nie lubi wielokrotnej inicjalizacji). Unit test ma działać tak samo jak zawsze, uruchomiony w momencie zmiany czasu, dodania sekundy przestępnej, w przestrzeni kosmicznej, lecąc z prędkością bliską prędkości światła, na klingońskim Linuksie odpalony w Mono.

Zależność od systemu plików jest tematem dyskusyjnym - jeśli bardzo potrzebujemy użyć systemu plików, test powinien załadować odpowiedni plik w miejsce skąd może go odczytać (.NET ma do tego odpowiedni atrybut, JS nie powininen grzebać w plikach) i dostać odpowiednią ścieżkę; moja preferencja to wrzucić plik bezpośrednio do testu jako stały string (dla binarnych - Base64, dla tekstowych albo escaped string, albo też Base64 + tekst w komentarzu żeby nie trzeba było dekodować przy sprawdzaniu co tam siedzi).

### Test nie wymaga ingerencji użytkownika
Unit testy są 100% automatyczne - zero wejścia od użytkownika, test uruchamia się i robi wszystko sam. Użytkownik to dla testu świat zewnętrzny, więc zasada jest ta sama - nie wymagamy żeby ktoś coś przerzucił, kliknął itd.

### Test jest jednoznaczny
Test ma mieć jedną i tylko jedną ścieżkę wykonania, która albo przejdzie, albo zakończy się błędem. Jedyne instrukcje sterujące, jakie są ok, to:
- `foreach` i odpowiedniki jeśli sprawdzamy jakiś warunek dla kolekcji; `.All`, `.Any` itd. to odpowiedniki `foreach`
- `if`, tylko i wyłącznie jeśli konstrukcja to `if(warunek){ Assert.Fail("Powód"); }` - szczególnie jeśli warunek jaki sprawdzamy jest długi

Jeśli potrzebujemy sprawdzić kilka różnych ścieżek lub wariantów - to są osobne testy.

## Co testować

Nie wszystko łatwo przetestować, nie wszystko da się przetestować, nie wszystkie testy powinny być unit testami. Lista tego co powinno być objęte testami, a co nie; w razie wątpliwości zasady co jest unit testem powyżej to dobry punkt odniesienia, when in doubt - test.

### Nowe funkcje - tak
Nowe funkcje systemu zaczynamy od napisania testu, który sprawdza działanie tej funkcji, dopiero potem dodajemy daną funkcję. Przy TDD dużo sensu ma pisanie kodu top-down - najpierw dodajemy sposób wywołania danego kodu (endpoint w serwisie, metoda podpięta pod akcję w UI itd.), piszemy do tego test, a pod test piszemy funkcjonalność, na bieżąco mockując/stubując potrzebne nowe zależności (`throw new NotImplementedException()` robi za świetne `// TODO`). Jak dotrzemy do samego dołu - albo nie ma nic do testowania, albo odbijamy się od bazy/zewnętrznego systemu, zostajemy na mockach, nowa funkcja gotowa.

### Bugfixy - tak
Wpada zgłoszenie o błędzie, krok pierwszy: napisz test, któwy wywołuje ten błąd. Skoro wiemy, że w kodzie/działaniu jest błąd, test powinien nie przejść - jeśli przechodzi, to znaczy, że błędu jeszcze nie znaleźliśmy i w sumie nie mamy pojęcia, co tak naprawdę mamy naprawić. :) Skoro mamy test, który "udowadnia" nam istnienie błędu w sposób powtarzalny, krok numer 2 to poprawa błędu - jak po poprawce wszystkie testy przechodzą, *trud nasz skończony*, błąd rozwiązany, commit, push, PR.

Jeśli wysypują się inne testy, to znaczy że nasz bugfix popsuł coś w innym miejscu i może powinniśmy się temu przyjrzeć. Jeśli wysypują się inne testy i uważamy że powinny być zmienione - to nie jest bugfix do nieprzetestowanego kodu, tylko zmiana oczekiwanego zachowania, pytanie czemu to nie jest CR i jakie w sumie zachowanie powinno być - testy w takiej sytuacji to sygnał że trzeba sytuację wyjaśnić i wyklarować, a potem potraktować jak feature change.

### Zmiana zachowania - CR, feature change
Szukamy testów sprawdzających dotychczasowe zachowanie, jeśli nie ma - dodajemy, zgodne z obecnym zachowaniem (powinny być zielone). Commit, opcjonalnie można zrobić push i podrzucić nazwę brancha komuś z zespołu - żeby spojrzał czy testy wyglądają jakby odpowiadały temu co powinno się dziać. Kolejny krok: bierzemy istniejące testy i zmieniamy/rozszerzamy je tak, żeby odpowiadały nowemu zachowaniu - po uruchomieniu te testy nie powinny przechodzić, testowany kod ciągle jest w "starej wersji". Next: zmieniamy kod tak, żeby testy przechodziły. Jak jest zielono, zmiana gotowa - commit, push, PR.

### Operacje na bazie danych - nie w unit testach
Przy naszej obecnej strukturze systemu, nie mamy dobrej możliwości mockowania bazy danych na potrzeby testów; ostatnio na snapshot bazy czekałem prawie tydzień, gdzie pojedynczy test suite (zestaw testów jednej klasy) powinien się wykonywać w max sekundę. :) Operacji na bazie danych nie testujemy, operacje na bazie mockujemy, w samym kodzie odwołującym się do bazy dajemy absolutne minimum logiki - idealnie, tylko przekazanie parametrów.

Testy na bazie to temat testów integracyjnych i E2E - ugryziemy to osobno i pewnie w trochę inny sposób w obecnej aplikacji, w nowych aplikacjach na netcore używających entity możemy łatwo ad-hoc stworzyć sobie bazę Sqlite in-memory z modelu i na takiej bazie testować - wtedy każdy test ma swoją osobną, niezależną bazę.

### Komunikacja i połączenia sieciowe - nie w unit testach
Podobnie jak przy bazie - unit testy nie wykonują żadnych połączeń do zewnętrznych systemów, miejsce na to jest w testach integracyjnych. W przypadku frontu - dotyczy to też połączeń do webapi: wywołania webapi mockujemy - sprawdzamy zapytanie, wyrzucamy stałą odpowiedź.

Dla testów integracji, systemy klienta mockujemy używając albo znanych, albo oczekiwanych komunikatów/odpowiedzi, ostatnie na czym chcemy w testach polegać to to, że w momencie wykonywania testu system klienta będzie działał i zachowywał się poprawnie (dodatkowo, daje nam to referencyjne odniesienie do tego, czego z systemów klienta się spodziewamy).

### End-to-end (całość danej funkcji) - raczej nie
Temat testów end-to-end (od zapytania do odpowiedzi) jest dyskusyjny, takie testy potrafią robić się nadmiernie skomplikowane i trudne w utrzymaniu. Jeśli testowana funkcjonalność jest na tyle prosta, że możemy ją sprawdzić end-to-end (całościowo) bez robienia z danego testu ściany kodu, takie testy są jak najbardziej ok; w pozostałych przypadkach będzie na to miejsce w testach integracyjnych/E2E, gdzie będziemy mieli też podłączoną bazę itd.

### Logika w testach (helpery itd) - tak
Wszelkiego rodzaju logika w testach (np. helper ustawiający nam nowy scope, helper wywołujący kod w `BeforeCommit` itd.) też powinny mieć testy sprawdzające ich prawidłowe działanie. Testujemy całą logikę w kodzie - w tym testy tam gdzie to potrzebne. :) Tego typu logika *zawsze* leci do osobnych helperów - przypominam, że nie wrzucamy żadnej logiki do samych testów.

## Jak pisać testy
Typowy unit test składa się z trzech elementów:
1. Arrange - przygotowanie warunków początkowych/założeń testu.
2. Act - wywołanie testowanego kodu: metody lub listy metod.
3. Assert - sprawdzenie wyniku, czy zgadza się z oczekiwaniami.

Zdarza się, że niektóre z tych elementów będą puste, albo sklejają się w jeden. Jeśli testujemy kawałek, który nie zależy od żadnego stanu (`czysta` w rozumieniu funkcyjnym funkcja) - nasze `Arrange` jest puste; jeśli sprawdzamy czy wywołanie metody rzuca wyjątek, `Act` i `Assert` sklejają się w jedną konstrukcję `Assert.ThrowsException<Hal9000Exception>(() => {subject.DoThing();});` lub `expect(() => subject.doThing()).toThrow(new Error("I'm afraid I can't do that, Dave"))`. To jest jak najbardziej ok, o ile ogólna struktura (to, że mamy te trzy fazy i tylko te trzy fazy, w odpowiedniej kolejności) jest zachowana.

### Mocki
Jeśli testujemy kawałek kodu, który zależy od innych kawałków kodu, to z tymi zależnościami coś trzeba zrobić - a żeby z "unit" testu nie zrobił nam się "test połowy funkcjonalności systemu", takie zależności możemy mockować.

C#:

Korzystamy z [Moq](https://github.com/moq/moq4) - lekka i przyjemna biblioteka do mockowania interfejsów, pozwalająca na zasymulowanie różnych operacji. Moq pozwala - w momencie konfiguracji - zdefiniować dla jakich parametrów/wywołań ma być zwrócona jakaś wartość, czy ma być rzucony wyjątek, oraz zbiera historię wywołań danego mocka - historii możemy użyć czy zależność jest wywoływana w odpowiedniej sytuacji, i czy to wywołanie ma prawidłowe parametry.

Przykład:

```csharp
[TestMethod]
public void DoTheRightThing_WillDoTheThingForDoableThingObject()
{
  // arrange: ustawiamy nasze warunki testu
  // parametr oznacza, że mock rzuci wyjątek jeśli wywołamy cokolwiek nieprzewidzianego
  Mock<ISelfService> selfServiceMock = new Mock<ISelfService>(MockBehaviour.Strict);
  // wywołanie `DoTheThing` z dowolnym obiektem typu `Thing` zwróci string "The Thing"
  // .Verifiable() dodaje wywołanie jest do listy oczekiwanych wywołań, które możemy na końcu (assert) sprawdzić
  //   bez powtarzania warunków wejścia
  selfServiceMock.Setup(mock => mock.DoTheThing(It.IsAny<Thing>())).Returns(() => "The Thing").Verifiable();
  // wywołanie `DoTheThing` z obiektem typu `Thing` mającym pole `IsDoable` równe `true` zwróci `true`
  // wywołanie tej metody nie będzie weryfikowane na końcu
  selfServiceMock.Setup(mock => mock.CanDoTheThingTo(It.Is<Thing>(thing => thing.IsDoable == true))).Returns(() => true);

  // act: wykonanie testowanej operacji
  // utwórz testowany obiekt przekazując mock - jako typ `ISelfService` w konstruktorze - po to wyrzucamy `Factory`
  var subject = new ServiceDoer(selfServiceMock.Object);
  // testowana metoda
  subject.DoTheRightThing(new Thing(){ IsDoable = true });

  // assert: sprawdzamy, czy metoda zrobiła co miała zrobić
  // explicit: sprawdza czy `DoTheThing` było wywołane dokładnie raz z obiektem typu `Thing` mającym `IsDoable` ustawione na true
  selfServiceMock.Verify(mock => mock.DoTheThing(It.Is<Thing>(thing => thing.IsDoable == true)), Times.Once);
  // set up: sprawdza czy wszystkie metody z `.Verifiable()` były wywołane
  selfServiceMock.VerifyAll();
}
```

```javascript
// założenie, że nie używamy narzędzia typu chai do eleganckiego mockowania zależności
describe('Workload availability control', () => {
  it('disables save button if current time is unavailable', async () => {
    // arrange: zamockuj WebbookingService
    const webbookingServiceMock = { 
      getAvailabilityTable = async (workloadId: int, selectedTime: DateTime) => {
        // sprawdzamy warunki wywołania serwisu - tu jako 'assert'
        expect(workloadId).toBe(20);
        expect(selectedTime).toBe(new DateTime(2020, 7, 5, 11, 30));

        // await konwertuje obiekt na resolved promise - konieczne ponieważ caller oczekuje metody asynchronicznej
        await return {
          OrderId: 20,
          PlannedDate: new DateTime(2020, 7, 5, 11, 30),
          // zwracamy 36h slotów w 15-minutowych odstępach, ustawmy tylko pierwszy slot na zajęty, reszta niech ma 1000 pracochłonności
          AvailabilityTable: (new Array(36*4*15)).map((_, i) => i == 0 ? 0 : 1000),
          // koszt 1 = tylko pierwszy slot powinien być uznany za zajęty
          WorkloadCost: 1,
          // ... inne parametry wymagane przy parsowaniu odpowiedzi
        };
      },
    };

    // act: tworzymy i wywołujemy serwis, założenie że $scope i $rootScope są zamockowane na poziomie testSuite
    const subject = new WebbookingAvailabilityController($scope, $rootScope, webbookingServiceMock, inne, zależności);
    await subject.checkAvailability(new DateTime(2020, 7, 5, 11, 30));

    // assert: sprawdzamy czy efekt jest prawidłowy
    expect(subject.canSave).toBe(false);
    // jeśli nasze testy potrafią ruszyć DOM, możemy tu też sprawdzić DOM, jak np:
    expect(subject.view.getById('saveButton').attr('disabled')).toBe('disabled');
  });
});
```

### Setup i cleanup

W testach możemy wspólny kod ustawiający warunki wejścia wyciągnąć z samego testu do osobnych metod - sprowadza to kod metody testowej do bardziej zwartej i kompaktowej postaci, szczególnie w sytuacji gdy mamy wiele testów współdzielących ten sam setup warunków wejścia (faza arrange). Jak zawsze - przypominam że w testach nie mamy żadnych instrukcji warunkowych i żadnej logiki - wszystko hardkodowane, na sztywno, stałe i niezmienne.

W C# - to metody z atrybutami `[TestInitialize]` i `[TestCleanup]` - setup i teardown powinny trafić tam, wrzucając mocki itd. do pól klasy testowej. Używanie konstruktora/destruktora jest niezalecane - runner .NET (MSTest) lubił od czasu do czasu robić reuse obiektu testowego, wywołując na nim różne testy w różnej kolejności, konstruktor/destruktor tworzą zależności pomiędzy testami, gdy init/cleanup są bezpieczne.

W naszej praktyce (moje i KW testy) przyjęło się, że - jeśli konstrukcja obiektu testowanego przyjmuje całą serię parametrów, klasa testowa dostaje dodatkową metodę `GetSubject()`, której jedynym zadaniem jest utworzenie i zwrócenie nowego obiektu do testów, dostającego odpowiednie mocki jako parametry. Kwestia czytelności - jeśli bezpośrednie `new` w kodzie testu nie sprawia, że staje się on 2-3x dłuższy, lepiej użyć `new`, `GetSubject()` ma za zadanie tylko poprawić czytelność.

W przypadku JS, mamy do dyspozycji `before()`, `beforeAll()`, `beforeEach()` i odpowiedniki `after*()` dla danego test suite - zasada podobna: jeśli kod ustawiający test nam się powtarza wielokrotnie, wyciągnijmy go do metody inicjalizującej, żeby testy były bardziej przejrzyste.

## Od czego zacząć
Od napisania testu. Serio, tu nie ma co kombinować - kolejny task zaczynamy od odpowiednich testów, testy uruchamiamy na bieżąco, sprawdzamy wyjście, nie robimy merge kodu, który nie przechodzi testów.

Przeglądając PR - patrzcie czy do zmian w kodzie są odpowiednie testy/zmiany w testach. Czytanie PR najlepiej zacząć od testów - pokazują *co* się zmieniło w zachowaniu, gdzie sam kod pokazuje *jak* zmiana została zaimplementowana; patrząc najpierw co miało się zmienić łatwiej się zorientować czy sama zmiana ma sens.

Testy są lepsze niż brak testów, więcej testów jest lepsze niż mniej testów - nie siadamy do napisania testów do wszystkiego od razu (patrząc na samo webapi, to roboty na około 3-5 miesięcy, o ile ofiara nie rzuci wszystkiego i nie wyjedzie w Bieszczady wypasać owce), piszmy testy do wpadających nowych zadań - zarówno zmian, jak i bugfixów. Przykłady w webapi: zmiany do tłumaczeń w raportach (bugfix i nowa funkcja), cały webbooking 2.0 (nowy moduł), workload (nowy klocek do obsługi awizacji).

Tak, wiemy że pisanie testów zajmuje czas, wiemy że to wpłynie na terminy - to jest ok, lepiej dowieźć trochę później coś co będzie wymagało mniej poprawek, a w razie kolejnych zmian te zmiany będą szybsze i mniej ryzykowne, niż szybko oddać cokolwiek, co potem może się wyłożyć, a o problemie dowiemy się od klienta.

**Visual Studio:** przy domyślnych skrótach `Ctrl+R,A` uruchamia wszystkie testy w projekcie, z poziomu Test Explorer możemy przejrzeć testy i uruchomić (w tym w debuggerze) dowolny wybrany test. Bezpośrednio w kodzie VS dodaje adnotacje pokazujące - dla danego fragmentu - ile testów go obejmuje, jakie to testy i czy testy przechodzą. Zielony znaczy dobrze, więcej zielonego = lepiej.

Fun fact: `Ctrl+R,A` wykonuje też build projektu jeśli jest potrzebny, nasze testy potrzebują - łącznie z podniesieniem runnera i załadowaniem wyników - dodatkowo około 10s na wykonanie (na mojej maszynie). Webapi wstaje i podpina się w debuggerze w około 45s od zakończenia buildu - to 35s różnicy przy każdym sprawdzeniu czy wszystko jest ok. Spamowanie testów zamiast build szybko wchodzi w nawyk, biorąc pod uwagę ile to w sumie oszczędza czasu - warto dla samego faktu ograniczenia testów manualnych do minimum (nie rezygnować z testów manualnych, przeklikać proces, unit testy przynajmniej zapewniają że część błędów będzie wyłapana zanim do przeklikania przejdziemy).

**AngularJS:** IIRC na chwilę obecną nie mamy podpiętego żadnego test frameworka. Zadanie dla chętnego - wziąć dokumentację AngularJS, podpiąć framework testowy (karma + jasmine), wrzucić jakiś przykładowy test (przypominam: red-green-refactor) i do `package.json` wrzucić odpowiednią komendę, tak żeby uruchomienie testów wymagało tylko `npm run test`.

**Nowy Angular:** jest karma. Karma to runner, który startuje w przeglądarce (wspiera Chrome Headless - Chrome bez okna) i wykonuje wszystkie testy. Komenda `npm run test` odpala wszystkie testy w trybie headless i w konsoli wypluwa rezultat, `npm run watch` odpala testy w trybie watch, z przeglądarką (Chrome) wykonującą testy przy każdym zapisie. W czasie pracy nie ma powodu nie korzystać z tej funkcji.

## Nasze podejście do testów

### Nazewnictwo i grupowanie testów

#### Legacy C# 
Jeden projekt testowy dla każdego projektu w solucji, nazwa `<projectname>.Tests`. Testy w dirtree siedzą obok danego projektu, w widoku solucji grupujemy je wszystkie w filtrze `Tests`.

#### Nowy C# 
Podobnie, jeden projekt testowy dla każdego projektu w solucji, z różnicą że per solucja/repo przyjmujemy jedno z dwóch założeń (trzymamy się zasady używanej w danym repo):
- testy trafiają do `/src` obok projektów testowanych; w solucji testy dla projektu są obok projektu
- testy trafiają do `/tests` (nazwa folderu z projektem to albo nazwa projektu testu, albo nazwa folderu z testowanym projektem); w solucji testy są zebrane razem, wewnątrz `tests`

#### C# ogólnie

Minimum jedna klasa testowa per test - dla jednej klasy nazwa to `Test<nazwatestowanejklasy>` lub `<nazwatestowanejklasy>Tests` (powinniśmy przyjąć jedno podejście - do dyskusji). Jeśli mamy sytuację, gdzie dla danej klasy mamy kilka różnych scenariuszy (w praktyce: kilka różnych zestawów initialize/cleanup) - każdy scenariusz to osobna klasa, nazewnictwo to albo `Test<nazwatestowanejklasy>_Scenariusz`, `<nazwatestowanejklasy>Tests_<scenariusz>` (jeśli scenariuszy jest mało - max. 2-3), albo osobny folder+namespace o nazwie `<nazwatestowanejklasy>Tests` lub `Test<nazwatestowanejklasy>` i wewnątrz testy, każdy o nazwie `<scenariusz>`. C# nie wymaga odpowiednich nazw klas/metod testowych (w przeciwieństwie do niektórych innych frameworków, wymagających np. żeby metoda testowa zaczynała się od `test`), więc nazwenictwo ma przede wszystkim być jasne i - z listy testów - dawać od razu informację, które testy nie przeszły.

Nazwenictwo metod testowych: są tu dwa rozwiązania, które stosujemy - dobierać pod dany test według potrzeb i preferencji - zakładam, że jednolite podejście nam się wyklaruje, w razie czego rename testów jest trywialny, w końcu nic ich nie wywołuje ręcznie. :)

##### PrzedmiotTestu_ScenariuszLubWarunkiWejściowe_OczekiwanyRezultat
Preferowane dla metod, gdzie dla różnych warunków mamy różne rezultaty. Scenariusz lub warunki wejściowe to krótki opis wejścia naszego testu (stan systemu, parametry metody), rezultat to oczekiwany wynik (co jest zwracane, jaki wyjątek rzuca, jaka metoda w jakiej zależności jest wywoływana). Dobra nazwa testu powinna - dla kogoś kto przegląda testy i widzi interfejs testowanej klasy - dawać pojęcie w jakiej sytuacji testowa metoda jest wywoływana i co dla tej sytuacji powinno się stać.

Czytać jako: `PrzedmiotTestu: When ScenariuszLubWarunkiWejściowe Then OczekiwanyRezultat`

Przykłady:
- `TestUpdateWebbookingOrderWithAppointment`: `OnEdit_DriverIdChanged_DoesNotChangeSavedDriverData`
- `TestUpdateWebbookingOrderWithAppointment`: `OnEdit_DriverIdChanged_WillResetOrderToDriverRelation`
- `TestNHibernateDataClasses`: `DataClass_Each_HasClassMapDefined` - meta-test, używa refleksji do sprawdzenia poprawności definicji NH
- `DriverModelBuilderTests`: `Build_WhenUserCanEditDisplayName_ThenDontBuildDisplayNameFromNameAndSurname` - `When` i `Then` są opcjonalne, IMO zbędne, wedle preferencji
- `TestFtlLogic`: `DistinctShipments_ValidArrayExists_ReturnsRecord` - patrząc na sam test (bez zaglądania do metody testowanej), nazwa mogłaby być zmieniona na `DistinctShipments_HasTasksForAllQueriedShipmentDetails_ReturnsAllTasks`, bardziej opisowa
- `AdminDeliveryCarriersServiceTests`: `Post_DeliveryCarrierDto_SettingRegion_Allowed_ShouldChangeRegion` - tutaj pierwsza część to `Post_DeliveryCarrierDto` (yay servicestack!), "when" to `SettingRegion_Allowed`, a `ShouldChangeRegion` to "then"

##### PrzedmiotTestu_CoPowinnoSięStać
Bardziej przypomina formę opisywania testów w JS, druga część opisuje oczekiwane zachowanie. Sprawdza się dla przypadków bezwarunkowych, gdy coś powinno dziać się zawsze lub nigdy - szczególnie jeśli cały scenariusz mamy wyłączony do osobnego pliku.

Nazwa testu w tym przypadku powinna dać się czytać jako zwięzły przypadek użycia - co system robi.

Przykłady:
- `TestAuthLogic/MobileLogin`: `Login_PutsUserNameInAuditLog` - sprawdza czy przy próbie logowania mobile do auditlog dodawany jest wpis z nazwą użytkownika
- `TestAuthLogic/WebLogin`: `Login_SavesPasswordChangeRequiredFlagForExpiredPassword` - sprawdza czy przy próbie logowania web ustawiana jest flaga "wymaga zmiany hasła" jeśli ważność hasła została przekroczona
- `TestWebbookingScheduler`: `GetUnavailability_CombinesTimedAndClockUnavailabilities` - pobierana niedostępność ma połączoną niedostępność opartą o czas i opartą o zegar

#### Angular.js

Wytyczne z dokumentacji Angular.js / karma+jasmine jeśli Angular nic nie mówi. W razie braku informacji - osobny podfolder `test` w głównym (`WebApp`) folderze aplikacji. Struktura katalogów/plików odpowiada 1:1 strukturze plików w `app` - z wyjątkiem, że każdy plik testu musi kończyć się na `.test.js` lub `.spec.js` (wymóg runnera).

JS pozwala na bardzo opisowe nazwy testów - jako parametr opisowy test suite (funkcja `describe()`) wrzucamy nazwę scenariusza/testowanej rzeczy (**co**), jako opis testu (funkcja `it()`) wrzucamy założenie, które jest testowane (**robi**). Całość powinna dać się czytać jak w miarę sensowne zdanie, np: `Test workload availability | marks all unavailable fields as disabled`.

#### Angular (nowy)

Pliki o nazwie odpowiadającej nazwie testowanego klocka z rozszerzeniem `.spec.ts` (TypeScript) lub `.spec.js` (JavaScript, jeśli bardzo trzeba) - dla testów dotyczących konkretnego komponentu/serwisu/klasy plik z odpowiadającą nazwą obok danego komponentu/serwisu/klasy. Jeśli test dotyka kilku komponentów (co w sumie oznacza test integracyjny, takie jak najbardziej też można, o ile ogólne wytyczne - stabilność, testowana jedna rzecz, niezależność od świata zewnętrznego - są spełnione) lecą do podfolderu `tests` w folderze głównym aplikacji. W przypadku testów integracyjnych w `tests` - nazwa pliku powinna odpowiadać nazwie scenariusza.

Nazwenictwo samych testów identycznie jak w angularjs.

TODO: @BW: dostosować runner nowego angulara tak żeby w pierwszej kolejności łapał unit testy z `src`, a dopiero później leciał po `tests`.

### Framework testowy

Dla Angular sprawa jest prosta - używamy tego co Angular zaleca i wspiera, tu nie bardzo mamy wybór.

Dla C# - nasze główne opcje to MSTest, nUnit i xUnit. Niejako z przypadku decyzja zapadła na używanie MSTest - główne powody to dobre wsparcie dla trzymania testów osobno (xUnit preferuje testy w tym samym pliku/projekcie co klasa testowana), runner (spina się z VS bez żadnych pluginów, daje się uruchamiać bezpośrednio z `MSBuild`/`dotnet`), konfiguracja w pełni na atrybutach i całkiem dobra wydajność przy uruchamianiu z poziomu VS.

Nie ma problemu z mieszaniem różnych bibliotek testowych w jednej solucji gdyby była taka potrzeba - więc jeśli gdzieś z jakiegoś powodu MSTest nie daje rady, możemy użyć xUnit lub nUnit.

### Mocki

Dla angular - karma+jasmine+ngMock+tworzenie obiektów ręcznie powinny dawać sobie radę, jeśli potrzebny jest jakiś framework do mocków - doinstalować, nie orientuję się co jest teraz popularne.

Dla C# - Moq to de-facto standard dla mocków. Mockowanie bazy danych w netcore robimy tworząc instancję bazy używającą in-memory sqlite.


