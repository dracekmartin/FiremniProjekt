# Adversariální review project-specification.tex, 3. kolo

Datum: 28. 8. 2026. Read-only review, specifikace nebyla editována.

Ověřeno proti: spec na HEAD `da349f9` (working tree čistý), project-proposal.tex, notes-for-specification.md, šabloně fakulty (`af8ff32`), oběma předchozím review, CLAUDE.md, a repům EDISON (legacy), EDISON2 (nový stack) a edwcore (podpora.m-line.cz, mlweb). Tvrzení o legacy systému a realističnost slibů ověřovalo šest subagentů přímo v kódu; citace soubor:řádek níže pocházejí z jejich evidence a namátkou jsem je přeověřil. Kompilace: 2x pdflatex do `build/r3`, bez chyb, 18 stran, žádné duplicitní destinace, jeden underfull hbox v tabulce harmonogramu. Screenshoty prohlédnuty v originálním rozlišení i v sazbě.

Nereportují se schválené odchylky a vědomá rozhodnutí ze zadání (harmonogram z perspektivy 1. 6. 2026, vyřazení licenčního nástroje, tray a zprávy, plovoucí mini-apps, DEMO licence, N4 bez metriky, průběžný developer feedback, chybějící screenshot update promptu, žádný CI server, kategorie nice-to-have, 78 MD). Nereportuje se ani to, co v EDISON2 zatím není implementované: specifikace implementaci předchází. Nálezem je jen nepravda, nerealističnost nebo vnitřní rozpor.

**TL;DR:** Dokument je po dvou kolech oprav v dobrém stavu: faktický obraz legacy systému drtivou většinou sedí na kód, struktura odpovídá šabloně, forma je čistá. Zbývají tři věci, které by komise nebo konzultant našli sami: (1) sekce Changes nevyjmenovává všechny odchylky od záměru (zúžený testovací závazek, zúžená rekonstrukce updateru), (2) licenční outcome nemá milník, man-days ani evaluační kritérium, ačkoli je jedním ze šesti slibovaných výsledků, a (3) screenshoty v Mockups ukazují jiné věci, než tvrdí jejich popisky (status bar s „Verze nezjištěna" a začerněným uživatelem, prázdné panely, šedé bloky místo dat, reálný zákazník CB_DP). K tomu několik věcných nepřesností v požadavcích (M6 „první přímý kanál" není první, T1/N10 slibují architekturu opačnou k té, kterou toolkit má zdokumentovanou jako konvenci, N13 s chybným zdůvodněním) a slabá měřitelnost NFR, kde úvodní věta slibuje metodu kontroly u každého požadavku, ale mají ji dva.

Žádný nález nepovažuji za blocker: dokument jde odevzdat po opravě majorů, které jsou všechny lokální.

---

## Nálezy

### MAJOR

**1. Sekce Changes není úplná: záměr slibuje víc, než spec u testů a updateru drží, a odchylky nejsou přiznané.**
Místo: sekce 9 (ř. 559-571); intro ř. 68 („The expected outcomes are those committed to in the approved project proposal"); N3 (ř. 292); U1 (ř. 258); Architecture (ř. 340).
Co je špatně: Záměr (project-proposal.tex:65) slibuje „all public APIs in the toolkit and shell ship with unit tests". Spec z toho dělá N3 „at least 80 % of the non-visual types". To je rozumné zeslabení, ale je to odchylka od schváleného textu a v Changes není. Druhá: záměr (ř. 68) říká u updateru „Most likely a full recreation", spec drží legacy apply engine (U1 „may be carried out by the existing update engine", Architecture „The low-level apply engine may stay") a rekonstruuje jen koordinační logiku. Záměr má slovo „most likely", takže to není porušení, ale směr se změnil a intro dál slibuje „Update mechanism modernization on both the client and the server side" bez upřesnění. Třetí, drobná: „around thirty agendas" v záměru versus „around forty" ve spec (popisné číslo, stačí věta).
Proč to vadí: Sekce Changes vznikla právě proto, aby čtenář nemusel dokumenty porovnávat sám. Review 2 (nález 5) na neúplnost upozorňovalo, a oprava zavedla sekci se dvěma položkami, kde chybí ta nejsnáz odhalitelná (testy).
Návrh opravy: Do sekce 9 přidat odstavce „The testing commitment was made measurable" (all public APIs -> 80 % netriviálních typů + každá Caché třída s logikou) a „The update engine may stay" (rekonstruuje se koordinace, download/apply může zůstat legacy; klientská i serverová strana se mění, ale ne celé). Volitelně větu o počtu agend.

**2. Licenční outcome nemá milník, man-days ani evaluační kritérium.**
Místo: intro bullet „Licensing configuration" (ř. 73); Milestones (ř. 484-492, „The work is split into six milestones"); tabulka harmonogramu (ř. 524-541); text harmonogramu ř. 519 („Licensing and the update mechanism follow once the foundation is stable, partly in parallel"); Evaluation (ř. 447-457).
Co je špatně: Intro slibuje šest outcomes, milníků je šest, ale licence mezi nimi není (MS1 testy, MS2 toolkit, MS3 shell, MS4 mini-apps, MS5 update, MS6 pilot). L2 je přitom netriviální práce: `EDISON\EDCORE\Resources\edisontree.xml` má přes 3100 řádků a 917 uzlů se sémantikou `hidden`, `mlineOnly`, `input`, `state`, a legacy shell i legacy licenční nástroj ho čtou přes `ALVA.CORE.ApplicationTree.ToXmlEx` (`EDISON\EDCORE\VyjmutoEDISONApi.vb:316-345`, `EDISON\Licence\EDLicence\CTrees.vb:138-160`). Migrace do Caché „replacing the manual XML maintenance" plus zachování funkčnosti čteček XML není práce, která se schová do MS3 „Login, module tree, legacy bridge". Text harmonogramu licenci zmiňuje, tabulka pro ni nemá řádek, a Evaluation má tři pilíře (testy, pilot, developer feedback), z nichž žádný L2 neověřuje. Stejně tak mini-app outcome (M5) nemá evaluační kritérium, jen demonstraci na obhajobě.
Proč to vadí: Komise čte Milestones a Evaluation jako kontrolní seznam. Outcome, který není v žádném z nich, vypadá buď zapomenutý, nebo jako věc, kterou si autor nechává prostor nesplnit.
Návrh opravy: Buď řádek „Licensing configuration: module definitions moved to Caché, XML readers kept working" s vlastními MD (kapacita je: viz nález 12, listopad a prosinec jsou v tabulce podobsazené), nebo L2 výslovně přiřadit do MS3 v popisu milníku a v tabulce. Do Evaluation přidat kritérium pro L2 (pilotní instalace běží nad stromem z Caché, legacy shell na téže instalaci funguje dál) a pro M5 (demonstrativní mini-app v denním používání u vývojářů / na pilotu).

**3. Úvodní věta NFR sekce je nepravdivá: „Each non-functional requirement states how it will be checked."**
Místo: ř. 288 a N1-N14.
Co je špatně: Metodu kontroly uvádějí jen N3 (statické měření + hook) a N6 („measured side by side on the same workstation"). N1, N2, N4, N5, N7-N14 říkají, co má platit, ne jak se to ověří. Například N8 „themes are consistent", N12 „never freezes for more than two seconds", N13 „an unhandled error in one window does not terminate the client", N14 „operable by keyboard" nemají ani náznak postupu.
Proč to vadí: Věta zve k tomu, aby si oponent NFR projel jeden po druhém. Slib, který nedrží u 12 ze 14 položek, podkopává i ty dvě měřitelné.
Návrh opravy: Buď větu smazat, nebo ke každému NFR přidat půlvětu s metodou (N1: pilotní web s oběma shelly současně + test zámků; N2: instalace obou na jedné stanici; N12: profiler / sampling během typické seance; N13: injektovaná výjimka v testovacím okně; N14: průchod bez myši po vyjmenovaných scénářích; N8: theme audit toolkitu).

**4. Mockups: popisky tvrdí něco jiného, než je na screenshotech vidět, a screenshoty nesou reálný zákazník a vývojářský režim.**
Místo: fig:mock-login, fig:mock-main, fig:mock-miniapps (ř. 306-326), soubory img/*.png.
Co je špatně, bod po bodu:
(a) Popisek hlavního okna: „The status bar shows the company, the user, and the client version (S17)". Na obrázku status bar ukazuje `CB_DP`, dva šedé obdélníky (začerněné jméno uživatele a druhá hodnota) a text „Verze nezjištěna", tedy verze klienta nebyla zjištěna. S17 navíc slibuje viditelnou expiraci licence, která na obrázku není.
(b) Popisek mini-apps: „both showing agenda data through the toolkit's data list (T2)". Datové oblasti obou panelů jsou souvislé šedé bloky (začerněná data), žádná data vidět nejsou. A panely nejsou zobrazené datovým listem toolkitu, ale obyčejným WPF `DataGrid` (`EDISON2\EDISONMAIN2.LegacyMiniApps\dotnet\LegacyDashboardMiniAppView.xaml:17,21`), takže popisek i M7 („displayed with the toolkit's data list") popisují něco, co obrázek nepotvrzuje.
(c) Popisek loginu: „The three panels next to the form are the built-in mini-apps with company news, the user wiki, and the training offer". Panely jsou prázdné, jen s textem „Novinky z portálu podpory" apod. Je to záměrný anonymizační režim (`EDISON_SCREENSHOT_MODE`, `EDISON2\EDISONMAIN2\dotnet\MiniApps\SupportWidgetHost.cs:9-14`), ale popisek to neříká a čtenář vidí tři prázdné plochy přes dvě třetiny obrazovky.
(d) Firma „CB_DP - DPmCB" v titulku okna, v combu i ve status baru je reálný zákazník (Dopravní podnik města České Budějovice). Dokument bude ve fakultním repozitáři; jestli firma souhlasí, ať to autor ví, jestli ne, použít testovací firmu.
(e) V menu je položka „Test funkcí", podle komentáře v XAML „dev-only nadstavba" (`EDISON2\EDISONMAIN2\dotnet\Windows\EdisonMainWindow\EdisonMainWindow.xaml:69,173`). Strom ukazuje uzel „Online aplikace", který je v legacy XML `hidden="1"` (`EDISON\EDCORE\Resources\edisontree.xml:1155`). Obojí říká, že screenshoty jsou z vývojářského účtu, což dokument nezmiňuje, a u „Online aplikace" to zároveň zpochybňuje L2 („The migration preserves the semantics of the existing definitions (hidden ...)"), pokud nejde o účet v doméně M-line.
(f) Datum „Dnes je 27.08.2026" v pravém horním rohu je v pořádku (schválená odchylka), ale spolu s (e) dává komisi jasný signál, že jde o běžící implementaci, ne mockup. Pak je fér to říct.
Proč to vadí: Mockup sekce je jediné místo, kde komise vidí produkt. Popisek, který slibuje verzi a uživatele, zatímco obrázek ukazuje „nezjištěna" a šedé pruhy, působí jako nepozornost; reálný zákazník je věc firemního souhlasu.
Návrh opravy: Screenshoty pořídit s testovací firmou a fiktivním uživatelem (např. `jan.novak`, který už je na loginu), s detekovanou verzí a s daty (testovací databáze), nebo v popiscích přiznat anonymizaci („data and identities are blanked out for this document"). Odstranit dev menu, nebo o něm říct větu. Zvážit výřezy místo celé obrazovky 2560 px (viz nit 25).

**5. M6 tvrdí, že novinky jsou „the company's first direct channel to the end users". Legacy shell tenhle kanál už má.**
Místo: M6 (ř. 224), login caption, Project description „Mini-apps" (ř. 173).
Co je špatně: Legacy shell načítá tytéž widgety z podpora.m-line.cz jako nový shell (`EDISON\EDISONMAIN\frmMain.vb:954-958` vs. `EDISON2\EDISONMAIN2\dotnet\MiniApps\WidgetUrls.cs:16-25`, server `edwcore\apps\mlweb\backend\src\routes\widgets.ts:26-50`). Legacy má navíc vlastní informační panel se systémovými hlášeními (nová verze, expirace licence, `frmMain.vb:832-871`). „First direct channel" tedy není pravda. Dále: M6 popisuje mini-app, ale v hlavním okně je to pevný panel nad hosting area, mimo dock (screenshot; `EdisonMainWindow.xaml:187-191`; interní pravidlo „Widgety podpora.m-line.cz v docku nejsou"). M1 přitom definuje mini-apps jako věci v hosting area pod správou shellu a M3 dělá z M6 „jedinou výjimku" z uživatelské volby. A serverová strana widgetu je dnes „Info widget (configurable links)" (`widgetController.ts:61-66`), tedy odkazy konfigurované podporou, ne „news written by M-line staff on the company web".
Proč to vadí: Konzultant i kolegové vědí, že widgety v legacy jsou. Věta „first" je zbytečně napadnutelná a M6 jako celek popisuje jinou věc, než jaká se staví (a než jakou ukazují screenshoty).
Návrh opravy: M6 přeformulovat: „A permanently visible panel with company news and announcements, taken over from the legacy shell, which the user cannot remove." Vypustit „first direct channel" (nebo „the first channel the company controls end to end", pokud to platí). V M3 výjimku vztáhnout na „the permanently visible news panel", ne mini-app. Zdroj novinek popsat tak, jak bude (kdo píše, kde), viz otázka 3.

**6. T1 a N10 slibují architekturu, kterou toolkit má zdokumentovanou opačně.**
Místo: T1 (ř. 271, „keeping view models free of UI-framework references"); N10 (ř. 299, „explicit and strongly typed dependency injection as the default for new code"); Project description „AI-augmented development" (ř. 176, „preference for explicit and strongly typed dependency injection over hidden global states").
Co je špatně: Základní view-model toolkitu referencuje `System.Windows`, `System.Windows.Input`, `System.Windows.Media` a má metodu `AttachToWindow(Window)` (`EDISON2\EDControls\dotnet\EDViewModel\EDViewModel.cs:3-5`); knihovna je `UseWPF=true`; testy fungují jen proto, že testovací projekty jsou WPF-enabled. Žádný DI kontejner není, statický locator `EDServices.Current` má 49 volání ve 24 souborech a kořenový CLAUDE.md ho předepisuje jako konvenci („controls read EDServices.Current themselves"). Toolkit má ~1900 xUnit testů postavených na tomto designu. Spec tedy jako essential požadavek slibuje přestavbu, kterou nikdo neplánuje a která nemá rozpočet. Review 1 (nález 11) N10 změkčilo na „as the default for new code", ale konvence repa říká opak i pro nový kód.
Proč to vadí: Nejde o chybějící implementaci (ta by nálezem nebyla), ale o slib, který je v rozporu se zvolenou a zdokumentovanou cestou. Supervizor, který uvidí kód, se zeptá, proč spec říká něco jiného.
Návrh opravy: T1: „A view-model base with command creation, unified error presentation, and backend access, testable without a running UI (no code-behind logic, no live window needed in tests)." N10: „explicit, typed dependencies; access to shared services through one documented, test-replaceable seam" a stejně přeformulovat větu v AI-augmented development. Pokud autor chce čistý VM assembly opravdu dodat, patří to do rozsahu s MD.

**7. Rozsah update outcome versus 12 MD: U3 a U7 mají proti sobě celý legacy řetězec.**
Místo: U3 (ř. 260), U7 (ř. 264), U8-U10, intro „on both the client and the server side", tabulka MS5 (12 MD).
Co je špatně: Legacy distribuce je řetězec Compiler.exe -> `ALVA.Compiler.Function` (běží jen na Cloud2, není v repu) -> SETUPWIZARD -> IIS `http://release.m-line.cz:59993` -> EDTRANSFER2 -> EDISONINS -> EDUPDATE -> EDLOADER, s ručně editovaným manifestem `FILES_DESCRIPTION.INF` na release serveru (`EDISON\DOCS\08-nasazeni.md` §8.4, 8.9; `EDISON\Compiler\CompilerApi\SetupWizard.cs:30-51`). HTTPS v něm není nikde, jen `PathIsUri` ho toleruje (`EDISON\EDISONMAIN\clsUpdate.vb:204-206, 1010`). Dnes je z nového stacku v manifestu jen `EDISONMAIN2API.dll` (`EDISON\FILES_DESCRIPTION.INF:56`); shell, EDControls ani Caché projekt `ALVA_EDISONMAIN2` tam nejsou. U3 „served from the same pipeline" a U7 „supports secure transport (HTTPS)" tak znamenají zásah do release serveru, manifestu, EDTRANSFER2 a EDUPDATE, a k tomu U8 (odmítání loginů, které je v legacy zakomentované: `EDISON\EDISONAPI\clsUpdater.vb:2610-2616`), U6 s cílem na company web, U9, U10. Za 12 MD, spolu s koordinační logikou z U1/S9.
Proč to vadí: Je to jediný outcome, kde spec nemá ve svém prospěch existující práci (nový update kód v EDISON2 není, klient volá legacy EDUPDATE: `EDISON2\EDISONMAIN2\dotnet\Startup\Updater.cs:117-147`). Review 1 (nález 4) na napětí slib/rozpočet upozorňovalo, od té doby U3 a U7 přibyly a rozpočet klesl ze 14 na 12.
Návrh opravy: Buď U3 a U7 (HTTPS část) označit nice to have s vysvětlením, nebo přesunout MD z podobsazeného listopadu a prosince (viz nález 12) a v U3 vymezit minimum („the release manifest and the server-side installer carry the new client's files; the transport stays as it is unless time allows").

**8. C4 context diagram vynechává externí systémy, na kterých stojí essential požadavky.**
Místo: fig:c4-context (ř. 97-141), fig:c4-container caption (ř. 431).
Co je špatně: Klient mluví s podpora.m-line.cz (widgety M6 a login panely přes WebView2, error reporty T9 na `https://podpora.m-line.cz/api/telemetry/ingest`, support panel T11), s release serverem M-line (`VERZE.INF` přes HTTP pro U5/U9: `Updater.cs:161`) a zákaznický backend tahá licenci z centrálního archivu M-line (`release.m-line.cz:57772`, `EDISON\EDISONMAIN\CACHE\ALVA\EDISON\MLAPI.cls:31`). Context diagram ukazuje jen zákaznickou Caché. Caption containeru přiznává vynechané update balíčky, ostatní ne. Chybí také persony administrátor (L3, L4, U8, U10) a vývojář (celá T skupina).
Proč to vadí: Context diagram v C4 existuje právě proto, aby ukázal všechny systémy, se kterými systém komunikuje. Tři externí závislosti přes internet z každé stanice jsou pro komisi relevantní (offline chování, závislost na M-line).
Návrh opravy: Do contextu přidat jeden šedý box „M-line services (support portal, release server, license archive)" se šipkami z EDISON 2, legacy a zákaznického backendu, a persony Administrator a Developer (nebo alespoň větu v caption, proč jsou vynechané).

### MINOR

**9. Favorites versus desktop: granularita persistence si odporuje a screenshot ukazuje, že jde o jedna data.**
Místo: S5, S7 („Per-user settings (favorites, last company)"), S13 („persisted per user and company"), fig:mock-miniapps caption („shortcut tiles for the modules the user marked as favorites in the tree (the stars)").
Co je špatně: Popisek i hint na ploše („přidejte přes ★ v kontextovém menu") říkají, že dlaždice plochy jsou oblíbené moduly. S7 ukládá oblíbené per user, S13 dlaždice per user a firma. Legacy ukládá oblíbené (`fav_item_*`) per user a IdFa (`EDISON\EDISONMAIN\CACHE\ALVA\EDISON\Main.cls:1054-1062`). Review 2 (nit 20) vedlo k zúžení S7 na „per user", což vytvořilo tento nový rozpor.
Návrh opravy: V S7 dát favorites per user a firma (jako v legacy), nebo v S13 říct, že plocha zobrazuje oblíbené (a pak S13 nepotřebuje vlastní persistenci).

**10. U2 a U7 tvrdí o legacy chování věci, které se čtou jako protichůdné.**
Místo: U2 („Server-only changes are tolerated by the version convention, as in the legacy system"), U7 („fixes two known failures of the legacy mechanism: ... a Caché-only patch never forces running clients through the update cycle").
Co je špatně: Obojí je zvlášť pravda, ale o různých mechanismech: runtime guard porovnává jen segmenty 1 a 2 verze a patch toleruje (`EDISON\MLRest\MLRest\CACHE\ALWA\SYS\Rest.cls:1700-1725`), zatímco klientský updater při startu vynutí update na jakoukoli novější `VERZE.INF` (`clsUpdate.vb:663-668`, ForceYes `frmNoMDI.vb:4446`) a noční `RunAutoUpdate` ukončuje klienty bez ohledu na TYPE (`EDISON\EDISONMAIN\CACHE\ALVA\EDISON\System.cls`, RunAutoUpdate). Čtenář bez tohoto kontextu vidí „as in legacy" a „legacy failure" o téže věci. Review 2 (nález 11) navrhlo dnešní znění U7; tato nejasnost je vedlejší efekt.
Návrh opravy: V U7 upřesnit: „a Caché-only patch is picked up at the next start and never forces running clients out, unlike the legacy nightly update".

**11. S6 vyjmenovává, co bridge předává, a chybí stav licence, na kterém stojí L1 read-only.**
Místo: S6 („passes the established session, company context, and the user's licensed permissions"), L1 („After the license expires, the installation stays read-only, as in the legacy system").
Co je špatně: Read-only režim je v legacy počítaný na klientu z stavu licence (`MLProps.ReadOnlyMode = E04 OrElse E03`, `EDISON\EDISONMAIN\Rozhrani (mainform)\frmNoMDI.vb:1258-1259`) a vynucovaný každým formulářem zvlášť (přes 470 souborů referencuje `ReadOnlyMode`). Aby legacy agenda spuštěná z nového shellu zůstala read-only, musí jí bridge stav licence předat. S6 to nezmiňuje a `EdisonLaunchArgs` dnes žádný takový příznak nenese (`EDISON2\EDControls\dotnet\EdisonLaunchArgs.cs:7`). Bez toho L1 „as in the legacy system" pro agendy spuštěné novým shellem neplatí.
Návrh opravy: Do S6 přidat „and the license state (including the read-only mode after expiry)".

**12. Harmonogram po měsících nesedí na kapacitu 2 dny v týdnu a nemá rezervu.**
Místo: tabulka ř. 524-541, text ř. 519.
Co je špatně: Kapacita je asi 8,7 MD na měsíc. Červen až srpen: MS1 6 + MS2 17 = 23 z 26, dobře. Srpen až říjen: MS3 17 + MS4 6 + začátek MS5 při zbylých ~3 MD ze srpna dává ~23 MD proti ~20 dostupným. Listopad a prosinec: MS5 zbytek ~8-12 MD proti 17,4 dostupným (rezerva 5-9 MD, nikde nepojmenovaná). Leden a únor: MS6 5 + Documentation 10 + Evaluation 5 = 20 proti 17,4, a projekt končí 1. 3. bez jakékoli rezervy. Součet 78 sedí, rozložení ne.
Návrh opravy: Posunout MS5 na 11-12/2026 a uvolněný říjen dát MS3/MS4, nebo přidat řádek pro licenci (nález 2) do listopadu a prosince. Do textu větu o tom, kde je rezerva (nebo že žádná není a proč je to přijatelné).

**13. N13 zdůvodňuje požadavek nesprávně: „so no orphaned process holds a license slot".**
Místo: N13 (ř. 302).
Co je špatně: Licence se počítají per stanice (IP) a firma, volající IP se nepočítá (`EDISON\MLRest\MLRest\CACHE\ALWA\SYS\Activity.cls:58-72`; `EDISON\EDUsers\EDUsers\CACHE\ALVA\SYS\Login.cls:187-208`). Osiřelý proces agendy na téže stanici tedy žádný další slot nedrží. Skutečná cena osiřelého procesu jsou zámky záznamů (N1) a neuvolněná aktivita. Navíc slot drží i tvrdě ukončený shell sám, dokud server nevyprší timeout (interní dokumentace EDISON2, sekce Session), takže „no orphaned process holds a license slot" nechrání ani před tím.
Návrh opravy: „...so no orphaned process keeps record locks or a stale session alive."

**14. N4 a Architecture si odporují u legacy agend a Architecture udává důvod výjimek, který evidence nepotvrzuje.**
Místo: N4 („adding a new agenda requires only its registration, not changes to the shell"), Architecture (ř. 334, „Some agendas need specific exceptions in the host because of their age and the versions of standard components they use").
Co je špatně: Pro legacy agendy N4 neplatí z definice (Architecture to říká). Skutečné výjimky v hostu se týkají životního cyklu formulářů a propagace práv (ISYJR zavírá sám sebe a otvírá ISYJR2, propagace práv pro ISYJR2/ADDISPECING, EDISONMAIN.exe spouštěné mimo host, `EDISON\EDISONMAIN2API\FormToRunGetter.cs:133-137,170-173`, `Program.cs:34-38`), ne verzí standardních komponent.
Návrh opravy: N4 omezit na „a new agenda built on the new stack"; v Architecture „because of the way some of them manage their own windows and rights".

**15. N5 a Architecture řadí modulový strom mezi „behavior that changes per deployment or per customer", L2 z něj dělá vývojářský kód shodný pro všechny.**
Místo: N5 (ř. 294), Architecture (ř. 340), L2 (ř. 247).
Co je špatně: L2: „Developers maintain them as Caché code in the development environment, and the definitions reach customer installations with releases like any other code." To není per-customer konfigurace, to je kód s jiným distribučním kanálem. N5 pak slibuje „lives in a database, not in the client binaries, so changing it does not require redistributing clients", což pro strom platí jen do té míry, do jaké Caché-only release nevyžaduje klienty (viz U7). Dvě různá zdůvodnění téže věci.
Návrh opravy: V N5 a Architecture oddělit „per-customer data (licensing, the inactivity limit)" od „shared definitions that must change without a client release (the module tree)".

**16. L2 neříká, jak se během přechodu udrží XML a Caché definice v souladu.**
Místo: L2 („the tools that read the XML keep working until they are replaced").
Co je špatně: Legacy shell čte XML embedované v `ALVA.CORE.dll`, legacy licenční nástroj také (nález 2 evidence). Po migraci existují dva zdroje a spec neříká, který je autoritativní a jak vzniká druhý. Dnes to je řešené generátorem opačným směrem (Caché strom generovaný z XML, `EDISON2\tools\GenerateTree.cs`, `sync-tree.ps1`, drift hook), což je opačná polarita, než L2 slibuje („move ... into the Caché backend").
Návrh opravy: Jedna věta: „During the transition one of the two forms is generated from the other, so that no definition is maintained twice."

**17. L1 „non-blocking warnings for approaching limits" nemá v legacy oporu a není definované, které limity a odkud.**
Místo: L1 (ř. 246).
Co je špatně: Legacy varuje před expirací v informačním panelu (`EDISON\EDISONMAIN\frmMain.vb:848-871`) a u limitů RZ/ZAM (`frmMain.vb:874-903`), ale žádné varování před dosažením limitu souběžných loginů nemá; hlídá jen tvrdé překročení (`Login.cls:187-211`). Počet aktivních stanic zná server, klient dostává jen ano/ne. „Approaching limits" je tedy nová funkce vyžadující serverovou změnu (počet volných slotů v odpovědi), a věta to neříká.
Návrh opravy: „...shows non-blocking warnings for approaching expiry and, where the backend reports it, for approaching numeric limits."

**18. T10 „sends the aggregate ... usage counts" versus telemetrie per událost s časovým razítkem.**
Místo: T10 (ř. 280).
Co je špatně: Spec slibuje agregát a počty. Toolkit posílá jednotlivé záznamy `(timestamp, form, control)` pod session uživatele (`EDISON2\EDControls\dotnet\EDViewModel\EDUsageTelemetry.cs:15,73-74`). Buď je spec cílový stav (pak je to změna proti hotovému a stojí MD), nebo je nepřesná. Rozdíl je i z hlediska ochrany osobních údajů: časová řada per uživatel není „aggregate".
Návrh opravy: Rozhodnout a napsat: „records which windows and controls are used and sends usage events (window, control, time) without the content of the user's work; the backend aggregates them".

**19. S6 slibuje spouštění new-stack aplikací „with the same context", druhý konzument toolkitu si kontext vědomě nebere.**
Místo: S6 (ř. 190), Architecture (ř. 340, „agendas and mini-apps it hosts reuse the logical session").
Co je špatně: TemplateDesigner úmyslně ignoruje shellGuid a drží vlastní session (`EDISON2\TemplateDesigner\dotnet\Config\CacheSchemaWiring.cs:14-24`), shell ho spouští bez argumentů („nesdílí shell session", `EdisonMainViewModel.cs:1336-1342`). Review 1 (nález 14) omezilo pravidlo na hostované věci; S6 ho pro spouštěné new-stack aplikace znovu zavádí. Není to chyba, pokud je to cíl, ale je to cíl, který jde proti rozhodnutí kolegy, se kterým se má toolkit ladit.
Návrh opravy: Buď v S6 „can launch ... with the same context" (nabídka, ne povinnost), nebo TD zmínit jako výjimku.

**20. Consulting plan neuvádí délku schůzek, kterou šablona výslovně chce.**
Místo: 8.1 (ř. 548).
Co je špatně: Šablona: „Explain the expected time frame for team meetings, that is, how often and how long. You can also comment on the expected content." Spec má frekvenci, ne délku ani obsah.
Návrh opravy: „...meet roughly once a month for about an hour to go over the report, open questions, and the next milestone."

**21. Formulace „not as an opt-in, at least for a time" v rizicích je nesrozumitelná.**
Místo: Risks, „Problems at the pilot site" (ř. 466).
Co je špatně: Není jasné, co platí „at least for a time": že web nemá možnost volby, nebo že po nějaké době dostane. Zároveň věta „The pilot does not depend on a customer volunteering" hned vedle „The real risk is an issue severe enough to send the site back" dává komisi otázku, zda pilotní zákazník o pilotu vůbec ví.
Návrh opravy: „The company deploys the new shell to a selected site as part of a regular update; the site is informed, but the switch is not optional for its users."

**22. S19 (startovací modul) je nice to have, ale v legacy shellu tahle funkce existuje.**
Místo: S19 (ř. 208), úvod FR („may be reduced under time pressure").
Co je špatně: Legacy má „nastavit startovací formulář" v kontextovém menu stromu, spouští ho po přihlášení a ukládá jako `param_form` v uživatelském nastavení (`EDISON\EDISONMAINrmMain.vb:2836-2878, 1985-1986`; `frmNoMDI.vb:2213`). Nice-to-have status znamená, že nový shell smí uživatelům funkci legacy shellu vzít, a spec nikde neříká, které funkce legacy shellu se smějí ztratit (pilotní kritérium hlídá jen spustitelnost agend). Pro rozhodnutí supervizora o nice-to-haves (červené TODO) je to podstatná informace: S19 je parita, S20 a S21 novinky.
Návrh opravy: S19 přesunout mezi essential (je to malá věc), nebo v úvodu FR říct, že nice-to-have položky s předlohou v legacy shellu se dodají, pokud je pilotní web používá.

**23. Několik funkcí legacy shellu nemá ve spec požadavek ani zmínku v Out of scope.**
Místo: Functional requirements, Out of scope (ř. 179).
Co je špatně: Out of scope vyjmenovává agendy, licenční nástroj a rollout. Legacy shell ale nese i tyhle věci, o kterých spec mlčí: správa zámků pro administrátora (`formUSERSLocks`, menu Nástroje, `frmNoMDI.vb:3399-3651`), více připojovacích profilů (`frmConnection`, registry profily, `frmNoMDI.vb:500-527, 2025-2095`; S10 mluví o jednom připojení), spuštění druhé kopie shellu pod jiným uživatelem („Spustit další EDISON", `frmNoMDI.vb:3624-3635`), nápověda z HTM souborů (`frmNoMDI.vb:3769-3787`), licencovaná lokalizace UI (`ALVA.EDISON.Localization`, právo `AllowLocalization`, uzel 20000005, `frmNoMDI.vb:3561-3564`). U tray aplikace a zpráv je mlčení vědomé; u těchto položek nevím, a čtenář se to nedozví.
Návrh opravy: Rozhodnout a napsat: buď věta v Out of scope („The following legacy shell functions are not carried over: ..."), nebo požadavky (správa zámků patří logicky k L3, více připojení k S10).

**24. M8 u legacy panelů: „refresh at a chosen interval" nemusí přinést nová data.**
Místo: M8 (ř. 226) ve vazbě na M7.
Co je špatně: Legacy panely nemají klientský timer; data předpočítává serverový task (`ALVA.EDISON.Task`, typ MiniAppRefresh -> `MiniAplikaceAPI.PrepareData`, `EDISON\EDISONMAIN\CACHE\ALVA\EDISON\Task.cls:26-28`) a klient jen čte výsledek s časovým razítkem (`EDMiniApplication.vb:739-740`). Klientský interval z M8 tedy u M7 panelů znovu načte tutéž předpočítanou tabulku, pokud se zároveň nevyvolá přepočet (který byl podle interní dokumentace EDISON2 hlavním zdrojem pomalého startu).
Návrh opravy: V M8 nebo M7 jedna věta: „for the legacy panels, refresh reloads the server-prepared data; the recomputation stays with the server task."

### NIT

**25. Čitelnost screenshotů.** 2560 px na 0,88 šířky textu (~15 cm) dává popiskům stromu asi 4 pt v tisku; na s. 11 jsou dva screenshoty pod sebou a text v nich nejde přečíst. Výřezy (levý strom + hosting area) nebo jeden screenshot na stranu.

**26. Zelený úchyt na pravém okraji všech tří screenshotů** (výsuvný panel) není nikde vysvětlený.

**27. Sazba.** S. 11 obsahuje jen dva obrázky s popisky, s. 13 jen container diagram, s. 18 je zaplněná z třetiny (sekce Changes by se vešla na s. 17, kdyby se zkrátila tabulka). V tabulce harmonogramu se dělí „infrastruc-ture" a hlavička „Man-/days" (underfull hbox, ř. 524-525); šířka prvního sloupce 11em je ze šablony, ale „MS1 Testing infrastructure" se do ní nevejde; stačí `\raggedright` nebo kratší název.

**28. Platform „communication over HTTP"** a C4 „Queries [HTTP]" vedle U7, které pro update chce HTTPS: přihlašovací jméno a heslo jdou v legacy přes REST po HTTP (`EDISON\EDUsers\EDUsers\EDUsers.vb:141`). Oponent se zeptá, proč se zabezpečuje kanál pro update a ne kanál s hesly. Stačí věta („the data channel runs inside the customer's network; the update channel crosses the internet").

**29. S15 neříká, jak se Windows identita ověřuje.** Legacy LDAP-AUTO pošle jméno účtu OS s prázdným heslem (`EDUsers.vb:128-130`, `Login.cls:1225-1230`), tedy server věří klientem poslanému jménu. Pokud S15 míní totéž, je to slabé místo, které by v NFR Security mělo zaznít; pokud míní skutečné SSO, je to nová práce.

**30. „The proposal was approved in June 2026."** Git: poslední věcná změna záměru 11. 6. 2026 („2nd iter"), commit „final version" až 12. 8. 2026. Pokud schválení proběhlo v červnu, věta sedí; jen ať to autor umí doložit.

**31. Evaluation developer study: „keep the developer workflow close to the legacy one, so the switch costs the colleagues as little as possible"** stojí vedle cíle „establishing a test-first culture absent from the legacy stack". Obojí je legitimní, ale bez věty o tom, co se má změnit a co zůstat, to zní jako protimluv.

**32. Project description „Each user chooses in their settings which mini-apps to display"** versus M3 „adds and removes them directly in the shell". Sjednotit („directly in the main window").

**33. Risk „Update mechanism underestimated"**: mitigace „the first step adapts the legacy updater to also serve the new shell" je z perspektivy 1. 6. správně, ale spolu s U1 a Architecture je to už třetí místo, kde se říká, že engine zůstává. Jedno by stačilo.

---

## Otázky na autora

Věci neověřitelné z rep, nebo věci, kde kód a spec říkají různé věci a nevím, co je záměr:

1. **CB_DP / DPmCB na screenshotech.** Souhlasí firma s uvedením zákazníka v dokumentu, který skončí ve fakultním repozitáři? Začerněné jméno uživatele: nahradit fiktivním účtem?
2. **Layout mini-apps a plochy: per uživatel a firma (M11, S13, S7), nebo per stanice a firma?** Kód ukládá layout do `%APPDATA%\m-line\EDISONMAIN2\local.json` per stanice × IdFa (`LocalPrefs.cs:48`), dlaždice plochy do Caché per uživatel × IdFa. Dva uživatelé na jedné stanici dnes sdílejí layout. Co je cíl?
3. **Zdroj novinek pro M6.** Kdo bude novinky psát a kde (podpora.m-line.cz jako dnes, firemní web, něco nového)? Dnešní widget jsou konfigurované odkazy podpory.
4. **Bridge a práva.** Host nastavuje pro každou spuštěnou legacy agendu `UserIsAdmin = true` (`EDISON\EDISONMAIN2API\FormToRunGetter.cs:118`). Je to dočasné? S6 slibuje předání „the user's licensed permissions" a L4 „administrator rights never bypass the license".
5. **Read-only po expiraci u agend spuštěných z nového shellu** (nález 11): počítá se s předáním stavu licence bridgem, nebo se to má řešit jinak (server)?
6. **N3 metodika.** Které typy se počítají jako „non-visual" (DTO, enumy, records, generovaný kód?), jak se poměr měří (skript?), a co je „Caché class with logic"? Bez toho jde 80 % posunout definicí. Hrubý odhad z repa: 121 ze 145 souborů má typ zmíněný v nějakém testu (83 %), ale „zmíněn v testu" není „má unit test".
7. **N6 definice měření.** Start shellu od čeho po co (dvojklik -> login okno? -> hlavní okno po přihlášení?), s update checkem nebo bez, kolik opakování; spuštění agendy první (studený host) versus opakované.
8. **Telemetrie (T10):** má být agregát, nebo per událost jako dnes? Je opt-out uživatele záměrně mimo spec (v kódu je jen statický přepínač bez UI)?
9. **S15:** skutečné SSO (Kerberos/SSPI), nebo důvěra ve jméno OS účtu jako v legacy LDAP-AUTO?
10. **Windows 10 (N9)** skončil podporu v říjnu 2025. Podporuje ho firma dál (ESU), nebo stačí „Windows 11, Windows 10 where still deployed"?
11. **Pilot a licence/update.** Projde pilotní web do 1/2027 stromem z Caché (L2) a updatem novým mechanismem? Evaluation to tvrdí u updatu, u L2 mlčí.
12. **„The other agendas in the author's care are in maintenance mode only"** (Risks): neověřitelné z rep, přenáší se jako otázka.
13. **Uzel „Online aplikace" na screenshotu**: vývojářský účet v doméně M-line, nebo se `hidden` v novém stromu nerespektuje?
14. **Datum schválení záměru** (nit 30).
15. **Používá některý zákazník licencovanou lokalizaci UI** (právo `AllowLocalization`)? Pokud ano, N7 „full multi-language support is not a goal" je regrese proti legacy a patří do Changes nebo Out of scope (nález 23).

---

## Co prošlo bez nálezu

Struktura a forma:

- Struktura sekcí 1:1 se šablonou fakulty (`af8ff32`), hlavičková tabulka kompletní (start a konec navíc, jako v záměru), titul „Specification of company software project" konzistentní se záměrem.
- Kompilace bez chyb a varování mimo jeden underfull hbox v tabulce; PDF/A metadata (xmpdata: Title, Author, Subject, Keywords) přítomná a shodná s titulem; 18 stran; duplicitní destinace z 2. kola vyřešené.
- Všech 18 `\req{}` odkazů vede na existující `\reqdef`; žádné duplicitní ID; nice-to-haves důsledně na konci skupin (S19-S22, L5-L7, U11, T14); číslování souvislé (S1-S22, M1-M11, L1-L7, U1-U11, T1-T14, N1-N14, celkem 79 požadavků, 9 nice to have).
- Man-days 6+17+17+6+12+5+10+5 = 78; 78 MD / 39 týdnů = 2 dny v týdnu, sedí na text.
- Všech 8 externích odkazů vrací HTTP 200 (včetně %UnitTest dokumentace a Wikipedie pro Caché a ObjectScript).
- Jazyk: žádné em dashes ani unicode pomlčky, ` -- ` 37x střídmě, žádná z dříve vyhozených frází (fortnightly, developer-friendly, dogfooding, test doubles), žádné `% MH` komentáře, jediné červené TODO (záměrné). „license" konzistentně s americkým pravopisem, „licenser" jako role. Angličtina plynulá, v hlase záměru; věty rozdělené, středníky jen v tabulce.
- Milníky MS1-MS6 bez kolize s ID požadavků. Popis milníků a tabulka si odpovídají (názvy, pořadí).
- Notes-for-specification pokryté: definice licenčních dat (preambule Licensing), per-user opt-in mini-apps (M3), srovnání s legacy a WPF motivace (Project description, Architecture), persona multi-agenda uživatelů, demo scope v Out of scope.
- Záměr přenesen bod po bodu (intro, Relation, stakeholders, benefity, pain points, AI-augmented development, delivery, snapshot, consent, demo na obhajobě), mimo odchylky v nálezu 1.
- Opravy z 2. kola provedené a bez nové škody u: L4/storyboard (storyboard zmizel s nástrojem), L9 (odpadlo s nástrojem, přiznané v Changes), risk „full recreation" (přeformulováno), evaluace updatu (pilot), L1 „replacing the opaque format" (zmizelo), U7 (přeformulováno, viz jen nález 10 o čitelnosti), počet agend (forty sedí: 44 kořenových skupin stromu, z toho 3 skryté), T9 osobní údaje (výslovně), MVVM odkaz (Wikipedie), figure 1 pozice ([tb], teď na začátku s. 3), duplicitní destinace, kadence konzultací.

Faktická tvrzení ověřená v kódu (výběr; „sedí" = potvrzeno):

- **S1**: tři módy EDISON / LDAP / LDAP-AUTO jako per-user atribut (feature 23) rozhodovaný v `ALVA.SYS.Login.Authenticate` (`Login.cls:1129-1230`) sedí.
- **S2**: firma při loginu automaticky (registry + `param_firma`), přepnutí za běhu přes menu, zavírá hlavní okno a znovu se přihlašuje (`frmNoMDI.vb:1575-1591, 2934-2973`) sedí.
- **S8**: inaktivita vynucovaná serverem (`ALWA.SYS.Activity.UpdateTimeout`), 60 s odpočet balónkem (`frmNoMDI.vb:5097-5105`), admin force-out přes `^CacheTempEDISON` sedí. „Close all" v legacy neexistuje, spec ho správně uvádí jako nový.
- **S9/U2**: runtime guard `ALWA.SYS.Rest.EdisonVersion` porovnává segmenty 1 a 2, patch toleruje; mid-session mismatch v legacy = vynucený konec (`frmNoMDI.vb:5223-5232`); S9 „asks the user to restart" je vědomé zlepšení.
- **S10**: legacy `frmSetup` + vynucené ukončení a ruční restart (`frmNoMDI.vb:4752-4759`) sedí, S10 je zlepšení.
- **S11**: vynucená změna hesla při loginu (feature 17) i dobrovolná změna (`ALVA.SYS.Login.ChangePassword`) existují.
- **S16**: server flag `^EDISON("System","Login","RememberUser")` sedí; legacy šifrování v registru je reverzibilní AES s klíčem v datech, nový shell používá DPAPI (`PasswordProtector.cs`), takže N11 „stored credentials are encrypted" je splnitelné a lepší než legacy.
- **S18**: globální limit v `^EDISON("System","Public","Settings")` + per-user feature 30 (`Api.cls:754-760`, `Activity.cls:146-152`) sedí.
- **L1**: licence = šifrovaný záznam v zákaznické Caché, pull z archivu M-line, žádná licenční data v XML (potvrzeno znovu); read-only po expiraci existuje (E04, `ReadOnlyMode`) a je kooperativní per formulář, což spec správně přiznává slovy „as in the legacy system" (viz jen nález 11 o propagaci).
- **L3**: admin při plném limitu dostane okno aktivity a může odhlásit (`frmNoMDI.vb:2569-2586`, `formUSERSActivity.vb`) sedí. **L4**: bez licence vznikne náhradní licence s Nastavením a importem (`frmNoMDI.vb:415-428`), admin práva licenci neobcházejí (jediný bypass je servisní účet spojení, ne admin) sedí. **L6/L7**: legacy nemá typ klienta v `GetActiveUsers` ani audit; spec je správně uvádí jako nové.
- **Licenční preambule**: typ, platnost, feature set, numerické limity (LOGIN, USERS, RZ, ZAM, TELMAX, CACHE) sedí jako férové shrnutí; per-firma vazba a init klíče vědomě vynechané.
- **U1**: EDUPDATE oddělitelný, bere `WaitProcess=`/`RunExe=`, rollback s exit kódy 100/200 (`frmClientUpdate.vb:59-61, 502-507`) sedí. **U5**: pasivní „Je dostupná nová verze" existuje. **U6**: INF soubory per agenda, zobrazení před otevřením agendy, „už neukazovat" per verze (`frmNoMDI.vb:3081-3111, 4234`) sedí; spec správně slibuje klíčování podle obsahu jako novinku. **U9**: pauzu nastavuje M-line přes Compiler (`SetupWizard.cs:52-67`), „The company can pause" sedí. **U10**: admin nebo `UserMayUpdate` + právo `AllowUpdate` + kontrola aktivních uživatelů (`frmNoMDI.vb:4454, 4483, 4591-4603`) sedí. **U11**: legacy ukončuje s hláškou, spec to správně jen vylepšuje.
- **Intro o updateru** („known security and reliability issues"): plain HTTP s hardcoded hosty, admin heslo v plaintextu na příkazové řádce nočního tasku (`System.cls` RunAutoTransfer), heslo uživatele na příkazové řádce se statickým klíčem, hardcoded klíč CRC listu, busy-wait bez sleep, zakomentovaný login-deny, opuštěný rewrite `EDISON\ClientUpdater` (prázdná kostra). Věta je podložená.
- **Architecture**: bridge = samostatný .NET Framework 4.8 proces (`EDISONMAIN2API.csproj`), agendy načítá reflexí (`FormToRunGetter.cs:34,62`), legacy spawnuje přes Remoting (EDPROCESS, IpcChannel); „Spawns [.NET Reflection]" a „[.NET Remoting]" v containeru sedí. Jedna logická session pro hostované agendy sedí (bridge dostává shellGuid, `FormToRunGetter.cs:106-110`). Update balíčky jako soubory na serveru zákazníka sedí jako konvence (HTTP přímá cesta existuje jako alternativa, spec ji nevylučuje).
- **N1**: zámky per session GUID, `LockClear` při logoutu (`Lock.cls:166-197`, `EDUsers.vb:191-206`) sedí. **N2**: žádný mutex, oddělená konfigurace, sdílené DLL hlídané při publish; oba shelly na jedné stanici a instalaci reálně běží. **N11**: DPAPI sedí. **N13**: Job object `KILL_ON_JOB_CLOSE` existuje (jen zdůvodnění, nález 13). **N7**: 11 sdílených klíčů v jednom místě, zbytek inline česky, spec to přiznává.
- **M1/Platform**: AvalonDock (MS-PL), panely uspořádatelné a měnitelné; **M3**: plocha je zavíratelná (info bar nabídne obnovu), takže „jediná výjimka" pro news sedí; **M7**: 15 legacy panelů přes nezměněný kontrakt `MiniAplikaceAPI.LoadData`; **M8**: intervaly 10 s až 60 min; **M10**: poškozený layout se zahodí a nastartuje default.
- **T2, T3, T4, T5, T6, T8 (částečně), T9, T11, T13**: realistické, mají precedens v toolkitu nebo legacy; T9 „own windows only" je vědomé zúžení proti legacy (celá plocha, `EDISON\EDCORE\clsCore.vb:141-177`) a spec to říká.
- **Platform**: net10.0 (LTS) sedí, xUnit + NSubstitute + %UnitTest přítomné, WPF + MVVM bez knihovny je legitimní volba.
- **S4, S5, S13, S14, S17**: vyhledávání ve stromu, oblíbené, plocha s dlaždicemi (EDDesktop, drag and drop ze stromu), seznam běžících modulů s fokusem místo druhé kopie (`frmNoMDI.vb:312-355, 4249-4266`), status bar s uživatelem, licencí a verzí (`frmNoMDI.designer.vb:33-40`) mají legacy předlohu; S21 a S20 ji nemají a spec je správně vede jako novinky.
- **M9**: UseFO existuje v serverovém kontraktu (`MiniAplikaceAPI.cls:84-104, 254-272`), „as the legacy panels do" sedí. **M7**: 15 typů panelů v klientském enumu, 13 ve stromu (skupina 15, skrytá).
- **T7**: export do tabulky a tisková tabulka v gridu legacy existují (`EDFlexGrid2.Designer.vb:276`, `EDFlexGrid2.vb:1301-1327`). **T8**: pozice oken per IdFa + uživatel na serveru a „Reset forms" existují (`ALVA.LIBRARY.Form`, `frmNoMDI.vb:3472-3496`); potvrzení neuložených změn v legacy základu není, spec ho správně dává toolkitu jako novinku. **T9**: legacy posílá dva screenshoty všech monitorů (`clsCore.vb:141-172`, `frmMLErrorEx`), bez pole pro popis a bez odeslání bez chyby; obě věci spec správně slibuje jako nové. **T14**: barevné schéma per databáze (`^EDISON("System","Database",db,"Scheme")`) existuje, kreslí se na plochu.
- **S22**: v legacy žádný jednotný instalační průvodce (SETUPWIZARD, EDISONINS, KeyMaker, `CreateSuperUser` zvlášť), „stays a job for the legacy tools" sedí.
- **Tray a zprávy**: `EDTRAYEXE` není v `EDISON.sln` a integrace do shellu je zakomentovaná (`frmNoMDI.vb:843-863, 1042-1053`), takže vědomé vynechání je bezpečné.
- **Mockupy**: login bez výběru firmy sedí na S2; „Zapamatovat heslo" sedí na S16; plocha jako mini-app v hosting area sedí na S13; dva legacy panely sedí na M7 (mimo nález 4b o datovém listu).
