# Notes for Specification

Tracker informací, které byly z Proposalu vyňaty během konsolidace (9 → 6 bulletů
v outcomes listu), ale které patří do Specification dokumentu — typicky do sekcí
*Functional requirements*, *Non-functional requirements*, *Mockups*, *Architecture*.

Doplňovat průběžně jak narazíme na další body.

---

## Licensing data migration

- **Definice toho, co je "licensing data":** *"what does the buyer of the software have access to, for how many users"* — feature set + uživatelský limit per zákazník. Specification by měla mít přesnější schéma (entity / fields, kdo to editje, kdo to konzumuje).

## Architectural foundation and reusable toolkit

- **Speed claim:** *"enabling rapid construction of new agenda functionalities"*. Specification může mít explicit success metric — např. "first end-to-end agenda port using the toolkit should fit within X developer-days" — měřitelný evaluation criterion pro projekt.

## Modernized shell with mini-app subsystem

- **User-configurability:** mini-apps jsou opt-in **per uživatel** (toggle on/off v settings). Není to globální / per-deployment volba. UX detail patří do mockup / functional req sekce.
- **Comparison vs. legacy:** *"old shell has only a limited capability in this"* — motivation paragraph, proč modernizovat mini-app surface a nezachovat legacy approach.
- **Technology motivation:** *"the full options WPF provides in comparison to WinForms"* — proč právě WPF dělá rozdíl pro mini-apps (data binding, templating, animace, custom controls). Hodí se do Architecture sekce.
- **Target persona:** mini-apps cílí na *"users who use multiple agendas"* — power users co skáčou mezi moduly a potřebují at-a-glance summary. Definovat v User stories / persona sekci.
- **Demo scope clarification:** *"agenda itself won't be rewritten in full, just the mini-app will be, as part of the project"* — explicit statement do *Out of scope* sekce: portace agend = ne; mini-app pro jednu agendu jako demo = ano.

---

## Otevřené otázky pro supervisora

(věci označené `\textcolor{red}{...}` v Proposalu)

- Zda zahrnout commitment k "Observations on AI assistant's strengths and failure modes summarized in final documentation."
- Zda zachovat červeně označené části nebo je vyhodit.
