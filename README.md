# SoT Visualizer — Claude Skill

Skill dla Claude, który automatycznie analizuje dowolny dokument metodą 
**Structure of Thought** i generuje interaktywny graf węzłów i relacji.

## Co robi

Wrzucasz dokument (raport, brief, artykuł, transkrypt) → Claude:
1. Ekstrahuje strukturę argumentacyjną jako graf węzłów i linków (JSON)
2. Generuje interaktywny wizualizer React + D3 jako artefakt

Bez dodatkowych instrukcji. Wystarczy wrzucić plik.

## Jak wygląda output

- Węzły kolorowane według typu: teza / fakt / dane / sprzeczność / luka / wniosek
- Interaktywne linki z etykietami relacji (wyjaśnia / kontrastuje / wynika_z...)
- Panel boczny z cytatem źródłowym po kliknięciu węzła
- Filtrowanie po typie, drag & zoom

## Instalacja

Pobierz `sot-visualizer.skill` z [Releases](../../releases) i zainstaluj 
w Claude Code lub przeciągnij do folderu `.claude/`.

## Inspiracja

Oparty na metodzie Structure of Thought z pracy:  
*T2S-Bench & Structure-of-Thought* (Wang et al., 2026, arXiv:2603.03790)
