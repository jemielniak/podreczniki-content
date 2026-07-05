# podreczniki-content - repo treści (GitHub Pages)

Publiczne repo z treścią aplikacji "Podręczniki". Aplikacja pobiera stąd
katalog i paczki podręczników. Zawartość tego folderu przenieś 1:1 do
osobnego repozytorium i włącz GitHub Pages (Settings → Pages → Deploy from
branch → `main`, folder `/ (root)`).

Po włączeniu Pages adres bazowy to zwykle:

```
https://<twoja-nazwa>.github.io/podreczniki-content
```

Ten adres wpisujesz w aplikacji w JEDNYM miejscu:
`src/config.js` → `CONTENT_BASE_URL`.

**Najważniejsza zasada tego repo: dodanie i aktualizacja podręcznika NIE
dotyka aplikacji.** Żadnego rebuilda, żadnej submisji do sklepów - commit
tutaj wystarczy, urządzenia same zobaczą zmianę.

## Układ plików

```
catalog.json              katalog: klasy, przedmioty, podręczniki (manifest)
bundles/<id>.json         paczka treści jednego podręcznika
img/<id>/cover.png        okładka podręcznika (pokazywana na liście)
img/<id>/...              obrazki treści (dokładnie te z pola "images" paczki)
```

Źródła autorskie (rozdziały w JS/JSON) NIE leżą tutaj - mieszkają w repo
aplikacji w drzewie `content-src/<klasa>/<przedmiot>/<id>/` (instrukcja
dla autora treści: `content-src/README.md`; kompletny przykład:
`content-src/sp7/chemia/_chemia-7-przyklad/`; specyfikacja 10 typów
ćwiczeń: `src/chapters/_SZABLON_ROZDZIALU.js`). Tutaj trafia wyłącznie
wynik publikacji.

### Wizualizacje (od Fazy 9)

Infografiki i mapy pojęć rozdziałów leżą w `img/<id>/visuals/<rozdział>/`:

```
img/historia-5/visuals/r01/01_czas_i_epoki.jpg
```

Skrypty publikacji kopiują je tam same z folderu `visuals/` w źródłach
podręcznika i wpisują metadane do rozdziałów paczki (`chapter.visuals`),
więc suma kontrolna treści je obejmuje. Walidator sprawdza istnienie każdego
pliku z paczki na dysku (`validate-bundle.mjs --img-dir ...`).

## Jak dodać NOWY podręcznik - przepis od zera

1. **Przedmiot.** Jeśli to pierwszy podręcznik nowego przedmiotu, dopisz
   go jedną linijką do rejestru `content-src/_przedmioty.json` w repo
   aplikacji: `{ "id": "chemia", "label": "Chemia", "icon": "⚗️" }`.
   Do katalogu trafi automatycznie przy pierwszej publikacji podręcznika,
   który go używa. Klasy (sp5-sp8, lo1-lo4) są stałe.
2. **Folder w drzewie.** `content-src/<klasa>/<przedmiot>/<id>/` -
   położenie mówi wszystko: klasa i przedmiot biorą się ze ścieżki, id
   z nazwy folderu (małe litery, cyfry, myślniki; po publikacji
   niezmienialne). Szkielet założy komenda
   `node scripts/new-companion.mjs --level sp7 --subject chemia --id <id>
   --title "..."` (tworzy folder-SZKIC z podkreślnikiem), albo skopiuj
   przykład `content-src/sp7/chemia/_chemia-7-przyklad/`.
3. **`_bookConfig.json`**: minimum `{ "title": "..." }`; opcjonalnie
   `icon`, `publisher`, `subtitle`. Reszta (entitlementy, darmowy rozdział,
   okładka, adres paczki) jest WYPROWADZANA z położenia: konwencja
   `companion_<id z podkreśleniami>`, `freeFirstChapter` = true poza
   repetytoriami. Wyjątki nadpisuje opcjonalny `_catalogEntry.json`
   (np. wspólny entitlement dla dwóch wydań).
4. **RevenueCat PRZED publikacją**: entitlement + offering o nazwie
   z konwencji oraz produkty w sklepach (checklista: BUILD_PROGRESS.md,
   Fazy 4 i 7).
5. **Rozdziały**: `rozdzialXX.js` albo `rozdzialXX.json` wg
   `_SZABLON_ROZDZIALU.js`. **Obrazki**: do `img/` w folderze podręcznika,
   okładka `cover.png` w folderze podręcznika (albo w `img/`).
6. **Repetytoria**: folder `repetytorium` w klasach egzaminacyjnych
   (`sp7`, `sp8`, `lo3`, `lo4`) - w aplikacji pojawiają się na złotej
   karcie zbiorczej "Repetytorium" na ekranie przedmiotów tej klasy,
   bez darmowego rozdziału. Zero konfiguracji.
7. **Publikacja**: patrz niżej. Foldery z podkreślnikiem na początku to
   szkice - pipeline je pomija do czasu zmiany nazwy.

## Publikacja (nowe podręczniki i aktualizacje razem)

W repo aplikacji:

```
node scripts/publish-all.mjs --content <ścieżka-do-tego-repo>
```

Skrypt skanuje całe drzewo `content-src/`, waliduje KAŻDY podręcznik
(schemat 10 typów, sekcje, unikalne id, obrazki), publikuje wyłącznie te
ze zmienioną treścią (porównanie hashem), sam nadaje numery wersji
(opublikowana + 1), dopisuje brakujące przedmioty z rejestru, podbija
`version` manifestu o 1 i wypisuje kroki `git add/commit/push` dla
zmienionych plików. Zasada wszystko-albo-nic: jeden błąd gdziekolwiek
w drzewie = zero zapisów.

Warianty: `--dry-run` (sam plan), `--only <id>` (jeden podręcznik),
`--force` (publikuj mimo niezmienionego hasha), `--allow-missing-images`,
`--ascii`.

Pojedynczy katalog źródeł SPOZA drzewa publikuje po staremu
`node scripts/publish-companion.mjs --src <katalog> --content <repo>`
(wersje z `_bookConfig`/`--bump`, wpis katalogowy z `_catalogEntry.json`).

Ręczna publikacja (fallback, bez skryptów): `node scripts/build-bundle.mjs
--out <repo>/bundles/<id>.json`, obrazki skopiuj sam, w `catalog.json`
podbij `bundleVersion`, wklej `contentHash` paczki jako `bundleHash`,
zaktualizuj `chapterCount` i podbij `version` manifestu. Potem OBOWIĄZKOWO
walidator (niżej).

## Walidacja - zepsuta paczka nie ma prawa się opublikować

```
node scripts/validate-bundle.mjs bundles/<id>.json \
     --catalog catalog.json --img-dir img/<id>
```

Sprawdza m.in.: schemat każdego z 10 typów ćwiczeń (pola, typy i zakresy
odpowiedzi, liczba luk `fill_in`, odpowiedź `odd_one_out` wśród elementów
pytania, zgodność multizbiorów `sequence`, spójność `match` i `sort`),
unikalność id, sekcje w `sectionOrder`, gołe nazwy obrazków i ich obecność
na dysku, przeliczony `contentHash` oraz zgodność wpisu w katalogu
(wersja, hash, `chapterCount`, reguła repetytorium, entitlementy).
Kod wyjścia 0 = czysto; 1 = NIE publikować. `--strict` blokuje też przy
ostrzeżeniach.

`publish-companion.mjs` uruchamia te same kontrole automatycznie (przed
zapisem i ponownie na tym, co faktycznie wylądowało na dysku), więc
samodzielny walidator jest potrzebny głównie po ręcznych edycjach i w razie
wątpliwości.

### Checklista przed `git push`

1. `node scripts/validate-bundle.mjs bundles/<id>.json --catalog catalog.json --img-dir img/<id>` → `✓ CZYSTA`.
2. `git status`: zmienione tylko `catalog.json`, `bundles/<id>.json`, `img/<id>/`.
3. Nowy podręcznik: entitlement + offering + produkty istnieją w RevenueCat
   i sklepach (inaczej paywall nie ma czego sprzedać).
4. `version` manifestu podbite (skrypt robi to sam).
5. Push i pętla kontrolna niżej.

## Zasady katalogu

- `levels.stage`: `"sp"` (klasy 5-8) albo `"lo"` (klasy 1-4). Aplikacja
  grupuje karty klas po tym polu.
- **Każdy companion z `subjectId: "repetytorium"` MUSI mieć
  `freeFirstChapter: false`** - repetytoria nie mają darmowego rozdziału.
  Aplikacja wymusza tę regułę po swojej stronie, walidator ją egzekwuje,
  a katalog ma mówić prawdę.
- Pozostałe podręczniki: `freeFirstChapter: true` = rozdział 1 gratis
  (standard). Rozdział jest odblokowany, gdy: (indeks 0 i podręcznik ma
  darmowy R1) LUB aktywny entitlement podręcznika LUB aktywny `all_access`.
- `entitlementId` / `offeringId`: identyfikatory RevenueCat, konwencja
  `companion_<id z podkreśleniami>` (np. `companion_chemia_7_nowa_era`).
  Aplikacja CZYTA JE Z KATALOGU: `entitlementId` decyduje o zamkach,
  `offeringId` wybiera paywall. Dla płatnej treści oba pola OBOWIĄZKOWE.
- `cover` i `bundleUrl` mogą być względne (od adresu bazowego) albo pełnymi
  URL-ami `https://...`.
- `version` manifestu rośnie przy każdej zmianie katalogu (skrypt podbija sam).

## Konwencje treści

- **Id ćwiczeń**: styl `R04_BIZ_01` - rozdział + skrót sekcji + numer.
  Unikalne w obrębie całej paczki (tooling to egzekwuje); między
  podręcznikami najbezpieczniej dokładać prefiks książki (przykład używa
  `C7_R01_SUB_01`), żeby id były unikalne w całej aplikacji.
- **Nazwy obrazków**: pole `image` i lista `images` to GOŁE nazwy plików -
  bez ścieżek, bez `img/`. Wielkość liter musi się zgadzać 1:1 z plikiem
  (GitHub Pages rozróżnia, Windows nie - walidator łapie kolizje). Zalecany
  wzorzec dla nowych treści: `NN_Tytul_z_podkresleniami.png` albo
  `rNN_nazwa_z_podkresleniami.png` (tak nazwane są istniejące obrazki
  historii). Bez spacji; polskie znaki technicznie przechodzą, ale walidator
  ostrzega - między systemami plików bywają kruche.
- **Sekcje**: każde ćwiczenie musi mieć sekcję z `sectionOrder` rozdziału
  (przy publikacji to błąd, nie ostrzeżenie - ćwiczenie spoza listy nie
  pojawiłoby się w menu). Sekcja `Super trudne` jest specjalna: pytania
  wchodzą tylko na życzenie ucznia.

## Pętla kontrolna po publikacji (bez rebuilda aplikacji)

Po `git push` odczekaj 1-2 minuty na GitHub Pages, potem na urządzeniu
z zainstalowaną aplikacją (ta sama binarka, żadnych zmian w kodzie):

1. **Katalog dojechał**: ZAMKNIJ aplikację całkiem (swipe z listy ostatnich)
   i uruchom ponownie - katalog jest pobierany świeżo przy starcie ekranu
   klas. Nowy podręcznik widać na ścieżce klasa → przedmiot; przy pierwszym
   dodaniu przedmiotu pojawia się też jego karta. Szybki test bez
   urządzenia: otwórz `https://<pages>/catalog.json` w przeglądarce
   i sprawdź `version` oraz wpis.
2. **Plakietki**: nowy podręcznik ma ☁ „Do pobrania" i 🎁 „Rozdział 1
   gratis" (repetytorium: 🔒 „Pełny dostęp płatny"). Okładka z
   `img/<id>/cover.png`; brak pliku = ikona przedmiotu (fallback).
3. **Pobranie i weryfikacja**: wejdź w podręcznik - ekran pobierania,
   pasek postępu obrazków, potem lista rozdziałów. To dowód, że
   `bundleHash` w katalogu zgadza się z paczką (przy niezgodności aplikacja
   ODRZUCA pobranie i pokazuje błąd weryfikacji).
4. **Bramka płatności**: rozdział 1 otwiera się za darmo (poza
   repetytorium), rozdziały 2+ mają 🔒, a tapnięcie pokazuje paywall
   z produktem podręcznika i upsellem `all_access`. Po zakupie testowym
   (konto licencji testowej / sandbox) zamki znikają bez restartu;
   „Przywróć zakupy" w ⚙ też je zdejmuje.
5. **Offline**: włącz tryb samolotowy i otwórz pobrany podręcznik -
   działa z cache'u razem z obrazkami.
6. **Aktualizacja**: podbij treść (`--bump`, push), na urządzeniu zamknij
   i otwórz aplikację - karta dostaje ⟳ „Aktualizacja", wejście pobiera
   nową wersję i sprząta starą. Postępy ucznia zostają (klucze localStorage
   są niezależne od wersji paczki).
7. Coś nie gra? `Pobrane podręczniki` (stopka ekranu klas) pokazuje wersję
   i rozmiar zapisanej paczki; usunięcie i ponowne wejście wymusza świeże
   pobranie.

## Jak aplikacja weryfikuje paczki (mechanika)

1. Przy otwarciu podręcznika aplikacja porównuje `bundleVersion` +
   `bundleHash` z manifestu z tym, co ma zapisane. Zgodne = otwiera
   z cache'u bez sieci. Różne = pobiera `bundleUrl`, liczy sha256 z
   `JSON.stringify({bookConfig, chapters, images})` sparsowanej paczki
   i porównuje z `bundleHash`. Niezgodność = paczka odrzucona.
2. Obrazki z listy `images` są pobierane od razu po paczce, spod
   `img/<id>/<nazwa>`. Brak pojedynczego pliku nie blokuje podręcznika
   (aplikacja dociągnie go przy następnym otwarciu), ale docelowo wszystkie
   pliki z `images` muszą leżeć w repo.
3. Stare wersje na urządzeniach sprzątają się same przy pierwszym otwarciu
   po aktualizacji.
4. GitHub Pages serwuje pliki z nagłówkami CORS `*`, a aplikacja i tak
   pobiera przez natywny CapacitorHttp, więc webview nie blokuje żądań.

## Paczka testowa

`bundles/biologia-5.json` to mała, poprawna paczka (1 rozdział, 6 ćwiczeń,
1 obrazek `img/biologia-5/b5_test_lupa.png`) służąca wyłącznie do testowania
pobierania end-to-end. Jej `bundleHash` w `catalog.json` jest prawdziwy.
Przed publikacją produkcyjną podmień ją na prawdziwą treść albo usuń wpis
z katalogu. Paczka `bundles/historia-5.json` to kopia paczki przykładowej
wbudowanej w aplikację.
