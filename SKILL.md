---
name: sot-visualizer
description: >
  Analizuje dowolny dokument metodą Structure of Thought (SoT) i automatycznie
  tworzy interaktywny wizualizer grafu węzłów i relacji w React+D3.
  ZAWSZE używaj tego skilla gdy użytkownik:
  - wrzuca dokument (raport, brief, artykuł, transkrypt, PDF) i prosi o analizę,
    syntezę, wizualizację, mapę myśli lub "pokaż strukturę",
  - pyta o "węzły", "graf", "strukturę dokumentu",
  - chce zobaczyć zależności, argumenty, sprzeczności w tekście,
  - mówi "przeanalizuj", "zwizualizuj", "zrób mapę" dla jakiegokolwiek dokumentu.
  Nie czekaj na wyraźną prośbę o "SoT" lub "wizualizer" — kontekst dokumentu wystarczy.
---

# Structure of Thought Visualizer

## Co robi ten skill

1. Ekstrakcja struktury dokumentu → JSON (węzły + relacje)
2. Generowanie interaktywnego grafu React+D3 jako artefakt

Cały flow dzieje się automatycznie, bez pytania użytkownika o szczegóły.

---

## KROK 1 – Ekstrakcja SoT JSON

Przeczytaj dokument i wyodrębnij strukturę w formacie:

```json
{
  "nodes": [
    {
      "id": "n1",
      "typ": "fakt | trend | teza | dane | sprzeczność | luka | wniosek_autora",
      "label": "krótka etykieta (maks. 5 słów)",
      "cytat": "dosłowny fragment z dokumentu",
      "pewność": "wysoka | średnia | niska"
    }
  ],
  "links": [
    {
      "source": "n1",
      "target": "n2",
      "relacja": "potwierdza | podważa | wyjaśnia | poprzedza | kontrastuje | wynika_z"
    }
  ]
}
```

### Zasady ekstrakcji

- **8–16 węzłów** – tylko elementy krytyczne, nie streszczaj wszystkiego
- **Obowiązkowe typy**: minimum 1× `sprzeczność`, minimum 1× `luka`
- **`cytat`** – dosłowny fragment z dokumentu, obowiązkowy dla każdego węzła
- **`pewność`**: `wysoka` = poparte danymi w tekście, `średnia` = opinia/interpretacja, `niska` = spekulacja
- **Linki** pokazują logikę argumentu, nie chronologię – szukaj związków przyczynowych
- **Nie generuj** węzłów bez pokrycia w tekście

### Typy węzłów – kiedy używać

| Typ | Kiedy |
|-----|-------|
| `teza` | Główne twierdzenie autora |
| `fakt` | Stwierdzenie opisowe, weryfikowalne |
| `dane` | Liczby, statystyki, wyniki badań |
| `wniosek_autora` | Interpretacja danych przez autora |
| `sprzeczność` | Napięcie, paradoks, nieścisłość w tekście |
| `luka` | Czego brakuje, co pominięto, co wymaga weryfikacji |
| `trend` | Kierunek zmiany opisany w dokumencie |

---

## KROK 2 – Generowanie artefaktu React

Po ekstrakcji JSON **natychmiast** wygeneruj artefakt .jsx.

Wczytaj szablon z: `references/component-template.md`

Podstaw wyekstrahowany JSON w miejsce `DATA_PLACEHOLDER`.

### Checklist przed generowaniem

- [ ] JSON ma minimum 8 węzłów
- [ ] Każdy węzeł ma `cytat`
- [ ] Jest minimum 1 węzeł `sprzeczność` i 1 `luka`
- [ ] Linki mają poprawne `source`/`target` (id węzłów które istnieją)
- [ ] Tytuł dokumentu wyekstrahowany do zmiennej `TITLE`

---

## KROK 3 – Komunikacja z użytkownikiem

Po wygenerowaniu artefaktu napisz krótko (3–4 zdania):

1. Ile węzłów i relacji wyekstrahowano
2. Jakie główne napięcie/sprzeczność znaleziono
3. Co można zrobić dalej (np. eksport JSON, deep-dive konkretnego węzła)

**Nie opisuj** jak działa wizualizer – użytkownik sam to widzi.

---

## Obsługa edge cases

**Dokument bardzo krótki (<500 słów):** Zredukuj do 5–8 węzłów, poinformuj użytkownika.

**Dokument bez wyraźnych sprzeczności:** Stwórz węzeł `sprzeczność` dla największego napięcia między założeniem a danymi.

**Dokument techniczny / kod:** Używaj typów `fakt` i `dane` dominująco, `teza` dla głównego celu systemu.

**Wiele dokumentów naraz:** Przetwarzaj każdy osobno, potem zaproponuj scalony graf.

---

## Ważne

- Nie pytaj użytkownika o potwierdzenie formatu – działaj od razu
- Nie pokazuj JSON w odpowiedzi – tylko artefakt wizualny
- Jeśli dokument jest w języku angielskim, etykiety węzłów tłumacz na polski (chyba że użytkownik pisze po angielsku)
- Cytaty zostawiaj w oryginalnym języku dokumentu
