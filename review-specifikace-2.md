# Adversariální review project-specification.tex — 2. kolo

Datum: 14. 8. 2026. Read-only review přepsané verze specifikace (po zapracování review z 13. 8.), žádné editace dokumentu.

Ověřeno proti: spec na HEAD `f043db3` (working tree shodný), project-proposal.tex, notes-for-specification.md, šabloně fakulty (`af8ff32`), review-specifikace-2026-08-13.md, a repům EDISON (`8896933`), EDISON2 (`04f7108`), edwcore (`6022244`). Tvrzení o legacy systému ověřovali tři subagenti přímo v kódu (licensing, update, shell/NFR); citace soubor:řádek níže pocházejí z jejich evidence. Kompilace: 2× pdflatex, bez chyb, 19 stran, 0 overfull boxů.

Nereportují se schválené odchylky a vědomá rozhodnutí: harmonogram psaný z perspektivy 1. 6. 2026 (implementace je napřed), neportování tray aplikace a zpráv mezi uživateli, žádné plovoucí/multi-instance mini-apps, nezmiňování DEMO licence, portace agend out of scope.

**TL;DR:** Přepis po prvním review odvedl hodně práce — licenční narativ, C4 toky, N11, zúžené U1 i milníky MS1–MS7 teď sedí na realitu a drtivá většina faktických tvrzení o legacy systému prošla ověřením v kódu. Zbývají čtyři věci, které stojí za opravu před odevzdáním: nedořešené místo, kde developeři editují vazby modul→licencovatelná feature (L4 říká zákaznická Caché, storyboard licenční editor nad centrálním archivem); L9 degradované na nice-to-have, ačkoli proposal i vlastní Out of scope ho slibují; věta „the plan is a full recreation" v rizicích, která přežila zúžení U1; a rozpočet 14 MD na celou licenční modernizaci, který první review rozporovalo a nezměnil se.

---

## Nálezy

### MAJOR

**1. Kde se editují definice modul→licencovatelná feature: L4, storyboard a diagram si odporují.**
Místo: L4 (ř. 249), storyboard (ř. 350), fig:c4-container (caption + cylinder „Customer Caché backend — module definitions").
Co je špatně: L4 stěhuje definice modulového stromu „into the **customer's** Caché backend" a tam dává developerům „a supported way to edit them". Storyboard ale říká: „A **related view** for developers allows editing the module relationships (which elements belong to which licensable feature), replacing the manual XML editing" — related view = ve webovém licenčním editoru, který podle téhož odstavce ukládá do **centrálního** archivu M-line. Container diagram drží module definitions u zákazníka a editor pouští jen na archiv. Licenser přitom potřebuje katalog licencovatelných features centrálně (zaškrtává moduly pro libovolného zákazníka dřív, než jeho instalace existuje) — takže data musí žít na obou místech a specifikace ten rozklad nikde nepopisuje, ani jak se centrální změny dostanou do zákaznických backendů.
Proč to vadí: Je to jádro licenčního outcome; konzultant i oponent se na tom zaseknou. Realita ten rozklad má: mapování `licensed=` je dnes v `edisontree.xml` v repu klienta (435 výskytů), zatímco feature bundly („karty") definuje licenser centrálně (`EDISON\Licence\Licence\FrmCardDefinition.vb`) a nový katalog se odvozuje ze stromu (`EDISON\Licence\CACHE\ALWA\LICENCE\API\Catalog.cls:1-14`).
Návrh opravy: Rozdělit v textu dvě věci: (a) modulový strom per instalace → zákaznická Caché (L4), (b) katalog licencovatelných features → centrální archiv, editovaný v nástroji z L3/storyboardu. Jedna věta o tom, kudy teče synchronizace (např. s releasem), a sladit storyboard s L4.

**2. L9 je nice-to-have, ale proposal i vlastní Out of scope ho slibují.**
Místo: L9 (ř. 254), Out of scope (ř. 193), proposal ř. 67.
Co je špatně: Proposal: „Preparation for reuse across other M-line products **is in scope**." Spec Out of scope tvrdí: „The licensing data model **is prepared** for reuse by other M-line products…" A pak L9 říká, že product-agnostický návrh je nice to have, tedy „may be reduced under time pressure". Dokument si tak současně slibuje i vyhrazuje právo nesplnit tutéž věc — a proti schválenému proposalu je to tiché zeslabení závazku.
Návrh opravy: Nejlevnější je L9 povýšit na essential (product-agnostický *návrh* modelu není velká práce navíc a už se tak fakticky děje). Alternativně přeformulovat Out of scope a přiznat odchylku v intru vedle pilotního zpřísnění.

**3. Riziko „the plan is a full recreation" přežilo zúžení U1.**
Místo: Risks, „Update mechanism underestimated" (ř. 515).
Co je špatně: U1 po přepisu správně říká „The download and apply steps may be carried out by the existing update engine. Replacing it is possible but not promised," a Architecture mluví o „recreates the coordination logic". Risk paragraf ale pořád tvrdí „the plan is a full recreation" — komise čte závazek, který požadavky už nedávají. Pro úplnost: převzetí legacy enginu je reálné, EDUPDATE.EXE je oddělitelný (download UNC/HTTP, apply s backupem, rollback s exit kódy 100/200, restart — `EDISON\UPDATERCL\clsAPI.vb:1235-1360`) a už má parametry pro nový klient (`WaitProcess=`/`RunExe=`, `frmClientUpdate.vb:58-61`).
Návrh opravy: V risku „…and the plan is a recreation of the coordination logic" (nebo ekvivalent), ať risk sedí na U1 a Architecture.

**4. 14 MD na celou licenční modernizaci — neopraveno z 1. kola, a evidence pro podcenění přibyla.**
Místo: Time Schedule, řádek MS5; L1–L7.
Co je špatně: První review (nález 6) rozporovalo 14 MD na data model + store + napojení na delivery + webový editor + developer tooling + verifikovanou migraci; číslo se nezměnilo (TODO „review the man-day split" trvá). Nová evidence problém spíš zhoršuje: migrace se musí vypořádat s limity USERS/RZ/ZAM/TELMAX/CACHE (`EDISON\Licence\EDLicence\CLicence.vb:36-78`), s odečtem CountsCACHE od login limitu, s hardwarovými init klíči (`Activation.vb:328-366`, nesouhlas → read-only stav E03) a s TRIAL elementy přilepovanými za běhu k aktivní licenci (`Activation.vb:241-259`) — nic z toho „same modules before and after" nepokryje (viz nález 9). Polehčující okolnost: značná část L1/L3 už fakticky existuje (webový editor v `edwcore\apps\mlweb` + nový store `ALWA.LICENCE.*` s `Migrate.cls`), takže rozpočet může být obhajitelný — ale pak je to argument, který v dokumentu není a před komisí ho nelze použít (spec má implementaci předcházet).
Návrh opravy: Přerozdělit man-days ve prospěch MS5 (např. z MS2/MS3), nebo v L3/L4 explicitně vymezit MVP hranici toho, co se za 14 MD slibuje.

### MINOR

**5. Intro přiznává jedinou odchylku od proposalu, ale jsou nejméně dvě další.**
Místo: ř. 68 „(with the pilot criterion tightened since approval)".
Co je špatně: Proti proposalu se změnilo i (a) testovací kritérium: „all public APIs in the toolkit and shell ship with unit tests" → N3 „all testable public logic" (vědomé a rozumné zeslabení po 1. review, ale zeslabení), a (b) L9 (nález 2). Půlvěta v intru tak vytváří dojem, že zbytek je 1:1.
Návrh opravy: Buď vyjmenovat („with the pilot criterion tightened and the testing commitment narrowed to testable logic"), nebo formulaci zobecnit.

**6. Slibovaný „buffer before the project end" v tabulce neexistuje.**
Místo: Time Schedule text (ř. 558), Risks „Company priorities" (ř. 521) vs. tabulka.
Co je špatně: Text i mitigace risku slibují buffer před koncem projektu, ale tabulka plánuje Documentation 1–2/2027 a Evaluation 2/2027; projekt končí 1. 3. 2027. Rezerva je nulová — buffer existuje leda implicitně v tom, že 120 MD nevyčerpává kapacitu, což text neříká.
Návrh opravy: Buď buffer skutečně vyhradit (ukončit aktivity v polovině února), nebo formulovat, v čem buffer spočívá (nevyčerpaná kapacita posledních měsíců).

**7. M6 (povinná mini-app s novinkami) nemá zdrojovou stranu.**
Místo: M6 (ř. 236).
Co je špatně: „Company news and announcements… the company's first direct channel to the end users" — ale odkud se novinky berou (kdo je píše, kde jsou uložené, jak se dostanou do zákaznické instalace), neříká žádný požadavek, milník ani diagram. Srovnej U6, které svůj zdroj (INF soubory → později company web) řeší explicitně. M6 je přitom essential.
Návrh opravy: Jedna věta o zdroji a autorské cestě (např. centrální služba / company web, s fallbackem při nedostupnosti) a zařazení do MS4.

**8. L1 „replacing the opaque encrypted format" odporuje N11.**
Místo: L1 (ř. 246) vs. N11 (ř. 300).
Co je špatně: N11 správně říká, že legacy klienti musí dál číst kompatibilní formát (3DES-ECB, klíč `MD5("mline2k9")` v distribuované `EDLicence.dll` — `EDISON\Licence\EDLicence\CryptText.vb:27-32`, `MConsts.vb:24`; nový web tool drží byte-for-byte kompatibilitu, `edwcore\apps\mlweb\backend\src\services\licence\licenceCrypto.ts:5-13`). Šifrovaný formát tedy nezaniká — přestává být jediným zdrojem pravdy, ale delivery ho dál emituje. „Replacing" slibuje víc, než N1+N11 dovolují.
Návrh opravy: Např. „…a new, structured data model and store in M-line's central archive that becomes the authoritative source behind the existing delivery" — a „replacing the opaque encrypted format" vypustit nebo omezit na roli zdroje pravdy.

**9. Ekvivalenční kritérium L5 je užší než data, která migruje.**
Místo: L5 (ř. 250) vs. licenční preambule (ř. 244).
Co je špatně: Preambule definuje licenční data jako typ + validity + feature set + numerické limity, ale ekvivalence se měří jen „sees the same modules before and after". Limity, validity a další sémantiku (init klíče, TRIAL merge — viz nález 4) kritérium nepokrývá, takže migrace může projít kontrolou a přesto změnit chování instalace.
Návrh opravy: „…a customer installation sees the same modules, limits, and validity before and after the migration" + rozhodnout (a napsat), zda init klíče do nového modelu patří, nebo se ruší.

**10. L8 slibuje jako nice-to-have budoucnost něco, co existuje dnes.**
Místo: L8 (ř. 253) vs. preambule „delivered automatically…".
Co je špatně: Automatická obnova licence z centrálního archivu před expirací už běží: pull <15 dní před koncem platnosti (`EDISON\Licence\EDLicence\Activation.vb:286-296`, `Update.vb:21-89`), spouštěný při loginu oprávněného, z updateru i z Feedback tasku. Druhá klauzule L8 („the installation renews it automatically from the central archive before it expires") tak deklaruje existující chování jako volitelné budoucí — a nice-to-have status paradoxně znamená, že nový systém by ho směl i ztratit. Nová je jen první klauzule (runtime refresh bez restartu shellu).
Návrh opravy: Druhou klauzuli přeformulovat jako zachování existujícího chování (a přesunout k essential, pokud o něj instalace nesmí přijít), L8 nechat jen runtime refresh.

**11. U7 popisuje druhou „known failure" nepřesně.**
Místo: U7 (ř. 265).
Co je špatně: Skutečná vada legacy mechanismu není primárně „re-download of the client executables", ale to, že Caché-only patch bumpne jednotný řetězec verze (`VERZE.INF`) a tím **požene všechny stanice povinným update cyklem** (exit → EDUPDATE → restart; ForceYes v `EDISON\EDISONMAIN\Rozhrani (mainform)\frmNoMDI.vb:4446`). Samotnému překopírování nezměněných exe už dnes částečně brání CRC diff (`EDISON\UPDATERCL\clsAPI.vb:1095-1114`) — plný re-download nastává jen v degradovaných cestách (chybějící `EDISONCRC.bin`, HTTP ListFiles větev). Kdo legacy zná, doslovné čtení U7 zpochybní.
Návrh opravy: „…and a Caché-only patch never forces running clients through the update cycle" (což mimochodem přesně sedí na U2 „server-only changes are tolerated").

**12. Evaluace nepokrývá licenční a update outcome.**
Místo: Evaluation (sekce 4).
Co je špatně: Tři evaluační pilíře (testy, pilot, developer study) pokrývají čtyři z šesti outcomes; licensing a update nemají žádné kritérium. Pilot je nesupluje — spec nikde neříká, že pilotní web do 1/2027 projde migrací licence a updatem přes nový mechanismus. Přitom měřitelná kritéria leží na stole: L5 ekvivalence, licence reálně vydaná novým editorem, klient aktualizovaný novým mechanismem na pilotním serveru.
Návrh opravy: Přidat do Evaluation odstavec s těmito třemi kritérii (nebo výslovně napsat, že MS5/MS6 demonstrace jsou evaluačním kritériem).

**13. „Around fifty agendas" — proposal říká thirty, strom říká ~42.**
Místo: ř. 161 vs. proposal ř. 190.
Co je špatně: Spec tiše změnila proposalové „around thirty" na „around fifty". Počítáno z `EDISON\EDCORE\Resources\edisontree.xml`: 45 kořenových skupin, z toho 42 viditelných (skryté: Licence systému, Online aplikace, Miniaplikace); licencovatelných hlavních balíků je naopak 73 (`EDCORE\EDFormInterfaceLic.vb:80-159`). „Fifty" je tedy obhajitelné jen při vstřícném počítání a proposal je každopádně vedle — ale dvě různá čísla v navazujících dokumentech vyvolají otázku.
Návrh opravy: „Around forty" / „over forty" (agendy v nabídce), případně poznámka o metodice; odchylku od proposalu není nutné inzerovat, číslo je popisné.

**14. N16/N17 sbírají osobní údaje a dokument o ochraně dat mlčí.**
Místo: N16, N17 (ř. 305-306), Risks.
Co je špatně: Error report dnes reálně nese login, celé jméno, IP a obsah oken (`EDISON2\EDControls\dotnet\ErrorReporting\ErrorReporter.cs`; technické jádro odchází automaticky bez interakce uživatele) a EDISON provozuje mzdové agendy — obsah okna může být osobní údaj zaměstnance zákazníka. Usage telemetrie je per user. Spec nemá ani větu o právním základu, anonymizaci či retenci a rizika to nezmiňují. Own-windows-only screenshoty jsou přitom oproti legacy (celá plocha všech monitorů, `EDISON\EDCORE\clsCore.vb:141-177`) vědomé zlepšení — o důvod víc to říct nahlas.
Návrh opravy: Jedna–dvě věty v N16/N17 (co se sbírá, že screenshoty jen vlastních oken jsou záměrná ochrana, agregace u telemetrie) a případně řádek v rizicích.

**15. Rizika: chybí závislost na WPF docking knihovně.**
Místo: Risks vs. Platform „Mini-app hosting: an established WPF docking/layout library".
Co je špatně: M1 (hosting area, jádro mini-app subsystému) stojí na third-party knihovně, jejíž volba se odkládá. Licence, kvalita, údržba a výměna takové knihovny je klasické projektové riziko a sekce Risks ho nemá — přitom obsahuje i měkčí rizika (company priorities).
Návrh opravy: Řádek v rizicích: výběr podle licence a údržby, toolkit knihovnu obalí, aby šla vyměnit.

**16. Zbytky měřitelnosti z 1. kola: N6 bez prahu, N3 bez vynutitelnosti.**
Místo: N6 (ř. 295), N3 (ř. 292) + Evaluation.
Co je špatně: Neopravené body 15b/15d z minulého review: „not perceptibly slower" nemá práh ani metodu měření a „tests run automatically before every push" stojí na lokálním pre-push hooku, který jde obejít `--no-verify` — jediný ověřitelný gate, žádné CI. Obojí je hlavní evaluační opora.
Návrh opravy: N6: práh (např. „startup within 1.5× of the legacy shell on the same workstation") nebo aspoň metoda; N3/Platform: jedna věta o tom, kde testy běží povinně.

### NIT

**17. C4 zbytky.** (a) „M-line license archive" je v contextu `[Software system]`, v containeru `[Database]` — sjednotit; (b) License Editor, celý kontejner „(this project)", nemá v context diagramu žádný mateřský systém — licenser tam interaguje jen přes label „[web editor]" na šipce; (c) context značí celé „EDISON 2 (this project)", zatímco container do EDISON 2 zahrnuje TemplateDesigner, který projektem není (caption to vysvětluje, značka přesto přehání); (d) šipka shell→bridge jako jediná nemá technologii („Hosts agendas via"); (e) container míchá kontejnery několika systémů v jednom diagramu — po rozhodnutí z 1. kola snesitelné, ale s (a)+(b) by šlo vyřešit najednou.

**18. S9 „during a session" odkazuje na U1, které je startup flow.** U1 začíná „At startup…"; co přesně se stane při mid-session mismatchi (U2 odmítá requesty, ale kdo spustí update?), zůstává mezi S9/U1/U2/U8 nedořečené. Jedna věta v S9 (výzva k restartu do updatu) to zavře.

**19. U2 „server-only changes are tolerated" — podle čeho.** Legacy guard toleruje podle **segmentu verze** (patch), ne podle obsahu: FULL release, který klienta nemění, dnes běžící klienty vyhodí (`EDISON\MLRest\MLRest\CACHE\ALWA\SYS\Rest.cls:1700-1725`). Pokud U2 slibuje toleranci podle obsahu, je to změna sémantiky, ne parita — stojí za upřesnění. (Runtime guard sám o sobě je mimochodem legacy chování, ne novinka.)

**20. Granularita persistence: S7 vs. M11/T8/S13.** S7 mluví o per-user settings „(layout, favorites, last company)", zatímco M11, T8 i S13 persistují per user **and company**. Pokud „layout" v S7 je tentýž layout jako v M11, granularity si odporují; S7 zúžit na čistě per-user položky.

**21. Storyboard „No file is sent to the customer manually."** Jako popis dneška to není pravda — manuální cesta je zabudovaná (kopie šifrované licence do schránky, `EDISON\Licence\Licence\FrmLicence.vb:729-734`, ruční vložení v `formEDISONMAINSettingsShow.vb:1405-1415`). Věta je míněná jako cílový workflow, ale present tense ji čte jako fakt. Přeformulovat („no file needs to be sent…") a říct, zda manuální fallback zůstává.

**22. L2 „the system runs read-only" je zděděné, kooperativně vynucované chování.** Read-only po expiraci existuje (E04_OUT_OF_DATE → `MLProps.ReadOnlyMode`), ale vynucuje si ho každý formulář sám, server nic neblokuje — a u neportovaných agend to tak zůstane. Věta se čte jako systémová garance projektu; přesnější je „the installation stays read-only, as today".

**23. M3 „the one exception is M6" vs. desktop z S13.** Desktop je podle S13 také mini-app; pokud ho uživatel nemůže odebrat, výjimky jsou dvě, pokud může, stojí to za slovo v S13.

**24. Terminologie M9 „functional restrictions".** Jinde se týž koncept jmenuje „the user's permissions" (S3, S6, M4); „functional restrictions" (zřejmě legacy „funkční omezení") se objevuje jednou a bez definice.

**25. „Agendas and mini-apps… reuse the session established at login instead of opening their own" (Architecture) vs. diagram.** Legacy agendy jsou samostatné procesy s vlastním spojením (v containeru správně „Queries [HTTP]" přímo na backend; legacy jim dnes dokonce předává jméno+heslo a ony se připojují znovu — `EDISON\EDLIBRARY\clsForm.vb:4640-4897`). Věta platí jen pro logickou session (GUID/seat), ne pro spojení; jedno slovo („the logical session") to vyjasní.

**26. pdfTeX: 6× „destination with the same identifier (figure.N) … duplicate ignored".** Kombinace balíčku `float` ([H] mockupy) s hyperref bez hypcap; odkazy na figures mohou mířit na špatné místo na stránce. Fix: `\usepackage[hypcap]{caption}` nebo `hypcap` u float.

**27. Figure 1 (context diagram) doplavala doprostřed odrážkového seznamu sekce 2** (str. 3: mezi „Testability reduces…" a „The architecture will…"). Referencovaná v 1.1, zobrazená uvnitř cizího výčtu. `[tb]` místo `[h]`, nebo posunout ve zdrojáku.

**28. MVVM odkaz vede na MAUI e-book** (`learn.microsoft.com/.../architecture/maui/mvvm` — ověřeno, stránka je živá a MAUI-specifická). Pro WPF projekt působí odkaz na MAUI příručku nedbale; existuje obecný/WPF zdroj.

---

## Otázky na autora

Věci neověřitelné z rep, které můžou být nepřesnost:

1. **Kde budou developeři editovat vazby element→licencovatelná feature** — centrálně (v editoru z L3), per zákazník (v jeho Caché dle L4), nebo obojí s synchronizací? (Rozhoduje o nálezu 1.)
2. **Kdo píše a kde žijí company news pro M6?** (Rozhoduje o nálezu 7.)
3. **Projde pilotní web do 1/2027 migrací licence a updatem novým mechanismem?** Pokud ano, evaluace to může říct a nález 12 se zmenší.
4. **U2: má být tolerance server-only změn podle obsahu release, nebo podle segmentu verze jako dnes?**
5. **Je desktop mini-app (S13) odstranitelný uživatelem?** (M3 připouští jedinou výjimku.)
6. **Jakou definicí se počítá „around fifty agendas"?** Kořenových skupin stromu je 45 (42 viditelných), licencovatelných balíků 73.
7. **Patří hardwarové init klíče do nového licenčního modelu, nebo se ruší?** (Ovlivňuje L1/L5 — nález 9.)
8. **Funguje odkaz `intersystems.com/products/cache/`?** Z tohoto prostředí se nepodařilo ověřit (spojení resetováno); Caché je discontinued produkt a web InterSystems se restrukturalizoval.
9. **Keywords v xmpdata mají „InterSystems Cache" bez háčku** — záměr kvůli PDF/A metadatům, nebo přehlédnutí?
10. **Fakta o osobách** (licenser jako role, konzultant „responsible for the biggest portion of the legacy stack") — z rep neověřitelné, přenáší se z 1. kola.

---

## Co prošlo bez nálezu

Struktura a forma:

- Struktura sekcí 1:1 se šablonou fakulty (`af8ff32`), hlavičková tabulka kompletní.
- Man-days sečteny přesně na 120; 120 MD / 9 měsíců ≈ 63 % úvazku sedí na „substantial part of the author's working time".
- Všech 12 užitých `\req{}` odkazů vede na existující `\reqdef`; žádná duplicitní ID; nice-to-haves jsou důsledně na konci skupin (S18–S21, L8–L9, U11, N22–N24).
- Jazyk: žádné em dashes (`---`/`—`), ` -- ` střídmě jako v záměru, nula AI výraziva, angličtina konzistentně lidským hlasem záměru; 3× červené TODO dle konvence.
- Kompilace bez chyb a overfull boxů; PDF/A metadata (xmpdata) přítomná; oba TikZ diagramy čitelné, tabulka bez přetečení.
- Notes-for-specification pokryty beze zbytku: entity licence v preambuli, per-user opt-in (M3), srovnání s legacy, WPF motivace v Architecture, persona multi-agenda uživatelů, demo scope v Out of scope.
- Proposal přenesen bod po bodu (intro, relation, stakeholders, benefity, pain points, AI-augmented development, delivery+snapshot+consent, demo na obhajobě); zpřísnění pilotu přiznané v intru.
- Konzultační kadence sedí (7 milníků / 39 týdnů ≈ 5,6 týdne ≈ „about every six weeks").
- Mockupy konzistentní s S2/S15/S16/S17/U1; storyboard (mimo nálezy 1 a 21) sedí na reálný legacy nástroj FrmManage/FrmLicence (stromy s checkboxy, limity, validity).

Faktická tvrzení ověřená v kódu (výběr):

- **Licensing preambule**: licence = šifrovaný záznam v zákaznické Caché (`^ALVA.LICENCE`), automatická delivery pullem z centrálního archivu `ALVA.LICARCHIV` — potvrzeno (`EDISON\Licence\EDLicence\Update.vb:21-89`).
- **Module tree = XML distribuované s klientem**: `edisontree.xml` (918 uzlů) + `edisonnavbar.xml`, embedded v ALVA.CORE.dll; sémantika z L4 existuje celá — `hidden` (26×), `mlineOnly` (133×), `input` (27×), `state` (window defaults).
- **Per-workstation counting** (Architecture, TD bez extra seatu): `ALWA.SYS.Activity.CountIdfa` iteruje IP adresy a vlastní IP volajícího vynechává — potvrzeno.
- **L7**: bez licence vznikne fallback licence (Type „Nezadána", jen Nastavení s importem) — potvrzeno v legacy i EDISON2.
- **N11**: plná tamper-resistance skutečně nedosažitelná při N1 (statický klíč v distribuované DLL, symetrická šifra; nový web tool musí držet byte-for-byte kompatibilitu).
- **S1**: tři auth módy (EDISON/LDAP/LDAP-AUTO) jako per-user atribut rozhodovaný na serveru (delegated auth `ALVA.SYS.Login`, feature 23) — potvrzeno.
- **S3**: strom filtrovaný současně licencí a per-user právy z Caché — potvrzeno.
- **S6**: ne-agendová okna (import/export záloh v UTILITIES) existují; label „Spawns [.NET Remoting]" u legacy shellu je věcně správný (EDPROCESS.EXE + IpcChannel) — otázka č. 2 z 1. kola tímto uzavřena.
- **S10, S13, S14**: první konfigurace připojení dialogem, desktop se zástupci modulů (EDDesktop) i fokus už běžící agendy — vše má legacy předlohu.
- **S16**: „where the backend permits it" sedí — server flag `^EDISON("System","Login","RememberUser")`.
- **N2**: side-by-side je designový i dnešní dev setup, žádný mutex/single-instance konflikt.
- **N14**: zámky per session GUID vč. `LockClear` při logoutu (`ALWA.SYS.Lock`) — potvrzeno.
- **N16/N17**: support backend (podpora.m-line.cz, telemetry ingest v edwcore/mlweb), error reporting s own-windows-only screenshoty i usage telemetrie na obou stacích už existují — požadavky jsou realistické a rozpočtově nenáročné.
- **N20**: inaktivita vynucovaná na serveru (`ALWA.SYS.Activity.UpdateTimeout`) s per-user override (feature 30) — „as in the legacy system" sedí.
- **N21**: sdílení theme/font mezi novými aplikacemi přes `%APPDATA%\m-line\common` existuje; „Legacy WinForms agendas keep their own look" sedí.
- **N22**: barevné schéma per databáze (produkce vs. test) v legacy existuje.
- **M7**: 15 legacy panelů už hostováno přes nezměněný kontrakt `ALVA.EDISON.MiniAplikaceAPI.LoadData`.
- **T2**: server-driven formuláře mají precedens v obou generacích (legacy EDFORMS BIG/BDG, EDInputForm).
- **Update fakta**: legacy update = prostá kopie ze serveru zákazníka, plain HTTP (hardcoded `http://release.m-line.cz:59993`), release notes jako per-agenda INF soubory s diffem a per-verze „už neukazovat", force-out s 60s floorem („about a minute" v U8 je přesné), pauza distribuce (`BlokovatAktualizace`), autorizace startu updatu (admin/AllowUpdate + kontrola aktivních uživatelů) — U-skupina stojí na pravdivém obrazu legacy.
- **Intro o updateru** („known security and reliability issues that nobody dares to fix"): plain HTTP bez integrity kontroly, admin credentials v command line, busy-wait smyčky; opuštěný rewrite `EDISON\ClientUpdater` (prázdná kostra) nepřímo potvrzuje i „nobody dares".

Opravy z 1. kola ověřeny jako provedené: nálezy 1–5, 7–14, 16–19, 21, 22 (licenční narativ, C4 toky, N11/L2, U1, editor v containeru + Platform + delivery, N3, M3, S2, storyboard, N10, N7, S6 fix list, TD výjimka, MS čísla, entity, S1 módy, S9↔U1 odkaz, kadence, půlvěta o pilotu). Neopravené zbytky jsou zachyceny v nálezech 4 (ex-6), 16 (ex-15b/15d) a 17 (ex-20).
