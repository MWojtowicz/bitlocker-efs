# Skrypt prelegenta – „Ochrona danych w spoczynku: BitLocker vs. EFS”

Łączny czas: ok. 30 min + pytania. W prezentacji: **N** – notatki na ekranie, **P** – osobne okno prelegenta z zegarem.

## Slajd 1: Ochrona danych w spoczynku  ·  ⏱ 0:45

Dzień dobry. Tematem mojej prezentacji jest ochrona danych w spoczynku, a konkretnie dwa mechanizmy, które Microsoft wbudował w Windows: BitLocker oraz EFS, czyli Encrypting File System.

Oba szyfrują dane na dysku, oba są „za darmo” w systemie – a mimo to chronią przed zupełnie innymi zagrożeniami. Pod koniec chcę, żebyście potrafili odpowiedzieć na pytanie: kiedy który, a kiedy oba naraz.

Uprzedzam, że będzie trochę memów – bo bezpieczeństwo bez humoru to po prostu audyt.

## Slajd 2: Plan na 30 minut  ·  ⏱ 0:30

Plan jest prosty. Najpierw powiemy sobie, czym w ogóle są dane w spoczynku i dlaczego hasło do Windowsa ich nie chroni.

Potem BitLocker – jak działa, skąd bierze klucze i jakie są na niego znane ataki. Następnie EFS, jego architektura i pułapki. Na koniec bezpośrednie porównanie i zestaw dobrych praktyk.

## Slajd 3: Trzy stany danych  ·  ⏱ 1:15

W bezpieczeństwie mówi się o trzech stanach danych. Dane w ruchu – przesyłane przez sieć – chronimy TLS-em, VPN-em, SSH. Dane w użyciu to to, co aktualnie siedzi w RAM-ie i w rejestrach procesora.

Nas dziś interesuje trzeci stan: dane w spoczynku, czyli wszystko, co leży zapisane na nośniku – dysk w laptopie, SSD w serwerze, pendrive, taśma z backupem.

Kluczowa obserwacja: nośnik jest fizycznym przedmiotem. Można go zgubić, ukraść, wyrzucić przy wymianie sprzętu. I wtedy żadne mechanizmy systemu operacyjnego już nie działają, bo system… nie jest uruchomiony.

## Slajd 4: Scenariusz nr 1  ·  ⏱ 1:15

Scenariusz numer jeden, klasyka: ktoś zostawia służbowego laptopa w pociągu albo w taksówce. Sprzęt jest najmniej ważny – laptop kosztuje kilka tysięcy złotych. Dane na nim mogą kosztować dużo więcej.

Z perspektywy RODO zgubienie niezaszyfrowanego nośnika z danymi osobowymi to naruszenie, które trzeba zgłosić do UODO i często także osobom, których dane dotyczą. Jeżeli natomiast dane były skutecznie zaszyfrowane, ryzyko dla tych osób jest znikome i obowiązek zawiadamiania ich zwykle odpada.

Czyli szyfrowanie to nie tylko technika – to też bardzo konkretna oszczędność stresu i pieniędzy.

## Slajd 5: „Ale przecież mam hasło do Windowsa!”  ·  ⏱ 1:30

Częsta odpowiedź brzmi: „ale ja mam hasło do Windowsa”. Problem w tym, że hasło logowania kontroluje dostęp do działającego systemu. Uprawnienia NTFS, listy ACL – to wszystko egzekwuje jądro Windowsa.

Wystarczy uruchomić komputer z pendrive'a z Linuksem albo wyjąć dysk i podłączyć go przez adapter USB do innego komputera. Linux czy inny Windows przeczyta NTFS i po prostu zignoruje nasze ACL-e. Hasło nawet nie zostanie sprawdzone.

Jedyną realną ochroną danych w spoczynku jest kryptografia: bez klucza na dysku jest tylko szum. Pytanie brzmi: co szyfrujemy i gdzie trzymamy klucz. I tu pojawiają się dwa podejścia.

## Slajd 6: Dwa podejścia  ·  ⏱ 1:30

Pierwsze podejście to szyfrowanie całego woluminu – Full Volume Encryption. Szyfrujemy każdy sektor partycji: system, plik stronicowania, plik hibernacji, pliki tymczasowe, a nawet wolne miejsce. Niczego nie da się zapomnieć zaszyfrować. To jest BitLocker.

Drugie podejście to szyfrowanie na poziomie plików. Wybieramy konkretne pliki lub foldery i tylko one są szyfrowane – za to kluczem przypisanym do konkretnego użytkownika. To jest EFS.

Już teraz widać różnicę filozofii: BitLocker odpowiada na pytanie „co jeśli ktoś ukradnie dysk”, a EFS na pytanie „co jeśli inny użytkownik tego samego komputera zajrzy do moich plików”.

## Slajd 7: BitLocker  ·  ⏱ 0:10

Przejdźmy zatem do BitLockera.

## Slajd 8: BitLocker w pigułce  ·  ⏱ 1:30

BitLocker pojawił się w Windows Vista w 2007 roku. Dziś domyślnie używa trybu AES-XTS ze 128-bitowym kluczem, opcjonalnie 256 bitów. XTS to tryb zaprojektowany specjalnie do szyfrowania sektorów dysku – każdy sektor szyfrowany jest z innym „tweakiem”, więc identyczne dane w różnych miejscach wyglądają inaczej.

Działa przezroczyście: sterownik filtrujący pod systemem plików szyfruje przy zapisie i odszyfrowuje przy odczycie. Użytkownik niczego nie zauważa, a na nowoczesnych procesorach z instrukcjami AES-NI narzut wydajnościowy jest praktycznie niezauważalny.

Pełny BitLocker jest w edycjach Pro, Enterprise i Education. W Home mamy uproszczoną wersję nazwaną „Szyfrowanie urządzenia”, a od Windows 11 24H2 jest ona włączana automatycznie przy czystej instalacji. Do tego BitLocker To Go do szyfrowania pendrive'ów.

## Slajd 9: Hierarchia kluczy  ·  ⏱ 2:00

To jest najważniejszy slajd o BitLockerze. Dane na dysku są szyfrowane kluczem FVEK – Full Volume Encryption Key. On się w zasadzie nigdy nie zmienia.

FVEK jest zaszyfrowany kluczem VMK – Volume Master Key. A VMK jest zaszyfrowany niezależnie przez każdy z tak zwanych protektorów: przez TPM, przez TPM z PIN-em, kluczem na pendrive'ie, 48-cyfrowym hasłem odzyskiwania i tak dalej. Każdy protektor to po prostu inna „kopia” VMK zamknięta innym zamkiem.

Dzięki temu dodanie PIN-u czy zmiana hasła odzyskiwania polega na przeszyfrowaniu kilkudziesięciu bajtów, a nie całego terabajtowego dysku. A szybkie „wymazanie” dysku polega na zniszczeniu kluczy – bez nich dane są bezużyteczne.

## Slajd 10: TPM – sejf na płycie głównej  ·  ⏱ 2:00

Najczęstszy protektor to TPM – Trusted Platform Module. To mały kryptoprocesor: kiedyś osobny układ na płycie, dziś coraz częściej wbudowany w procesor jako firmware TPM. Windows 11 wymaga TPM 2.0.

Podczas rozruchu każdy etap – firmware UEFI, Secure Boot, menedżer rozruchu – jest „mierzony”, czyli jego skrót trafia do rejestrów PCR w TPM. VMK jest „zapieczętowany” w TPM: TPM odda go tylko wtedy, gdy wartości PCR są takie same jak w chwili zapieczętowania.

Efekt: jeśli ktoś wyjmie dysk do innego komputera albo podmieni bootloader, TPM klucza nie wyda. Ale uwaga – w trybie „tylko TPM” klucz jest wydawany automatycznie, bez udziału użytkownika. Zapamiętajcie to, wrócimy do tego przy atakach.

## Slajd 11: Klucz odzyskiwania  ·  ⏱ 1:45

Skoro TPM odmawia wydania klucza przy każdej zmianie w łańcuchu rozruchu, to co się dzieje po aktualizacji BIOS-u, zmianie ustawień Secure Boot albo przełożeniu dysku? Dostajemy ten niebieski ekran: prośbę o 48-cyfrowy klucz odzyskiwania.

I tu jest największe praktyczne ryzyko BitLockera – nie złodziej, tylko my sami. Bez klucza odzyskiwania dane są stracone. Nie ma tylnej furtki, Microsoft też ich nie odzyska.

Dlatego klucz trzeba zapisać: na prywatnym komputerze robi to automatycznie konto Microsoft, w firmie Active Directory albo Entra ID przez Intune. Ewentualnie wydruk w sejfie. Dobra praktyka dla adminów: przed aktualizacją firmware'u wstrzymać BitLockera na jeden restart.

## Slajd 12: BitLocker w praktyce  ·  ⏱ 1:30

Krótko o praktyce. Oprócz graficznego panelu mamy narzędzie wiersza poleceń manage-bde oraz cmdlety PowerShella. Sprawdzamy stan, włączamy szyfrowanie, dodajemy protektory.

Przy włączaniu na nowym dysku warto wybrać opcję „szyfruj tylko zajęte miejsce” – jest dużo szybsza. Na dysku, który był już używany, lepiej zaszyfrować całość, bo w „wolnym” miejscu mogą leżeć pozostałości starych plików.

BitLocker To Go działa także na pendrive'ach z FAT32 i exFAT. Taki pendrive po podłączeniu pyta o hasło – i to jest chyba najprostszy sposób, żeby nie zgubić danych razem z nośnikiem za 20 złotych.

## Slajd 13: Ataki  ·  ⏱ 0:10

Skoro BitLocker jest taki dobry, to zobaczmy, jak się go łamie. A raczej – jak się go obchodzi, bo nikt nie łamie samego AES-a.

## Slajd 14: Nie łamiemy AES-a. Kradniemy klucz.  ·  ⏱ 2:00

Cztery znane klasy ataków. Pierwszy – cold boot, opisany przez badaczy z Princeton w 2008 roku. Pamięć RAM po odcięciu zasilania nie traci danych natychmiast, a schłodzona sprężonym powietrzem trzyma je dłużej. Można przełożyć kości do innej maszyny i wyszukać w zrzucie klucze AES.

Drugi – podsłuch TPM. Jeśli TPM jest osobnym układem, VMK płynie do procesora po magistrali LPC lub SPI otwartym tekstem. W 2024 roku pokazano to na laptopie Lenovo z Raspberry Pi Pico za około 10 dolarów – całość w mniej niż minutę.

Trzeci – DMA: urządzenia na Thunderbolcie czy PCIe mogą czytać pamięć bezpośrednio. Czwarty – bitpixie, pokazany na kongresie 38C3 w grudniu 2024: uruchamia się przez sieć stary, podatny menedżer rozruchu, który zostawia VMK w pamięci.

Wspólny mianownik: we wszystkich przypadkach komputer sam, automatycznie odszyfrowuje VMK. Zapamiętajmy to.

## Slajd 15: TPM sniffing w jednym obrazku  ·  ⏱ 1:00

Ten atak najlepiej podsumowuje ten mem. Wiele osób zakłada, że TPM to magiczna czarna skrzynka i klucz nigdy jej nie opuszcza. Tymczasem w BitLockerze TPM wydaje VMK procesorowi – i jeśli robi to po fizycznej magistrali na płycie, to wystarczy się do niej podpiąć.

Ważne zastrzeżenie: dotyczy to dyskretnych układów TPM. Firmware TPM wbudowany w procesor – Intel PTT, AMD fTPM – nie ma zewnętrznej magistrali do podsłuchania. A protokół TPM 2.0 przewiduje szyfrowanie sesji, tylko BitLocker go w tym miejscu nie używał.

## Slajd 16: Obrona  ·  ⏱ 1:00

Najskuteczniejsza obrona przed wszystkimi czterema atakami jest jedna: PIN przed rozruchem. Wtedy TPM wyda VMK dopiero po podaniu PIN-u – złodziej, który ma sam laptop, nie ma czego podsłuchiwać. TPM dodatkowo chroni PIN przed zgadywaniem, blokując się po kilku błędnych próbach.

Do tego Kernel DMA Protection, czyli IOMMU blokujące niezaufane urządzenia, regularne aktualizacje razem z migracją na nowe certyfikaty Secure Boot z 2023 roku – to łata na bitpixie – oraz używanie hibernacji zamiast uśpienia, bo w uśpieniu klucz siedzi w RAM-ie.

## Slajd 17: Pułapka: „Wstrzymaj ochronę”  ·  ⏱ 0:45

Jeszcze jedna pułapka. Opcja „Wstrzymaj ochronę” nie odszyfrowuje dysku – ale zapisuje na nim VMK w postaci jawnej, tak zwany clear key. Dane formalnie są zaszyfrowane, tylko klucz leży tuż obok. Dla złodzieja to praktycznie to samo, co dysk bez szyfrowania.

Dlatego wstrzymujemy ochronę tylko na czas restartu, najlepiej z parametrem RebootCount, który sam ją przywróci.

## Slajd 18: EFS  ·  ⏱ 0:10

Teraz drugi bohater – EFS. Tu zamiast jednej wielkiej kłódki na cały dysk mamy mnóstwo małych kłódek na poszczególnych plikach – trochę jak na tym moście w Lublanie.

## Slajd 19: EFS w pigułce  ·  ⏱ 1:30

EFS jest starszy od BitLockera – pojawił się w Windows 2000 jako funkcja systemu plików NTFS. Działa wyłącznie na NTFS i szyfruje pojedyncze pliki lub całe foldery – nowe pliki w zaszyfrowanym folderze szyfrują się automatycznie.

Włącza się go banalnie: Właściwości pliku, Zaawansowane, „Szyfruj zawartość”. Zaszyfrowane pliki Eksplorator pokazuje na zielono – to jest ten charakterystyczny znak rozpoznawczy.

Najważniejsza cecha: klucz jest przypisany do konkretnego użytkownika. Dla właściciela plik otwiera się normalnie, a inny użytkownik – nawet administrator zalogowany na swoim koncie – dostaje „Odmowa dostępu”. W edycji Home EFS nie jest dostępny.

## Slajd 20: Jak działa EFS  ·  ⏱ 2:15

EFS używa szyfrowania hybrydowego. Dla każdego pliku losowany jest osobny klucz symetryczny – FEK, File Encryption Key – i nim, AES-em 256, szyfrowana jest zawartość.

Sam FEK jest szyfrowany asymetrycznie, RSA, kluczem publicznym z certyfikatu EFS użytkownika, i zapisywany w specjalnym atrybucie NTFS o nazwie $EFS, w polu DDF. Jeśli skonfigurowano agenta odzyskiwania danych – DRA – to FEK jest dodatkowo szyfrowany jego kluczem publicznym, w polu DRF. Można też dodać kolejnych użytkowników, którzy mają mieć dostęp do pliku.

A gdzie jest klucz prywatny użytkownika? W jego profilu, chroniony przez DPAPI, a DPAPI wyprowadza klucze z hasła logowania. Czyli w praktyce całe EFS stoi na haśle użytkownika. To ma dwie konsekwencje – jedną dobrą, jedną bardzo złą.

## Slajd 21: Reset hasła a EFS  ·  ⏱ 1:00

Dobra konsekwencja: administrator nie może po prostu zresetować komuś hasła, zalogować się na jego konto i przeczytać plików EFS. Reset hasła zrywa łańcuch DPAPI, więc klucz prywatny staje się nieczytelny.

Zła konsekwencja: dokładnie to samo dzieje się, gdy to my zapomnimy hasła i poprosimy admina o reset. Zwykła zmiana hasła, gdy znamy stare, jest bezpieczna. Ale reset – i pliki przepadają.

Ratunek są dwa: kopia certyfikatu z kluczem prywatnym w pliku PFX albo agent odzyskiwania DRA. Na komputerach w domenie DRA jest domyślnie ustawiany, na samodzielnych komputerach – nie ma go wcale.

## Slajd 22: Gdzie EFS przestaje chronić  ·  ⏱ 1:30

EFS chroni plik tylko tak długo, jak leży na NTFS. Skopiujemy go na pendrive z FAT32 – Windows wyświetli ostrzeżenie, ale plik zapisze się jawnie. Wyślemy go mailem albo wrzucimy do chmury – wychodzi wersja odszyfrowana, bo to my, uprawniony użytkownik, ją czytamy.

Do tego EFS nie szyfruje metadanych: nazwy plików, rozmiary, daty są widoczne. Plik o nazwie „zwolnienia_2027.xlsx” sporo zdradza, nawet zaszyfrowany.

I wreszcie: programy tworzą kopie tymczasowe, a system zrzuca pamięć do pliku stronicowania. Te miejsca nie muszą być zaszyfrowane przez EFS. Czyli EFS sam w sobie jest dziurawy jako ochrona przed kradzieżą dysku – i to jest właśnie miejsce dla BitLockera.

## Slajd 23: EFS w praktyce: cipher  ·  ⏱ 1:00

Narzędzie wiersza poleceń dla EFS to cipher. Przełącznik /e szyfruje, /d odszyfrowuje, /c pokazuje, kto ma dostęp do pliku – czyli dla kogo zapisano kopie FEK.

Dwie rzeczy, które warto zrobić od razu po włączeniu EFS: cipher /x, czyli eksport certyfikatu z kluczem prywatnym do pliku PFX zabezpieczonego hasłem – i schować go poza komputerem. Oraz w firmie: cipher /r do wygenerowania certyfikatu agenta odzyskiwania.

Ciekawostka: cipher /w nadpisuje wolne miejsce na dysku, usuwając pozostałości skasowanych plików. Przydaje się także, gdy szyfrowaliśmy już istniejące pliki – EFS zostawia wtedy ich jawne kopie w zwolnionych sektorach.

## Slajd 24: Starcie  ·  ⏱ 0:30

Skoro znamy już obu zawodników, czas na starcie. Wielu adminów stoi przed tym wyborem jak ten pan przed dwoma przyciskami. Zobaczmy, czy to w ogóle jest właściwe pytanie.

## Slajd 25: BitLocker vs. EFS  ·  ⏱ 2:00

Zestawmy to w tabeli. BitLocker szyfruje wszystko, EFS – to, co wskażemy. BitLocker chroni przed kradzieżą sprzętu, EFS przed innymi użytkownikami tego samego systemu.

Najważniejszy wiersz to trzeci: po uruchomieniu systemu BitLocker jest „przezroczysty” – każde konto i każdy proces, łącznie z malware'em, widzi dane jawnie. EFS zostaje zamknięty dla wszystkich poza właścicielem i agentem odzyskiwania.

Za to EFS zostawia jawne pliki tymczasowe, plik stronicowania i nazwy plików, a BitLocker szyfruje wszystko. Każdy ma też swój „koszmar”: przy BitLockerze to zgubiony klucz odzyskiwania, przy EFS – reset hasła bez kopii certyfikatu.

Wniosek nasuwa się sam: one się nie wykluczają. Każde łata dziurę drugiego.

## Slajd 26: Nie „albo”, tylko „i”  ·  ⏱ 1:30

Zasada „defense in depth” – obrona w głąb. BitLocker jako fundament: chroni przed kradzieżą i szyfruje to, czego EFS nie łapie – pagefile, pliki tymczasowe, metadane. EFS jako druga warstwa tam, gdzie z jednego komputera korzysta kilka osób albo gdzie dane mają być niedostępne nawet dla administratora.

Praktyczne scenariusze: laptop – BitLocker z PIN-em. Pendrive – BitLocker To Go. Komputer współdzielony, np. w laboratorium czy recepcji – BitLocker plus EFS na foldery z poufnymi danymi. Serwer – BitLocker, w domenie z Network Unlock, żeby serwer mógł się sam zrestartować w zaufanej sieci.

## Slajd 27: Dobre praktyki  ·  ⏱ 1:30

Dobre praktyki w skrócie. Klucze odzyskiwania BitLockera centralnie – w Active Directory albo Entra ID. TPM z PIN-em na urządzeniach mobilnych. Wymuszenie algorytmu i trybu przez zasady grupy albo Intune.

Dla EFS: agent odzyskiwania i kopia certyfikatu w bezpiecznym miejscu – nie na tym samym dysku. Szyfrujemy nośnik zanim trafią na niego dane – inaczej pozostałości mogą zostać w nieużywanych sektorach, szczególnie na SSD.

I najważniejsze: przetestujcie odzyskiwanie, zanim będzie potrzebne. Klucz odzyskiwania na karteczce przyklejonej do laptopa to nie backup – to instrukcja obsługi dla złodzieja.

## Slajd 28: Co dalej?  ·  ⏱ 1:15

Kilka słów o tym, dokąd to zmierza. Od Windows 11 24H2 szyfrowanie urządzenia włącza się automatycznie przy czystej instalacji, także w edycji Home – czyli coraz więcej z nas ma BitLockera, nawet o tym nie wiedząc. Tym ważniejsze jest sprawdzenie, gdzie jest nasz klucz odzyskiwania.

Następcą idei EFS jest Personal Data Encryption: szyfrowanie plików, w którym klucze są powiązane z Windows Hello for Business, a dane są niedostępne, dopóki użytkownik się nie zaloguje.

Ciekawostka: w 2018 roku badacze z Uniwersytetu Radboud pokazali, że wiele dysków SSD z „sprzętowym” szyfrowaniem miało poważne błędy. Od tego czasu BitLocker domyślnie szyfruje programowo, zamiast ufać firmware'owi dysku. A dla porównania – w innych systemach odpowiednikami są LUKS w Linuksie, FileVault w macOS i wieloplatformowy VeraCrypt.

## Slajd 29: Podsumowanie  ·  ⏱ 1:15

Podsumowując – i rozwijając mózg krok po kroku. Brak szyfrowania to poziom zero. Hasło do Windowsa daje tylko złudzenie bezpieczeństwa. BitLocker z TPM to już realna ochrona przed kradzieżą. A pełna oświecona wersja to BitLocker z PIN-em, EFS tam, gdzie trzeba, i klucze odzyskiwania przechowywane centralnie.

Trzy zdania do zapamiętania: BitLocker chroni przed kimś, kto ma nasz sprzęt. EFS chroni przed kimś, kto ma konto na naszym komputerze. A szyfrowanie jest dokładnie tak dobre, jak zarządzanie kluczami – bo AES-a nikt nie łamie, za to klucze gubimy wszyscy.

## Slajd 30: Dziękuję! Pytania?  ·  ⏱ Q&A

Dziękuję za uwagę. Mam nadzieję, że nikt nie zareaguje jak ta małpka – chętnie odpowiem na pytania.

Przykładowe pytania, które mogą paść: „Czy BitLocker spowalnia SSD?” – przy AES-NI praktycznie nie. „Czy Microsoft ma mój klucz?” – jeśli logujesz się kontem Microsoft i masz szyfrowanie urządzenia, kopia klucza odzyskiwania jest na Twoim koncie; można ją usunąć i przechowywać samodzielnie. „Czy można używać EFS bez BitLockera?” – można, ale pagefile, pliki tymczasowe i nazwy plików zostają jawne.

## Slajd 31: Źródła  ·  ⏱ —

Slajd ze źródłami – zostawiam do wglądu.
