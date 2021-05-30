# Testowanie kodu legacy

Prawdopodobnie najtrudniejsze do napisania testy będą dotyczyły obecnie istniejącego, produkcyjnego, generalnie działającego kodu. Nie mamy czasu, żeby przygotować pełne pokrycie testami dla wszystkiego (to zajmie miesiące nie robiąc w tym czasie nic innego!), więc przy testowaniu legacy podstawowe założenie to: piszemy testy dające **maksymalny efekt przy minimum nakładu pracy**.

## Testy top-down
W przeciwieństwie do [TDD](../tdd.md), które zakłada testowanie bottom-up, w przypadku istniejącego kodu w pierwszej kolejności potrzebujemy testy ogólne, sprawdzające całościowo szersze procesy, bez zagłębiania się w szczegóły. Jeśli np. piszemy test związany ze zmianą sposobu wyszukiwania planu, które jest używane przy dodawaniu zadania do planu i mamy czas na zrobienie tylko jednego testu do istniejącej funkcjonalności, niech to będzie test sprawdzający cały proces dodawania nowego zadania do istniejącego planu. Na przetesowanie samego wyszukiwania przyjdzie czas, na razie o wiele większą wartość ma dla nas test sprawdzający, nawet w bardzo uproszczony sposób, całość operacji - jeśli jakaś inna zmiana w przyszłości będzie miała wpływ na ten proces, będziemy mieć test, który to w jakimś stopniu weryfikuje.

## Testujemy procesy, nie implementacje
To szczególny problem przy testowaniu istniejącego kodu - łatwiejsze może być napisanie testu, który sprawdza konkretną implementację (to jak dane zachowanie jest realizowane) zamiast procesu (to, co dane zachowanie ma osiągnąć). Pisząc test zwracać na to uwagę - kod testu powinien pokazywać "co dana funkcja/proces robi" zamiast "jak dana funkcja/proces są realizowane" - subtelna, ale bardzo istotna różnica. Jeśli będziemy pisać testy pod implementację, to w momencie zmiany implementacji, nawet jeśli efekt końcowy procesu jest identyczny, test przestanie działać.

## Priorytet ścieżek wykonania do testów
W kolejności: 
- główna/podstawowa ścieżka wykonania
- przypadki szczególne i wyjątki
- efekty uboczne i zachowania powiązane
- obsługa błędów
- diagnostyka (logi, metryki)

Dla testów istniejącej funkcjonalności najistotniejsze dla nas jest, żeby wszystko działało prawidłowo w odpowiedniej sytuacji - to że możemy mieć wytestowane wszystkie przypadki braku uprawnień dla danej operacji niewiele nam pomaga jeśli nie mamy testu, który sprawdza czy - przy odpowiednich uprawnieniach - operacja robi co ma robić.

## Testy nie są opcjonalne
Przeglądając PR patrzeć, czy do wprowadzanych zmian są odpowiednie testy - na poziomie szczegółowości odpowiadającym zakresowi zmian w PR, lub bardziej ogólne jeśli dany kawałek nie miał testów. Test powinien uwzględniać różnice w zachowaniu wynikające ze zmienionego w PR kodu - powinien weryfikować czy zmiana działa prawidłowo. PR bez odpowiednich testów odbijamy. Wyjątkiem jest sytuacja, gdy dany fragment jest realnie "nietestowalny" - jeśli zmiana dotyczy (niemal) w całości odwołania do bazy danych lub komunikacji z zewnętrznym systemem i nie wnosi żadnych istotnych zmian w działaniu systemu poza tymi zależnościami.

## Przykład: C#

```csharp
Mock<PlanRepository> _planRepositoryMock;
Mock<TaskRepository> _taskRepositoryMock;
Mock<IAuth> _authMock;
Mock<IClock> _clockMock;

// wszystkie wartości używane w testach to stałe, NIE używamy bieżącego czasu/locale/IP/strefy czasowej itd.
static readonly DateTime _now = new DateTime(2021, 5, 29, 15, 0, 0);

[TestInitialize]
public void Initialize()
{
  _planRepositoryMock = new Mock<PlanRepository>();
  _taskRepositoryMock = new Mock<TaskRepository>();
  _authMock = new Mock<IAuth>();
  // uprawnienia poza scope testu - zakłada że user ma wszystkie; testy sprawdzające uprawnienia mogą nadpisać to zachowanie
  _authMock.Setup(mock => mock.HasPermission(It.IsAny<string>())).Returns(true);
  _authMock.Setup(mock => mock.HasAccessToClient(It.IsAny<int>())).Returns(true);
  // ustawiamy sztywny symulowany czas testu
  _clockMock.SetupGet(mock => mock.Now).Returns(() => _now);
}

private TaskService GetSubject()
{
  // TaskLogic nie ma testów - zamiast mockować zachowanie, ładujemy zależność bezpośrednio, nasz test sprawdza całość zachowania aż do poziomu repozytoriów/bazy danych
  // null to zależności, które w ramach testowania tej funkcjonalności nie powinny być używane/mieć żadnego znaczenia
  return new TaskService(
    new TaskLogic(_taskRepositoryMock.Object, null, _clockMock.Object, null, null),
    _planRepositoryMock.Object,
    null,
    null,
    _authMock.Object,
    null
  );
}

// scenariusz: w momencie zlecenia dodania nowego zadania do istniejącego planu, zadanie jest zapisywane w bazie danych z prawidłowym ID planu
[TestMethod]
public void AddTask_ExistingPlan_TaskWithPlanIsSaved()
{
  // arrange: przygotuj zwracanie istniejącego planu, w testach wszystkie wartości hardkodujemy
  // na potrzeby zapisu plan nie powinien potrzebować więcej niż ID klienta i swój własny ID
  _planRepositoryMock.Setup(mock => mock.GetById(42L))
    .Returns(() => new Plan{ Id = 42L, ClientId = 18 });

  // act: stwórz obiekt testowanej klasy, wywołaj testowaną metodę/-y z odpowiednimi parametrami
  var request = new AddTaskDto
  {
    ClientId = 18,
    PlanId = 42L,
    TaskType = (TaskTypes)4,  // jeżeli API dostaje ID jako int, używamy int; jeśli na wejściu trafia enum/string, używamy odpowiedniej wartości
    ResponsibleEmployeeId = null, // najprostsza ścieżka wykonania - nie przypisujemy zadania do osoby
    StartDate = new DateTime(2021, 5, 30, 12, 0, 0), // task wymaga planowanej daty startu w przyszłości, wymaganie biznesowe, które jest uwzględnione w kodzie
    Name = "Do stuff",
  };
  var subject = GetSubject();
  subject.Post(request);

  // assert: zweryfikuj czy testowany kod zrobił co miał zrobić
  // tu oczekujemy dodania nowego zadania z odpowiednimi parametrami
  _taskRepositoryMock.Verify(
    mock => mock.Add(It.Is<Task>(task => task.PlanId == 42L && task.Name == "Do stuff" && task.Type == 4)),
    Times.Once
  );
}
```

Tak przygotowany test jest świetny jako pierwszy test dla procesu dodawania nowych zadań do planu:
- sprawdza najprostszą wersję dla głównej funkcji procesu - udane dodanie nowego zadania w scenariuszu bez dodatkowych komplikacji
- weryfikuje całość procesu, aż do punktu osiągnięcia bazy danych - jeśli gdziekolwiek po drodze pojawi się zmiana która wpływa na tą minimalną ścieżkę, test będzie wymagał aktualizacji, ale jednocześnie da sygnał żeby na proces spojrzeć
- weryfikuje jedną, konkretną, kluczową z punktu widzenia procesu funkcjonalność - zapis dodanego zadania w bazie danych jest główną funkcją dodawania nowego zadania
- testuje zachowanie, a nie implementację - wszystkie nieistotne dla testu detale implementacji są albo pominięte, albo zamockowane (uprawnienia, bieżący czas)

W miarę zmian do tego procesu kolejne testy do dodania powinny dotyczyć np.:
- dodawanie zadania przypisanego do osoby, weryfikacja czy przypisanie jest wykonywane
- automatyczne tworzenie planu jeśli tworzone jest nowe zadanie bez ustalonego planu
- obsługa błędu: weryfikacja dostępu klienta do planu (ID klienta w zapytaniu nie zgadza się z ID klienta w planie)
- uprawnienia: sprawdzenie uprawnień koniecznych do dodania planu
- uprawnienia: sprawdzenie dostępu bieżącego użytkownika do klienta, którego plan modyfikujemy
- zapisu operacji w historii zmian dla planu/zadania
- inne, zależnie od oczekiwanego zachowania funkcji

## Przykład: JS
```javascript
describe("SchedulingController", () => {
  // scenariusz: po udanym zapisie harmonogramu, wyświetlany harmonogram powinien być odświeżony używając danych z odpowiedzi na zlecenie zapisu
  it("refreshes scheduler after save", () => {
    // arrange: przygotuj zależności zewnętrzne/wejście
    // prosty mock - ma za zadanie zachować się jak klient serwisu API, ale bez odwołania do HTTP
    // zamiast tego można użyć biblioteki do mocków
    const schedulingServiceMock = {
      callback: null, // tu zapiszemy sobie callback, w którym przekazujemy odpowiedź HTTP do testowanego kodu
      data: null, // tu zapisujemy zapytanie, chcemy zweryfikować czy to co jest wysyłane zgadza się z oczekiwaniami
      // zapis harmonogramu - testujemy reakcję kodu na ten fragment
      save: (id, scheduleData) => {
        const self = this;
        this.data = scheduleData;
        return {
          then: function(cb) {
            self.callback = cb;
          },
        };
      },
      onGet: null, // tu przechowujemy tymczasowo callback, w którym przekazujemy dane domyślnie ładowane
      // kontroler potrzebuje pobrać istniejące dane zanim będziemy mogli coś zapisać
      get(id) => {
        expect(id).toBe(335);
        const self = this;
        return {
          then: function(cb) {
            self.onGet = cb;
          },
        };
      },
    };
    // odpowiedź jaką zwrócimy
    const response = {
      Id: 335,
      Entries: [
        {Date: '2021-05-30', TimeFrom: '13:00', TimeTo: '17:00'},
      ]
    };

    // act: stwórz obiekt testowy, wykonaj kolejne operacje prowadzące do zapisu harmonogramu
    // jako null wszystko to, co jest możliwe do pominięcia na potrzeby tego testu
    const subject = new SchedulingController({}, {}, null, schedulingServiceMock, null, null);
    // symulujemy załadowanie obiektu testowego, ID jest ustawiane normalnie z poziomu bindingów HTML
    subject.scheduleId = 335;
    subject.load();
    // zwróć odpowiedź przy ładowaniu, minimum potrzebnych danych
    schedulingServiceMock.onGet({ Id: 335, Name: 'Harmonogram' });
    // symulacja zachowania użytkownika: ustawia datę, wybiera godziny
    subject.$scope.setDate(new Date(2021, 5, 30));
    subject.$scope.addScheduleEntry(13, 2);
    // symulacja zachowania użytkownika: klika "zapisz"
    subject.$scope.saveForm();

    // assert: obsługujemy zapytanie, sprawdzamy wynik
    // obsłuż callback - to równie dobrze może być traktowane jako część 'act'
    schedulingServiceMock.callback(response);
    // sprawdzamy czy treść zapytania była zgodna z tym co powinno być wysłane
    expect(schedulingServiceMock.data).toEqual({Id: 335, Date: '2021-05-30', Entries: [{ Hour: 13, Duration: 2 }]});
    // sprawdzamy czy po otrzymaniu odpowiedzi kontroler ma prawidłowe dane, które są wyświetlane użytkownikowi
    expect(subject.$scope.entries[0].StartHour).toBe(13);
    expect(subject.$scope.entries[0].EndHour).toBe(17);
    expect(subject.$scope.entries[0].Duration).toBe(4);
  });
});
```

Test jest trochę bardziej złożony niż przypadek dla C#, głównie dlatego że zakłada naszą przypadłość wszystkorobiących kontrolerów.

Testowany scenariusz: użytkownik ładuje widok harmonogramu z pustymi danymi, dodaje nowy wpis, zapisuje harmonogram; w odpowiedzi powinien otrzymać aktualny stan harmonogramu z backend.

Weryfikowane zachowania (można, dobrze by nawet było to rozbić na kilka testów, ale lepszy taki test całościowy niż nic):
- załadowanie kontrolera, w tym załadowanie danych z backend
- wysłanie prawidłowych danych po operacjach wykonanych w UI
- weryfikacja czy odpowiedź z serwera została prawidłowo zastosowana do danych przekazywanych jako model do HTML

## Przykład: Android
Musicie przygotować sobie we własnym zakresie, nie jestem na tyle mocny w Kotlin żeby pokazać dobry przykład. W razie wątpliwości @KT.
