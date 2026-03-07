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

1. Sklonuj lub pobierz to repozytorium
2. Umieść folder `sot-visualizer/` w katalogu skills swojego środowiska Claude:

   skills/user/sot-visualizer/SKILL.md
   skills/user/sot-visualizer/references/component-template.md

3. Gotowe — skill uruchamia się automatycznie gdy wrzucisz dokument do claude.ai

## Inspiracja

Skill oparty na metodzie Structure of Thought z pracy:  
*T2S-Bench & Structure-of-Thought* (Wang et al., 2026, arXiv:2603.03790)

## Kontakt:
[heuristica.pl](https://heuristica.pl) · [LinkedIn](https://www.linkedin.com/in/staniszewskimarek/)

- License MIT — feel free to use, modify, and share.
