# turbohack
### Problem
Analizując proces kredytowy zauważyliśmy, że mimo przenoszenia usług bankowych do internetu, aby wziąć kredyt hipoteczny nadal trzeba stawić się w placówce banku na podpisanie umowy. Po konsultacji z prawnikiem udało nam się ustalić, że nie ma przeszkód prawnych aby proces ten odbywał się online. Obecny stan rzeczy wynika głównie z ostrożności banków. Aby mieć pewność co do tożsamości klienta, chcą aby wylegitymował się fizycznym dowodem osobistym, którego autentyczność jest sprawdzana.

### Koncepcja rozwiązania
Zachowawczość banków jest uzasadniona - w końcu chodzi tu o duże kwoty. Zauważyliśmy jednak, że dowód sprawdzany jest wiele razy, w szczególności każdy musiał go okazać zakładając konto w oddziale któregokolwiek z banków.<br/>
#### Czy nie można wykorzystać raz zdobytego potwierdzenia aby uniknąć w przyszłości konieczności wybierania się do oddziału? <br/>
Naszym zdaniem jest to możliwe. W tym celu proponujemy wykorzystanie rządowej aplikacji mObywatel jako identyfikatora klienta w sieci. Kiedy klient raz uda się do banku X i jego tożsamość zostanie potwierdzona i połączona z kontem w mObywatelu, bank X może wpisać tę informację do międzybankowej bazy danych wystawiając klientowi legitymującemu się danym kontem mObywatela "certyfikat zaufania".
Kiedy w przyszłości ten klient będzie ubiegał się o kredyt w banku Y, wystarczy że bank ten sprawdzi, że jego tożsamość została już raz zidentyfikowana przez zaufaną insytucję (Bank X) i nie ma potrzeby powtarzania tego.<br/>
Oczywiście pojawia się jeszcze wątpliowść - co jeśli do konta mObywatel dostała się nieuprawniona osoba? W teorii konto to powinno być chronione dobrym hasłem znanym tylko właśicielowi, w praktyce jednak bywa różnie. Dlatego jako finalny etap weryfikacji proponujemy krótką rozmowę wideo prowadzoną z konsultantem przez aplikację banku. W jej trakcie pracownik może stwierdzić czy to rzeczywiście klient podpisuje umowę (na podstawie wyglądu, pytań bezpieczeństwa), czy robi to dobrowolnie i czy jest w stanie pozwalającym mu na podejmowanie takich decyzji. Natomiast korzystanie z aplikacji banku wyklucza możliwość manipulacji wideo.
W ten sposób uzyskanie kredytu przez niewłaściwą osobę wymagało by sobowtóra i włamania się na konto bankowe oraz aplikację mObywatel klienta, co jest porównywalnie trudne jak kradzież dowodu i udanie się przez sobowtóra do banku, aby wziąć kredyt w tradycyjny sposób.

### Opis rozwiązania od strony technicznej
Proponowana przez nas architektura rozwiązania oparta jest o szynę ESB, która pozwala na łatwe łączenie ze sobą różnych komponentów, zapewnia elastyczność oraz prostotę modyfikacji.
Do szyny tej bezpośrednio mogą podpinać się banki próbujące potwierdzić tożsamość klienta. Dodatkowo podpięta będzie baza danych zawierająca certyfikaty zaufania, aplikacja sterujaca całym procesem oraz klient mObywatela, który będzie pozwalał potwierdzać dane.
Ze względów bezpieczeństwa wszystkie połączenia są szyfrowane, a baza danych musi wykonywać cykliczne backupy do hurtowni danych. Schemat architektury widoczny jest na obrazku poniżej.

![](https://i.imgur.com/aEozdiC.png)

Flow:
1) Klient klika "podpisz umowę przy pomocy mObywatela" w aplikacji banku, aplikacja poprzez bank wysyła zapytanie do naszego systemu, które przyjmuje aplikacja sterujaca i odpytuje klienta mObywatela.
2) Następuje przekierowanie i prośba o zalogowanie do mObywatela przy pomocy hasła (plus opcjonalnie biometrii), serwer mObywatela potwierdza tożsamość.
3) Aplikacja sterująca otrzymuje tożsamość (dane identyfikacyjne) klienta z serwera mObywatela.
4) Aplikacja sterująca sprawdza w bazie danych, czy jest tam już "certyfikat zaufania" wystawiony przez ten sam lub inny bank.
5) Po otrzymaniu pozytywnej odpowiedzi, zatwierdza i wysyła odpowiedź do banku.
6) Po otrzymaniu odpowiedzi aplikacja banku przechodzi do dodatkowej weryfikacji wideo.
