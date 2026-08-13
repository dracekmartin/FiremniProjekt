# Adversariální review project-specification.tex

Datum: 13. 8. 2026. Read-only review, žádné editace specifikace.

Ověřeno proti: project-proposal.tex, notes-for-specification.md, šabloně fakulty (commit af8ff32), repu EDISON2 (HEAD 04f7108) a — protože licenční a update runtime žije z velké části jinde — i repům EDISON (legacy) a edwcore (MLWeb). Tři schválené odchylky (harmonogram z perspektivy startu projektu, držení client+server slibu u updateru, zpřísněný pilot) se nereportují.

**Co prošlo bez nálezu:** struktura sekcí 1:1 se šablonou fakulty; man-days sečteny přesně na 120; všechny `\req{}` odkazy platné; žádné em dashes ani AI výrazivo; „current LTS release" sedí (net10.0 je LTS); mini-app subsystém, toolkit, legacy bridge, module tree z backendu, session lifecycle (S8) i šifrované credentials jsou realistické — většinu z toho už praxe ověřila.

**TL;DR nejhorších věcí:** licenční narativ specifikace stojí na nepravdivé premise (licence nikdy nebyla XML soubor a centrální DB s distribucí existuje od ~2009), C4 diagramy kreslí licenční a update toky, které neodpovídají realitě ani vlastnímu textu specifikace, a N11 „tamper-resistant" je při platnosti N1 nesplnitelné.

---

## Nálezy

### BLOCKER

**1. Licenční premisa je nepravdivá a část cíle už léta existuje.**
Místo: Licensing preambule („Today this lives in an XML file distributed with the client"), L1, L5, intro bullet „Licensing data migration from an XML file", risk „Licensing migration underestimated", captions obou C4 diagramů.
Co je špatně: Licence podle evidence v repech **nikdy nebyla XML soubor**. Od ~2009 je to šifrovaný řetězec („CodeLic", 3DES) v Caché globálech `^ALVA.LICENCE` na serveru zákazníka (`EDISON\Licence\CACHE\ALVA\LICENCE\Activation.cls`). Centrální M-line Caché instance s archivem `^ALVA.LICARCHIV` a automatickým pullem do zákaznické instalace (15 dní před expirací, `EDLicence\Update.vb`) běží v produkci léta. XML distribuovaná s klientem (`edisonnavbar.xml`/`edisontree.xml`) jsou **definice modulového stromu** — tedy látka L4, ne L1. Spec obě věci slučuje do jednoho nepravdivého tvrzení a jako outcome si připisuje infrastrukturu, která existuje.
Proč to vadí: Konzultant tohle pozná na první čtení. L5 (equivalence check) je postavená na neexistujícím zdroji; risk fallback „legacy clients keep reading the XML file either way" je nefunkční — legacy čte Caché globály.
Návrh opravy: Přepsat narativ na realitu: licence = šifrovaný záznam v zákaznické Caché plněný z existujícího centrálního archivu; projekt = nový datový model/store, nový editor, napojení nového store na distribuci, migrace ze starého úložiště s verifikovanou ekvivalencí. XML premisu nechat jen u L4.

### MAJOR

**2. C4 toky licencí a updatů neodpovídají realitě ani vlastnímu textu.**
Místo: oba diagramy — šipky „Fetches license and updates [HTTP]" z klienta přímo na M-line central DB; legacy „Downloads updates" z centrály; box „M-line central DB — Licenses and update packages".
Co je špatně: (a) Klient licenci nečte z centrály — čte ji ze zákaznické Caché, z centrály ji tahá server-side pull. (b) Update balíčky nejsou v Caché databázi, ale na release serveru; jeden cylinder slučuje dvě různé infrastruktury. (c) Šipka „legacy downloads updates from central DB [HTTP]" je v rozporu s vlastní risk větou „the legacy updater is a plain file copy".
Proč to vadí: Diagram kreslí architekturu, kterou nikdo nestaví, a zbytečně otevírá otázky (offline chování, přímá závislost všech klientských stanic na M-line serveru).
Návrh opravy: Licenční tok vést customer backend → central DB (pull), klient čte z customer backendu; central box rozdělit (license archive vs. release distribution); update šipky sladit s risk textem.

**3. N11 „tamper-resistant" je při platnosti N1 nesplnitelné a L2 dává enforcement do klienta.**
Místo: N11, L2, Architecture.
Co je špatně: L2 říká, že shell „blocks logins over the concurrent-user limit" — a realita to potvrzuje v tom špatném smyslu: limit posílá klient (`LicenceVerifier.cs:36-38`), filtrace modulů je čistě klientská, šifra je 3DES se statickým klíčem zapečeným v distribuované DLL (kvůli zpětné kompatibilitě přenášeným 1:1 do nového nástroje) a HW klíče se ruší. Dokud platí N1 (legacy klienti musí číst stejný formát), tamper-resistance nelze doručit — a je to NFR, proti kterému může být projekt u obhajoby hodnocen.
Návrh opravy: L2 → „backend odmítne login nad limit, shell to prezentuje". N11 zreálnit: server-side vynucení a validace pro nové flows, ochrana proti omylu/driftu; plnou tamper-resistanci explicitně vyloučit kvůli N1.

**4. Update: rozpočet 14 MD nekryje vlastní slib „full recreation" a klientské zúžení není podchycené.**
Místo: Architecture („full recreation rather than incremental improvement"), U1, řádek M6 tabulky harmonogramu.
Co je špatně: Schválená odchylka kryje zužování **serverové** strany. Jenže ani klientská strana se nerekonstruuje celá: download/apply/rollback/restart zůstává práce legacy EDUPDATE.EXE, nový kód je orchestrace. U1 tedy slibuje („download, apply with a rollback on failure, restart") práci, kterou dodá legacy komponenta — a 14 MD je rozpočet orchestrace, ne full recreation client+server. Slib maximalistický, budget minimalistický.
Návrh opravy: U1 přeformulovat na detekci + řízení povinného updatu s možným převzetím apply enginu, klientské zúžení přidat k odchylkám k vysvětlení; nebo narovnat budget.

**5. Licenční editor (L3) nemá ve specifikaci architekturu ani technologii.**
Místo: fig:c4-container (editor chybí jako kontejner; v contextu jen label „[internal tooling]" na šipce), Platform and technologies, Milestones/delivery.
Co je špatně: Editor není kontejner v žádném diagramu a Platform sekce (.NET/WPF/Caché) jeho stack nepokrývá — reálný směr firmy je webový interní nástroj mimo tento výčet, navíc v jiném repu, což delivery snapshot neřeší.
Proč to vadí: Jeden celý outcome bez odpovědi na „v čem a kde to poběží"; komise čte Platform sekci jako závazný výčet.
Návrh opravy: Přidat editor jako kontejner + odrážku do Platform (stačí obecně „web-based internal tool following the platform of the company's newer internal tools") a větu do delivery.

**6. M5 (licensing) za 14 MD je podceněné.**
Místo: Time Schedule tabulka, L3+L4+L5.
Co je špatně: Data model + verifikovaná migrace + editor + developer tooling za 14 MD. Srovnatelný reálný workstream (editor + store) jsou tisíce řádků a týdny práce a napojení store na distribuci je samostatný netriviální krok. L3 přitom není nice-to-have, takže spec se zavazuje k plné verzi.
Návrh opravy: Přerozdělit man-days, nebo v L3 vymezit MVP a zbytek označit nice-to-have — risk sekce fallback má, budget mu neodpovídá.

**7. „All public APIs … covered by unit tests" (N3 + Evaluation) je neměřitelné a nerealistické.**
Co je špatně: V repu není žádný coverage nástroj, WPF views/code-behind smysluplně unit-testovatelné nejsou a Caché strana je řádově řidší než .NET. Hlavní automatizované evaluační kritérium jde vyvrátit jedním grepem.
Návrh opravy: „All testable public logic (view models, services, parsers) ships with unit tests" + volitelně měřitelný proxy (coverage ≥ X % na vyjmenovaných assembly), nebo aspoň vypustit „all".

### MINOR

**8. M3 + fig:mock-miniapps popisují settings dialog s per-user toggly, který byl vědomě odstraněn.**
Mini-apps se přidávají z menu, persistuje se layout per user × firma; mockup „settings dialog with per-user toggles" nepůjde vyfotit. Přeformulovat M3 bez závazku na settings dialog, mockup popsat podle reálného UX (substance per-user volby, kterou žádaly notes, zůstává).

**9. S2 + fig:mock-login slibují výběr firmy při loginu; realita je vědomý auto-pick** (poslední → výchozí → první) + runtime přepínání. Buď S2 přeformulovat („establishes a company context at login…"), nebo evidovat jako odchylku — jinak login screenshot nebude odpovídat textu.

**10. „The licensing editor does not exist yet even in a legacy form worth showing" je zavádějící.**
Legacy editor existuje a je v produkci (WinForms Licence + KeyMaker). Kvalifikátor „worth showing" to činí obhajitelným, ale čte se to jako nepravda. Napsat „a legacy tool exists but is being replaced wholesale; instead, a storyboard: …".

**11. N10 „no hidden global state" neplatí a platit nebude.**
Vědomý dokumentovaný vzor je static service locator (`EDServices.Current`, ~57 call sites) plus další statické seams (theme, fonty, telemetrie). Zmírnit na „global state limited to documented, test-resettable seams; explicit typed DI preferred".

**12. N7 „texts centralized" koliduje s vědomou konvencí inline českých literálů** (pár sdílených klíčů, žádné resx, ani se neplánují). Zreálnit, nebo přiznat single-language scope.

**13. „Still-relevant legacy agendas" není definované.**
S6 i pilot evaluace na tom stojí, kritérium je gumové oběma směry. Závazek: seznam se zafixuje s firmou nejpozději v M3 a bude součástí dokumentace.

**14. „Agendas and mini-apps reuse the session established at login" neplatí pro TemplateDesignera** — vlajkový druhý konzument toolkitu si drží vlastní session GUID. Omezit pravidlo na proces shellu a TD uvést jako výjimku; vyjasnit vztah k login limitu.

**15. Evaluace je slabá na měřitelnost i mimo N3:**
(a) N4 „days, not weeks" místo explicitního „X developer-days" (které chtěly notes) a bez datapointu — portace agend je out of scope, takže říct, jak přesně se to změří; (b) N6 „not perceptibly slower" bez prahu; (c) „all M-line developer workstations" je binární kritérium mimo plnou kontrolu autora; (d) testovací pilíř nemá žádný závazek k CI — jediný gate je lokální pre-push hook obcházitelný `--no-verify`.

**16. Kolize ID: milníky M1–M7 vs. mini-app požadavky M1–M6** („M4 Mini-app subsystem" vedle M4 = license gating). Přejmenovat milníky na MS1–MS7.

**17. Chybí datové schéma licence** — notes explicitně žádaly entity/fields, kdo edituje, kdo konzumuje; spec má jen prózu. Stačí pět řádků entit (License, LicensableFeature/Element, Limits, Validity, Customer) v licensing preambuli.

### NIT

**18.** S1 „the backend decides the authentication mode per user" — vyjmenovat módy (password vs. Windows/LDAP SSO).
**19.** S9 vs. U1/U2 se překrývají bez cross-reference — jedna věta o vztahu (check → update → refuse jen když update nejde).
**20.** C4 konvence: [Database] elementy na context úrovni (mají být systémy); licenser→DB šipka nese „[internal tooling]" jako protokol místo systémového boxu; container diagram míchá kontejnery dvou systémů; TD nemá žádný license/update vztah.
**21.** „About once every two months" u konzultací nesedí na 7 milníků za 9 měsíců (spíš po 1–1,5 měsíci).
**22.** Intro „as committed to in the approved project proposal" — pilot bullet je proti proposalu (schváleně) zpřísněný; půlvěta „with the pilot criterion tightened since approval" srovná doslovné čtení.

---

## Otázky na autora

Věci, které nejdou ověřit z repa a můžou být nepřesnost:

1. Existuje dnes u kteréhokoli zákazníka licence fyzicky jako XML soubor distribuovaný s klientem? Evidence v repech říká, že od ~2009 ne — pokud se to potvrdí, nález 1 platí naplno.
2. Spouští legacy shell agendy opravdu přes .NET Remoting (label v container diagramu)? V EDISON2 Remoting není a most používá reflexi; legacy shell nebyl ověřován.
3. Odkud legacy klienti reálně berou update — lokální file copy ze serveru zákazníka, nebo HTTP z M-line?
4. Má nový shell v cílovém stavu číst licenci přímo z centrální DB (jak kreslí diagram), nebo přes zákaznickou Caché jako dnes?
5. Je webový licenční nástroj (jiné repo) součástí autorovy práce a deliverables projektu? Ovlivňuje L3, Platform sekci i snapshot delivery.
6. Sedí „around thirty agendas"?
7. Kolik lidí je „all M-line developer workstations" a kdo se počítá jako developer?
8. Počítá se TemplateDesigner (vlastní session) do concurrent-login limitu?
9. Fakta o rolích (licenser jako pozice, konzultant „responsible for the biggest portion of the legacy stack") — sedí?
