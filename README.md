# Kody okien — jednoznaczne adresowanie ekranów systemu (enova365)

> Narzędzie przekrojowe, wspólne dla obu rodzin:
> [obieg faktur zakupu z KSeF](https://github.com/trynityeu/enova365-obieg-faktur-ksef) ·
> [kartoteka, sprzedaż i fakturowanie](https://github.com/trynityeu/enova365-obieg-sprzedazy-i-kartoteki)

Dodatek do systemu ERP **enova365** (Soneta sp. z o.o.), który nadaje **każdemu
ekranowi programu krótki kod** widoczny w nagłówku i pozwala ten kod wpisać
w globalną wyszukiwarkę, żeby otworzyć dokładnie to okno — a przy dokumentach
także **ten konkretny dokument, na tej konkretnej zakładce**.

To repozytorium zawiera wyłącznie **opis funkcjonalny** — bez kodu źródłowego
i bez danych klienta.

## Problem, który rozwiązuje

W instrukcji, w zgłoszeniu do wsparcia i w rozmowie telefonicznej trzeba
umieć wskazać **to jedno okno**. Opis słowny — „Handel, potem zakup, potem
zamówienia, potem otwórz dokument i przejdź na trzecią zakładkę" — jest długi,
zawodny i starzeje się przy każdej zmianie układu menu.

Problem ma trzy odmiany, z których każda kosztuje osobno:

- **pisanie instrukcji** — autor musi opisać drogę do ekranu, czytelnik ją
  przejść, a przy zmianie menu opis trzeba poprawić w każdym dokumencie;
- **zgłoszenia do wsparcia** — użytkownik opisuje miejsce po swojemu, wsparcie
  zgaduje, i połowa wymiany zdań idzie na ustalenie, o który ekran chodzi;
- **odsyłanie do systemu** — odpowiedź „to ustawia się tutaj" wymaga podania
  drogi, a nie da się jej kliknąć.

## Jak działa

- **Każda strona formularza i każda lista** dostaje w nagłówku krótki kod
  postaci `XXX-XXX`, gotowy do skopiowania.
- **Globalna wyszukiwarka** przyjmuje ten kod i otwiera odpowiadające mu okno.
- Na formularzu konkretnego rekordu kod rozszerza się do **adresu**
  `KOD/IDENTYFIKATOR` — wtedy otwiera się nie sama lista, tylko **ten dokument
  i ta zakładka**.

### Kod jest ten sam na każdej instalacji

Kod nie jest numerem z bazy ani kolejnym licznikiem — to **funkcja skrótu
policzona z kanonicznej tożsamości okna** (typu ekranu i nazwy strony).
Wynika z tego rzecz najważniejsza praktycznie: **ten sam ekran ma ten sam kod
wszędzie**. Instrukcja napisana raz działa w każdej instalacji, na środowisku
testowym i produkcyjnym, i nie wymaga tabeli przekładowej.

Identyfikator rekordu w adresie to **globalny identyfikator, stabilny także po
skopiowaniu bazy** — a gdy tabela go nie ma, numer rekordu. Adres z innej
instalacji, wskazujący rekord, którego tu nie ma, kończy się **cichym
otwarciem listy** zamiast błędem.

Drobiazg, który widać dopiero w praktyce: identyfikator wyświetlany jest
**wielkimi literami**, bo tak łatwiej go podyktować przez telefon i przepisać.
Odczyt pozostaje niewrażliwy na wielkość liter, więc adresy zapisane wcześniej
nadal działają.

## Katalog budowany w trakcie pracy

Najciekawsze rozwiązanie w tym dodatku i dobry przykład obejścia ograniczenia,
którego nie da się usunąć.

Naturalny sposób zbudowania listy wszystkich okien to przejście po **drzewie
uprawnień**. Problem: system **wyklucza z tego drzewa strony oznaczone jako
niepodlegające uprawnieniom** — typowo panele i pulpity. Pełna lista stron jest
w systemie dostępna, ale wewnętrznie, więc dodatek nie może z niej skorzystać.
Skutek był jawnie sprzeczny: panel pokazywał swój kod, a wyszukiwarka tego kodu
nie znajdowała.

Rozwiązanie odwraca kierunek: **nakładka nagłówka rejestruje każde okno
w chwili, gdy wyświetla jego kod**. Powstaje katalog budowany w trakcie
normalnej pracy, a wyszukiwarka pyta po kolei: **katalog → drzewo uprawnień →
foldery nawigatora**.

Zasada, którą to daje, jest prosta i mocna: **cokolwiek użytkownik zobaczył na
ekranie, jest wyszukiwalne** — niezależnie od uprawnień i rodzaju okna. Okna
nigdy nieodwiedzone pokrywa nadal enumeracja przez drzewo uprawnień i foldery.

## Okna parametrów czynności — czego nie da się zrobić i co zrobiono zamiast

Okna parametrów uruchamianych czynności (np. „wybór pozycji dla dokumentu
podrzędnego") **nie są samodzielnymi oknami**. Powstają dopiero po uruchomieniu
czynności na zaznaczonych rekordach, a ich treść zależy od tego zaznaczenia —
system nie ma wejścia „otwórz okno parametrów". Odtworzenie takiego okna to
z definicji **powtórzenie czynności na tych samych rekordach**.

Zamiast udawać, że się da, dodatek robi tyle, ile jest możliwe: klasa
parametrów jest zagnieżdżona w klasie czynności, więc przez metadane da się
dojść do **typu rekordu, a stąd do listy, z której tę czynność się uruchamia**.
Klik w wynik otwiera tę listę, a etykieta mówi wprost, że chodzi o okno
parametrów czynności.

Przy takim adresie identyfikator wskazuje **dokument źródłowy** — ten, na
którym czynność uruchomiono. Zgłoszenie brzmi więc „czynność X na dokumencie
Y", a klik prowadzi do tego dokumentu.

## Rejestr „gdzie bywał operator"

Obok katalogu kodów dodatek prowadzi drugą mapę: **dla każdego operatora
historię ostatnich odwiedzonych okien** — kod, opis, identyfikator rekordu
i znacznik czasu. Wypełnia ją ten sam punkt, który rysuje nagłówek, bo tylko
tam znane są **jednocześnie** okno, rekord i operator.

Powód istnienia jest konkretny. Dodatki odpowiadające użytkownikowi na pytania
o system rozmawiają z nim w oknie konwersacji — **bez związku z dokumentem**.
Kontekstem rozmowy nie jest więc „bieżący rekord", tylko ekrany, po których
użytkownik przed chwilą chodził. Mając tę listę, asystent pokazuje ją
i prosi o wskazanie, którego okna dotyczy pytanie — zamiast zgadywać.

Trzy szczegóły przesądzają o tym, czy rejestr jest użyteczny:

- **Okna konwersacji są pomijane.** Bez tego kliknięcie w panel czatu
  nadpisywałoby kontekst dokumentu, o który użytkownik właśnie pyta — rejestr
  zapisywałby sam siebie.
- **Kluczem jest operator, nie zmienna globalna.** W kliencie przeglądarkowym
  jeden proces serwera obsługuje wszystkie sesje, więc pojedyncza zmienna
  mieszałaby kontekst różnych osób.
- **Zapis jest w pełni odporny na błędy.** Nakładka diagnostyczna nie może
  wywrócić renderowania formularza — na przykład w sesji, w której nie ma
  zalogowanego operatora. Jeśli rejestr nie zdoła nic zapisać, okno i tak się
  wyświetli.

## Szczegół, który kosztował osobną poprawkę

Kontekst jest **współdzielony przez wszystkie zakładki jednego okna**. Naiwna
implementacja — jedno pole na kod — powodowała, że wszystkie zakładki czytały
tę samą wartość, więc przełączanie ich pokazywało kod nieodświeżony, należący
do zakładki oglądanej wcześniej.

Każda zakładka rezerwuje więc **własną komórkę** i wiąże się z nią indeksem.
Ta sama tożsamość okna dostaje zawsze tę samą komórkę, więc tablica nie
rozrasta się przy każdym otwarciu.

## Pochodzenie: mapa uprawnień

Dodatek powstał jako narzędzie do **mapowania uprawnień**: kod jest skrótem
tożsamości okna, a ta odpowiada węzłowi w drzewie uprawnień. Stąd bierze się
jego przydatność przy pytaniach „kto ma dostęp do tego ekranu" i „gdzie
w drzewie praw znaleźć tę pozycję" — kod jest wspólnym adresem dla obu spraw.

## Ograniczenia, które trzeba znać

- **Katalog ma zasięg procesu.** W kliencie przeglądarkowym kody zbierają się
  od wszystkich sesji jednego procesu serwera; w kliencie stacjonarnym — od
  jednej instancji. Ponowne uruchomienie zaczyna zbieranie od nowa, ale
  wyszukiwanie nadal działa przez drzewo uprawnień i foldery.
- **Okien parametrów czynności nie da się otworzyć wprost** — patrz wyżej.
- **Adres z rekordem jest w pełni przenośny dla tabel z identyfikatorem
  globalnym**; przy pozostałych używany jest numer rekordu, który po
  skopiowaniu bazy może wskazywać co innego.
- **Rekord podrzędny** (element wewnątrz dokumentu) bywa niemożliwy do
  otwarcia bez rodzica — wtedy otwiera się dokument nadrzędny.

## Co jest do tego potrzebne

- enova365; dodatek nie wymaga cech, słowników ani definicji dokumentów.
- Uprawnienia użytkownika do ekranów, które zamierza otwierać — dodatek nie
  omija kontroli dostępu, a jedynie skraca drogę do miejsca, do którego
  użytkownik i tak ma prawo wejść.

## Zgodność

| | |
|---|---|
| System | enova365 (Soneta sp. z o.o.) — klient stacjonarny i przeglądarkowy |
| Postać | nakładka nagłówka formularzy + dostawca globalnej wyszukiwarki |
| Zapisuje | **nic w bazie** — katalog i rejestr żyją w pamięci procesu |
| Zmienia dokumenty | **nie** |
| Integracje zewnętrzne | **brak** |
| Zakres | wszystkie ekrany programu, także panele i pulpity |

## Czego tu nie ma

Kod źródłowy, dane klienta oraz konfiguracja specyficzna dla instalacji
pozostają w repozytorium prywatnym. To repo służy wyłącznie jako publiczny
opis funkcji dodatku.
