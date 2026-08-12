# FiremniProjekt

Akademické dokumenty k firemnímu projektu (NPRG071, MFF UK, program SDE): modernizace klienta EDISON v M-line. Student: Martin Hubata, supervizor: Mgr. Petr Škoda, Ph.D., konzultant: Bc. Jiří Krhánek. Projekt běží 1.6.2026 až 1.3.2027.

## Soubory

- `project-proposal.tex` — schválený záměr. Zdroj hlasu i obsahu pro navazující dokumenty; nic z něj nesmí v pozdějších dokumentech chybět.
- `project-specification.tex` — specifikace (šablona fakulty, vyplňuje se).
- `notes-for-specification.md` — body záměrně vyňaté ze záměru, které patří do specifikace.

## Pravidla pro text

- Dokumenty anglicky, lidským hlasem záměru: prosté věty, žádný korporátní/AI tón, žádné em dashes (`---` ani `—`); en dash ` -- ` střídmě, jako v záměru.
- Specifikace teoreticky předchází implementaci: žádné implementační detaily z ../EDISON2 (názvy tříd, knihoven, „already implemented"). Repo EDISON2 slouží jen ke kalibraci realističnosti; psát forward-looking na úrovni požadavků.
- Místa k ručnímu doladění autorem značit `\textcolor{red}{TODO: ...}`.

## Build

MiKTeX, `pdflatex <soubor>.tex` (2x kvůli referencím). Balíček `pdfx [a-2u]` vyžaduje `<soubor>.xmpdata` s metadaty. Výsledné PDF po změnách vizuálně zkontrolovat (TikZ diagramy, tabulky, přetečení).
