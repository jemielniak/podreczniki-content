# podreczniki-content - repo treści (GitHub Pages)

Publiczne repozytorium z gotową treścią aplikacji "Podręczniki". Aplikacja
pobiera stąd katalog i skompilowane paczki podręczników. GitHub Pages musi
być włączone (Settings → Pages → Deploy from branch → `main`, folder
`/ (root)`).

Zawartość tego repozytorium jest GENEROWANA - nie edytuje się jej ręcznie.
Powstaje z repozytorium źródeł `jemielniak/podreczniki` przez polecenie
publikacji (`node scripts/publish-and-push.mjs` w projekcie aplikacji),
które sprawdza treść, kompiluje ją i wypycha wynik tutaj. Ręczna zmiana
`catalog.json`, `bundles/` albo `img/` rozjedzie się z hashami paczek
i zepsuje działające podręczniki.

Adres bazowy po włączeniu Pages:

```
https://jemielniak.github.io/podreczniki-content
```

Ten adres jest wpisany w aplikacji w jednym miejscu: `src/config.js` →
`CONTENT_BASE_URL`.

Najważniejsza zasada tego repo: dodanie i aktualizacja podręcznika NIE
dotyka aplikacji. Żadnego rebuilda, żadnej submisji do sklepów - publikacja
wypycha zmianę tutaj, a urządzenia same ją zobaczą przy następnym starcie.

## Układ plików

```
catalog.json                 katalog: klasy, przedmioty, podręczniki (manifest)
bundles/<id>.json            paczka treści jednego podręcznika
img/<id>/cover.png           okładka podręcznika (pokazywana na liście)
img/<id>/...                 obrazki treści (dokładnie te z pola "images" paczki)
```

Źródła, z których to powstaje (pliki rozdziałów, surowe obrazki, plansze),
mieszkają w osobnym repozytorium `jemielniak/podreczniki` i tam pracuje
wykonawca treści.
