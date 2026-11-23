# Laboratorium 1. Podstawy pracy z agentem

**Repozytorium GitHub** — [szweyski/MSI-laboratoria](https://github.com/szweyski/MSI-laboratoria)

**Aplikacja i ustawienia** — Codex Desktop · `gpt-6.2-sol` · `xhigh` (według zapisu sesji)

**Końcowy raport** — [exercise-summary.json](project/results/exercise-summary.json) · [raport na GitHubie](https://github.com/szweyski/MSI-laboratoria/blob/main/lab1/project/results/exercise-summary.json)

## 1. Co zleciłem agentowi i w jakich granicach?

Zleciłem przygotowanie raportu z `project/data/exercise.csv` w `project/results/exercise-summary.json` oraz uruchomienie gotowego testu. W pierwszym etapie kod, dane i testy miały pozostać bez zmian. W drugim zezwoliłem na zmianę jednej etykiety i, dopiero po zapisaniu mojego przewidywania, ponowne uruchomienie skryptu.

Poprosiłem o pokazanie narzędzia, poleceń i wyników procesów, podanie środowiska wykonania, odczyt raportu, potwierdzenie pliku wejściowego oraz podanie wyniku testu i zgłoszenie ewentualnych błędów.

## 2. Jak powstał raport?

Polecenia wykonał mój komputer z Windows w PowerShellu, z katalogu `project/`. `Get-Location` potwierdził katalog, a `py -3 --version` zwróciło `Python 3.14.0`. Model wybrał polecenie, Codex Desktop wywołał narzędzie terminalowe `exec_command`, lokalny Python wykonał skrypt, a wynik wrócił do modelu.

```powershell
py -3 describe_data.py --data data/exercise.csv --out results/exercise-summary.json
```

Przed pierwszym uruchomieniem pliku raportu jeszcze nie było. Proces zakończył się kodem `0`, wypisał `Report saved:` ze ścieżką do raportu oraz `Messages: 8; labels: {'ham': 5, 'spam': 3}`. Agent odczytał JSON przez `Get-Content` i sprawdził, że `input_file` wskazywało na podany CSV, a `input_sha256` zgadzało się z wynikiem `Get-FileHash`.

Każdy rekord CSV odpowiada jednej wiadomości, nawet gdy `text` zajmuje kilka linii. `message_id` identyfikuje wiadomość, `text` zawiera temat i treść, a `label` podaje jej etykietę. W końcowym raporcie podlinkowanym u góry są 4 wiadomości `ham` (zwykła poczta) i 4 `spam` (niechciana poczta), razem 8.

## 3. Jak działa fragment zliczający etykiety?

W `describe_data.py`, w liniach 19–29, skrypt zaczyna od `counts = {"ham": 0, "spam": 0}`. Dla każdego poprawnego rekordu `counts[row["label"]] += 1` dodaje jeden do licznika jego etykiety. W linii 37 `sum(counts.values())` dodaje oba liczniki.

Agent pokazał mi przykład `spam, ham, spam`, a potem podał `ham, spam, ham, ham, spam`. Przewidziałem 3 `ham`, 2 `spam` i 5 wiadomości łącznie. Sprawdziliśmy to, przechodząc po etykietach według kodu. Każda wiadomość zwiększa jeden licznik o jeden.

## 4. Jak uruchomiono gotowy test i co sprawdził?

```powershell
$env:PYTHONDONTWRITEBYTECODE = '1'
py -3 -m unittest discover -s tests -v
```

Test `test_cli_writes_expected_counts` zakończył się kodem `0`. Terminal pokazał `Ran 1 test in 0.116s` i `OK`. Test uruchomił skrypt na `tests/fixtures/messages.csv`, zawierającym trzy syntetyczne wiadomości z wielowierszową treścią. Sprawdził kod wyjścia `0`, istnienie i poprawny odczyt JSON, zgodność `input_file` z plikiem testowym oraz wynik 3 wiadomości, w tym 2 `ham` i 1 `spam`. Użył osobnego raportu w katalogu tymczasowym.

## 5. Co się zmieniło po zmianie etykiety?

Agent zmienił `exercise-001` z `ham` na `spam` w `data/exercise.csv`. Raport z pierwszego uruchomienia nadal pokazywał stare liczby. Przewidziałem, że `ham` spadnie o 1, `spam` wzrośnie o 1, a suma pozostanie taka sama. Zleciłem ponowne wykonanie polecenia z punktu 2, bo zapisany JSON nie odświeża się po edycji CSV.

| Liczebność | Pierwszy raport | Moje przewidywanie | Końcowy raport |
|---|---:|---:|---:|
| `ham` | 5 | 4 | 4 |
| `spam` | 3 | 4 | 4 |
| Razem | 8 | 8 | 8 |

Ponowne uruchomienie zakończyło się kodem `0` i wypisało `Messages: 8; labels: {'ham': 4, 'spam': 4}`. Agent odczytał nowy raport z tej samej ścieżki i potwierdził, że zawartość zmieniła się po uruchomieniu. Ścieżka wejściowa była ta sama, ale SHA-256 danych się zmieniło. Suma z końcowego raportu zgadzała się z wynikiem `Get-FileHash` dla zmienionego CSV.

- Przed zmianą `95e9049fe9f1283ba65514c7060499d175deaa88cd8c582097ced4cceed6ed07`
- Po zmianie `7dc8080df480e67e5361218ae0e75f26c963ede9d72788985b25cedb818b8152`

Gotowego testu w drugim etapie nie uruchamialiśmy ponownie.
