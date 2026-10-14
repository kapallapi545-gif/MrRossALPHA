# MrRossALPHA launcher

Launcher do **jednych konkretnych buildów** Fortnite ze złotych prywatnych serwerów,
z multiplayerem i własnym serwerem. C# / WPF na .NET 8.

Zbudowany na bazie MrRoss Launcher v3 — logika uruchamiania klienta, injektora i backendu
jest sprawdzona i przeniesiona bez zmian w sensie działania. Wersja 4 to nowy interfejs,
automatyczne wykrywanie buildów i zakładka serwerów. Patrz `LICENSE`.

---

## Co launcher robi sam, bez pytania

1. **Znajduje Twoje buildy.** Skanuje `E:\Pobrane\3.6-CL-4019403\Fortnite` w pierwszej
   kolejności, potem `Pobrane`, `Downloads`, `Download`, `Fortnite`, `OGFN` na dyskach
   C–F. Build, który jest już w tym folderze, importuje się sam — nie ma kreatora
   „wybierz folder", bo to launcher dla jednego zestawu buildów, nie przeglądarka bibliotek.
2. **Podłącza patchery, które już masz na dysku.** Jeśli istnieje folder DLL z
   ATLAS Link (`E:\Pobrane\ATLAS-Link-2.0.7\...\assets\dlls`), launcher ustawia go i
   nakłada pierwszy kompletny zestaw (domyślnie *ATLAS default*: Tellurium + Magnesium).
   Bez tego krok klient zamyka się podczas logowania.
3. **Podłącza backendy z lokalnych archiwów.** LawinServer i Neonite V2 instalują się
   z `E:\Pobrane\ATLAS-Link-2.0.7\backends\*.zip`, więc pierwsze uruchomienie nie
   potrzebuje internetu.
4. **Sprawdza, czy serwery odpowiadają** i pokazuje ping.

Ustawienia, które kiedyś ustawiłeś ręcznie, nigdy nie są nadpisywane — powyższe działa
tylko na pustych polach i tylko dla adresów, które nadal wskazują na domyślne.

---

## Zakładki

| Zakładka | Co robi |
|---|---|
| **Start** | Wybrany build, duży przycisk *Graj*, gotowość launchera, rozpoznany problem z ostatniego uruchomienia, skrót do serwerów. |
| **Buildy** | Wykryte buildy: wybór, ulubione, zmiana nazwy, otwarcie folderu, log gry, wykrywanie nowych. |
| **Serwery** | Lista serwerów z pingiem, ulubionymi, filtrem online, dołączaniem, własnym backendem i listą z sieci. |
| **Ustawienia** | Profil, patchery, multiplayer, uruchamianie, foldery buildów. |
| **Logi** | Log launchera, rozpoznana przyczyna ostatniego niepowodzenia, kopiowanie raportu. |

---

## Multiplayer

**Serwery, do których możesz dołączyć.** Wpis ręczny albo wklejony plik JSON z listą
(`[{"name":"...","host":"...","port":3551}]`, albo `{"servers":[...]}`). Serwer z
pustym polem wersji pasuje do każdego builda; wersja używa składni z katalogu modów:
`3.6`, `3.0, 3.6, 4.1`, `3.0-4.1`. Serwery dla innej wersji są ukrywane, bo nie da się na
nich zagrać — w Ustawieniach można je pokazać. „Graj" uruchamia klienta od razu na
wybranym serwerze.

**Własny serwer.** Zakładka Serwery → Moj serwer: zainstaluj, uruchom i zatrzymaj backend
LawinServer lub Neonite V2. Uruchamiany jest jego własnym interpreterem Node z archiwum,
więc Node nie musi być zainstalowany w systemie. Port trafia do procesu przez zmienną
środowiskową `PORT`, bo te backendy czytają `process.env.PORT` i ignorują argumenty
wiersza poleceń.

**Hostowanie meczów.** Przycisk *Hostuj mecz* na stronie Start startuje proces klienta w
trybie serwera (`-nosplash -nosound`, opcjonalnie `-nullrhi`). Wymaga **Game Server DLL**
z Ustawień: backend obsługuje konta i lobby, ale meczów nie prowadzi.

**Kilku graczy naraz.** *Wiele klientów* uruchamia dodatkowe procesy z tego samego builda,
każdy z własną nazwą, w ilości ustawionej w Ustawieniach.

**Gra ze znajomymi (zakładka Multiplayer).** Jedna osoba hostuje, reszta dołącza po adresie IP.

- *Host:* wybiera adres swojego komputera widoczny dla znajomych (VPN: Radmin, ZeroTier,
  Hamachi, Tailscale albo LAN) i zapisuje go. Adres trafia do pliku
  `%APPDATA%\RossLauncher\multiplayer-host.txt`, z którego korzysta `start-atlas-backend.cmd`
  (backend podaje klientom ten adres jako matchmaker i serwer gry). Potem *Uruchom backend*
  i *Hostuj mecz*. Na komputerze hosta trzeba otworzyć TCP 3551 i 5555 oraz UDP 7777
  (przycisk *Kopiuj komendy zapory*).
- *Znajomy:* wpisuje adres hosta i klika *Połącz i graj*. Patcher logowania ma na stałe
  adres `127.0.0.1:3551`, więc launcher przekierowuje ten lokalny port na backend hosta i
  dopiero potem uruchamia klienta. Na tym komputerze nie może działać własny backend.
- Każdy gracz potrzebuje innej nazwy (Multiplayer → *Twoja nazwa gracza*) i tego samego
  builda gry.

**Znajomi (4.3).** Karta *Znajomi* na stronie Multiplayer trzyma listę (`friends.json`):
nazwa i adres komputera. *Dołącz* wypełnia pole adresu hosta i uruchamia ten sam przebieg
co *Połącz i graj* (sprawdzenie, przekierowanie, klient). Kropka przy znajomym pokazuje, czy
jego backend odpowiada (sprawdzenie co ~30 s, gdy strona jest otwarta). Znajomych dodaje się
ręcznie albo z **kodu** (`RL1:...`): *Kopiuj mój kod* daje kod z Twoją nazwą i zapisanym
adresem hosta, *Dodaj z kodu* przyjmuje też `nazwa@adres` i sam adres.

**Powrót do meczu (4.3).** Host Reboota nie odrzuca już graczy komunikatem „Match has
started”. Gracz, który wyszedł z rundy i wchodzi znowu (albo znajomy dołączający w trakcie),
trafia do trwającego meczu w powietrzu nad wyspą i zaczyna od kilofa i narzędzi budowania.
Nowa runda od zera wymaga ponownego *Hostuj mecz* — Reboot nie umie zrestartować meczu
w Rozdziale 2 i późniejszych.

### Serwer gry w paczce (4.2)

Instalator zawiera zbudowany `Dlls\ProjectReboot3.dll` (Project Reboot 3.0, host meczów
dla S20+, licencja BSD-3 w `reboot\`). Gdy w Ustawieniach pole *Game Server DLL* jest
puste albo wskazuje brakujący plik, launcher ustawia ten plik sam. Poprawki względem
oryginalnego źródła (zbieranie przedmiotów, materiały z kopania, skin z lobby, łup ze
skrzynek) leżą jako `reboot\reboot-22.30-host-fixes.patch`.

`start-atlas-backend.cmd` przy każdym starcie dopisuje (raz) do plików hotfix backendu
ATLAS fragmenty z `atlas-hotfixes\` — przełączniki wyłączające content beacon, pytanie o
inicjalizację serwera i szyfrowanie. Bez nich klient po dołączeniu do meczu na hoście
Reboota nie przechodzi ekranu ładowania. Po pierwszej zmianie trzeba zrestartować klienta
i hosta, bo oba pobierają hotfix przy logowaniu.

### Uwaga o dołączaniu do serwera

Przełącznik, który kieruje klienta na konkretny serwer, **różni się między buildami**
prywatnych serwerów. Dlatego jest edytowalny: Ustawienia → Multiplayer → *Argument
dołączania*, domyślnie `-ServerAddr={host}:{port}`. Dostępne znaczniki: `{host}`,
`{port}`, `{name}`.

Jeśli „Graj" uruchamia klienta, ale nie łączy go z serwerem — to najpewniej zła nazwa
przełącznika dla tego buildu, a nie niedostępny serwer. Ping w liscie mówi wprost, czy
serwer odpowiada.

---

## Status w Discordzie

Karta w Discordzie pokazuje, co teraz robisz. Pierwsza linia to zawsze **MrRosslauncher**
(Ustawienia → Discord → *Nazwa w Discordzie*), pod nią są dwie linie:

| Sytuacja | Linia 2 | Linia 3 |
|---|---|---|
| Launcher, strona Start | W launcherze | Gotowy do gry: Fortnite 22.30 |
| Launcher, inna strona | Przegląda buildy / Zarządza serwerami / W zakładce Multiplayer / Zmienia ustawienia / Czyta logi | np. liczba buildów |
| Gra się uruchamia | Uruchamia grę | Fortnite 22.30 |
| Klient w lobby | W lobby | Fortnite 22.30 · serwer: Kamil |
| Szukanie meczu | Szuka meczu | jw. |
| Łączenie z serwerem | Łączy z serwerem | jw. |
| W meczu | W meczu | Fortnite 22.30 · serwer: Kamil · graczy: 3 |
| Własny host | Hostuje mecz | Fortnite 22.30 · graczy: 2 |

Gdzie jest gra, launcher czyta z konsoli procesu (menu, wyszukiwanie, „Welcomed by server",
połączenia). Adresów IP nie ma na karcie nigdy: znajomy bez nazwy jest pokazany jako
„serwer znajomego", a całe „gdzie" wyłącza opcja *Pokazuj serwer w statusie*.

Pierwsza linia jest wysyłana jako własna nazwa aktywności, więc nie zależy od tego, jak
nazywa się aplikacja Discord o podanym *ID aplikacji*. Obrazek można podać jako adres
(`https://...`) — Discord pobiera go sam, bez wgrywania grafik.

---

## Zanim gra się nie uruchomi

Launcher sprawdza to przed każdym startem i pokazuje problem jako zdanie z instrukcją,
zamiast pozwalać klientowi wyłożyć się w raporterze błędów:

1. **Auth patcher musi być ustawiony** (Ustawienia → Patchery). Bez niego klient zamyka się
   podczas logowania.
2. **Backend musi nasłuchiwać na `127.0.0.1:3551`** — to endpoint, wobec którego klient się
   uwierzytelnia. Bez niego klient nie dostaje sesji i nie ma czego ładować.

Są to jedyne dwa twarde wymagania. Do hostowania dochodzi trzecie: *Game Server DLL*,
bo backend sam mecze nie prowadzi. Brak patchera przy hostowaniu jest cichy — klient
zamarza w momencie startu serwera — dlatego jest sprawdzany osobno.

---

## Uruchamianie

```powershell
.\launch.cmd            # uruchom w Debug
.\launch.cmd test       # testy jednostkowe
.\launch.cmd build      # publikacja self-contained do dist\app
.\launch.cmd clean      # usuń bin, obj i dist
.\launch.cmd installer  # pełny instalator (wymaga Inno Setup 6)
```

Albo bezpośrednio:

```powershell
dotnet build RossLauncher.slnx
dotnet test tests\RossLauncher.Core.Tests
powershell -File build\publish.ps1 -Clean
```

Rozwiązanie to `RossLauncher.slnx` (format XML ze SDK .NET 9+). Na starszym SDK użyj
plików projektów albo pozwól `launch.cmd` wybrać ten, który istnieje.

**Wymagania:** Windows 10 1809 lub nowszy, x64, [.NET 8 SDK](https://dotnet.microsoft.com/download/dotnet/8.0)
do budowania. Opublikowana wersja nie wymaga runtime'u — instalator dostarcza pojedynczy
plik `.exe`.

---

## Struktura projektu

```
MrRossLauncherv4/
|-- src/
|   |-- RossLauncher.Core/        net8.0-windows, cała logika, bez UI
|   |   |-- Models/               BuildEntry, ServerEntry, LauncherSettings, backendy
|   |   |-- Services/             Library, Launch, Injector, Mods, Backend, Servers,
|   |   |                         Discovery, LocalResources, Discord, diagnostyka
|   |   |-- Native/               punkty wejścia Win32 potrzebne launcherowi
|   |   `-- Storage/              ścieżki, atomowy zapis JSON, logowanie
|   `-- RossLauncher.App/         net8.0-windows, aplikacja WPF
|       |-- Themes/Dark.xaml      cały język wizualny
|       |-- ViewModels/           po jednym na zakładkę, własny MVVM
|       `-- Views/                po jednym UserControl na zakładkę + okna dialogowe
|-- tests/RossLauncher.Core.Tests/
|-- assets/
|   |-- catalog.json              przykładowy katalog modów
|   `-- servers.json              domyślna lista serwerów (ładowana przy pierwszym uruchomieniu)
|-- bundled/                      dokładana do instalatora obok .exe
|   |-- Dlls/ProjectReboot3.dll   host meczów (Project Reboot 3.0, BSD-3)
|   |-- atlas-hotfixes/           fragmenty hotfixów + skrypt dopisujący je backendowi
|   `-- reboot/                   licencja i patch źródła DLL-a
|-- build/publish.ps1             publikacja single file
|-- installer/RossLauncher.iss    skrypt Inno Setup
|-- launch.cmd
`-- build-installer.cmd
```

`Core` nie odwołuje się do WPF, więc logika uruchamiania, wstrzykiwania i serwerów
testuje się bez interfejsu.

---

## Gdzie są Twoje dane

`%APPDATA%\RossLauncher`

```
library.json      wykryte buildy
settings.json     wszystkie ustawienia
servers.json      lista serwerów, ulubione, notatki
friends.json      znajomi: nazwa i adres komputera
backends.json     backendy i ich foldery
dll-library.json  zapisane DLL-e
mods-state.json   które mody są gdzie zainstalowane
playtime.json     czas gry per build
Logs/             launcher-yyyy-MM-dd.log
Assets/           DLL-e, mody, backendy, awatary
```

Zmienna środowiskowa `ROSSLAUNCHER_DATA` przenosi cały folder w inne miejsce, np. obok
pliku `.exe` przy instalacji przenośnej. Odinstalowanie zostawia ten folder nietknięty.

---

## Testy

```powershell
.\launch.cmd test
```

335 testów jednostkowych: skanowanie buildów, filtrowanie listy serwerów, zgodność
wersji, argument dołączania, formaty odpowiedzi JSON z listą serwerów, lista znajomych i
ich kody, instalacja i start backendu oraz logika uruchamiania OGFN.

Test integracyjny backendu (rozpakowuje ok. 50 MB i uruchamia serwer) jest domyślnie
pominięty:

```powershell
$env:ROSSLAUNCHER_RUN_REAL_BACKEND_TEST = "1"
dotnet test tests\RossLauncher.Core.Tests --filter RealBackendSmokeTests
```

Archiwum bierze z lokalnego folderu ATLAS, jeśli go nie ma — wtedy z sieci.

---

## Podziękowania

- [ATLAS Link](https://github.com/cipherfps/ATLAS-Link) — cipherfps
- [Reboot Launcher](https://github.com/Auties00/Reboot-Launcher) — Auties00
- [LawinServer](https://github.com/Lawin0129/LawinServer) — Lawin0129
- [Project Reboot 3.0](https://github.com/Milxnor/Project-Reboot-3.0) — Milxnor (host meczów w paczce)

Fortnite jest znakiem towarowym Epic Games, Inc. Ten projekt nie jest powiązany z ani
zatwierdzony przez Epic Games. Uruchamia wyłącznie buildy już obecne na tym komputerze.
