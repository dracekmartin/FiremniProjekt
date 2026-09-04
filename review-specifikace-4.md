# Obsahové review specifikace — 4. kolo

Datum: 4. 9. 2026. Předmět: `project-specification.tex` (stav na main, commit 2d28f31).
Rozsah: pouze obsah — fakta, konzistence, úplnost, ověřitelnost. Jazyk a stylistika vynechány.

Zdroje ověření: `project-proposal.tex`, `notes-for-specification.md`, legacy repo
`c:\MLineRepos\EDISON`, nový stack `c:\MLineRepos\EDISON2`, analýza
`EDISON2/docs/analyzy/update-mechanismus.md`. Cesty `EDISON/...` a `EDISON2/...` jsou
relativní k těmto repům; `spec:NNN` je řádek v `project-specification.tex`.

Ohnisko dle zadání supervizora: každý požadavek musí mít realizaci v něčem, co sekce
Milestones/Deliverables výslovně odevzdává, a seznam neodevzdaných částí musí být úplný.

---

## Nálezy

### Blocker

#### 1. U1 si protiřečí s U3 a U7: „existující engine smí dělat download i apply" vs. „download je modernizovaný"

- **Místo:** U1 (spec:245), U3 (spec:247), U7 (spec:251), sekce Changes (spec:575), architektura (spec:340).
- **Problém:** U1 říká „The download and apply steps may be carried out by the existing
  update engine" a Changes to opakuje („The existing low-level engine that copies and
  applies the files may stay"). Pokud ale download smí zůstat na existujícím enginu, nemůže
  zároveň platit U3 („The download of release files ... is modernized") ani U7 (HTTPS,
  žádné zbytečné re-download). Existující engine umí jen plain HTTP a jeho diff logika je
  přesně to, co U7 opravuje.
- **Důkaz:** download na stanici dělá EDUPDATE — tentýž engine, který dělá apply
  (`EDISON/UPDATERCL/clsAPI.vb:1064-1428`; `update-mechanismus.md` §2.3, §4a). HTTP je
  v něm natvrdo (`EDISON/UPDATERCL/clsAPI.vb:1126`).
- **Návrh:** v U1 a v Changes omezit povolení na krok **apply** (výměna souborů na disku).
  Explicitně napsat, že download krok (odkud a čím se soubory stáhnou) je vždy součástí
  nové práce dle U3/U7. Bez toho jsou požadavky na update mechanismus vzájemně nesplnitelné.

#### 2. U7 není realizovatelné ani hodnotitelné uvnitř deklarovaného scope

- **Místo:** U7 (spec:251), úvod sekce Update (spec:243 „Everything else ... stays as it
  is"), architektura (spec:340 „How M-line builds and publishes a release stays as it is"),
  seznam neodevzdaného (spec:505 „the release server that publishes releases").
- **Problém:** obě opravy z U7 leží mimo hranici, kterou si spec sám nakreslil:
  - **HTTPS**: release server dnes TLS vůbec neumí (na :59993 selže handshake, na :443 nic
    neposlouchá — `update-mechanismus.md` §4a, §7, ověřeno naživo). Zprovoznění vyžaduje
    certifikát a binding na straně M-line — tedy na release serveru, který je v seznamu
    neodevzdaných částí a o kterém spec říká, že se nemění.
  - **Re-download při Caché-only patchi**: root cause je v instalačním kroku na serveru
    zákazníka — EDISONINS při patchi smaže `EDISONCRC.bin` a přegeneruje ho jen ze souborů
    balíku (`EDISON/EDISONAPI/clsUpdater.vb:3069-3071`), stanice pak spadne do větve
    „stáhni všechno" (`EDISON/UPDATERCL/clsAPI.vb:1095`). Instalace na serveru není
    „download" (U3), a spec ji řadí pod „everything else stays as it is".
- **Návrh:** rozšířit deklarovaný zásah tak, aby pokrýval i instalační krok na serveru
  zákazníka (nebo říct, že nový download nahrazuje EDTRANSFER2 i evidenci souborů, takže
  CRC mechanismus přestane existovat). U HTTPS přiznat závislost: serverová strana
  (certifikát, binding) je konfigurační předpoklad zajištěný M-line, odevzdaný kód musí
  HTTPS podporovat a hodnotí se právě to. Jinak U7 nejde z odevzdaných zdrojáků hodnotit.

### Major

#### 3. U3 vs. U4: download „serves both the new and the legacy client", ale legacy path „stays as it is"

- **Místo:** U3 (spec:247), U4 (spec:248), úvod Update (spec:243 „Everything else,
  including the legacy clients' own update path, stays as it is").
- **Problém:** když modernizovaný download na stanice obsluhuje i legacy klienta, mění se
  tím legacy update path — minimálně jeho transportní část. Obě věty nemohou platit naráz.
- **Důkaz:** legacy klient si soubory stahuje sám přes EDUPDATE
  (`update-mechanismus.md` §2.3); jiná cesta k jeho obsloužení novým downloadem neexistuje.
- **Návrh:** říct přesně, kde nový download začíná a končí. Pravděpodobný záměr: úsek
  M-line → server zákazníka je společný (jedna serverová instalace slouží oběma), úsek
  server → stanice se modernizuje jen pro nový shell a legacy stanice jedou postaru. Pokud
  je záměr širší, U4 přeformulovat (legacy klienti se updatují beze změn **svého kódu**,
  ale přes nový transport).

#### 4. T10: „existing mechanism ... hands the aggregate over ... and the project uses it unchanged" nesedí

- **Místo:** T10 (spec:269).
- **Problém:** existující mechanismus (FEEDBACK) přenáší k M-line verze, licence a počty
  entit — žádná usage data. Aby předal nové agregáty použití oken a prvků, musí se změnit;
  „uses it unchanged" je tedy vnitřně sporné. Usage telemetrie v legacy neexistuje vůbec,
  takže mechanismus nemá kam „unchanged" sáhnout.
- **Důkaz:** volání po dokončení automatické aktualizace
  (`EDISON/EDISONMAIN/CACHE/ALVA/EDISON/System.cls:174-181`, `FEEDBACK.exe UPDATE 7`);
  přenášený obsah = verze, licence, počty `LOGIN;USERS;RZ;ZAM;TELMAX;CACHE;...`
  (`EDISON/Feedback/Feedback/CACHE/ALVA/FEEDBACK/Update.cls:6-70`,
  `EDISON/Feedback/Feedback/CACHE/ALVAFEEDBACK.inc`).
- **Návrh:** buď (a) přiznat, že se FEEDBACK sběr rozšíří o usage agregát (a zařadit tu
  změnu mezi odevzdávaný Caché kód / „internal tools modified"), nebo (b) T10 zúžit na
  „events se agregují v Caché zákazníka" a předání M-line vypustit. Varianta (a) je blíž
  původnímu záměru.

#### 5. Kontextový diagram: licence nestahuje backend, ale klient

- **Místo:** C4 context diagram, šipka „Pulls licenses [HTTP]" z Customer Caché backend na
  M-line services (spec:146) + caption (spec:151 „the backend reach M-line's own services").
- **Problém:** v legacy stahuje licence **klient**: při loginu (jen admin / uživatel
  s právem update) si otevře druhé MLRest spojení na server M-line, licence uloží do
  zákaznického backendu a na M-line je označí za aktivované. Backend sám nikam nesahá.
- **Důkaz:** `EDISON/Licence/EDLicence/Update.vb:21-89` (`LoadLicsFromMline` :62-68,
  `SaveLicsToCustom` :80, `MakeLicActivated` :86); spouštěč při loginu
  `EDISON/EDUsers/EDUsers/EDUsers.vb:170,320-328`; dále automat 15 dní před expirací
  (`EDISON/Licence/EDLicence/Activation.vb:287-296`) a ruční akce z menu.
- **Návrh:** šipku vést z klientů (nebo z EDISON 2) na M-line services a caption opravit.
  Věta v úvodu sekce Licensing („reaches the installation automatically from M-line's
  central archive", spec:235) může zůstat — je pravdivá, jen ji nedělá backend.

#### 6. L2: strom modulů nejsou „XML files distributed with the client", ale embedded resource v klientské DLL

- **Místo:** L2 (spec:238), úvodní bullet Licensing configuration (spec:71 „move from XML
  files"), caption kontextového diagramu (spec:151 „module definitions once they move out
  of the XML files").
- **Problém:** `edisontree.xml` a `edisonnavbar.xml` nejsou soubory ležící vedle klienta —
  jsou zakompilované jako embedded resource do EDCore.dll. Změna stromu tedy dnes vyžaduje
  rebuild a vydání klientské knihovny. Popis „XML files distributed with the client" je
  fakticky vedle; skutečný stav motivaci L2 ještě posiluje (přesně tohle má migrace do
  Caché odstranit), ale dokument má popisovat realitu.
- **Důkaz:** `EDISON/EDCORE/My Project/Resources.resx:121-125` (ResXFileRef na
  `Resources/edisontree.xml`), čtení `EDISON/EDCORE/VyjmutoEDISONApi.vb:311,319`
  (`My.Resources.edisontree`); `EDISON/DOCS/08-nasazeni.md:16` („Změna pouze
  v edisontree.xml ... nespustí překlad EDCore.dll"). Sémantika atributů, kterou L2
  slibuje zachovat, potvrzena: `hidden`, `mlineOnly`, `input`, `state`
  (`EDISON/EDISONMAIN/frmMain.vb:1094-1134, 1199-1260`).
- **Návrh:** přeformulovat na „XML definitions compiled into the legacy client" (a „tools
  that read the XML" pak znamená kód čtoucí embedded resource). Zmínka „distributed with
  the client" zmizí i z captionu diagramu.

#### 7. TemplateDesigner chybí v seznamu neodevzdaných částí, přestože na něm stojí N4 i Evaluation

- **Místo:** seznam neodevzdaného (spec:505), N4 (spec:284), Evaluation — developer study
  (spec:467), architektura (spec:330).
- **Problém:** TemplateDesigner je kolegova práce a neodevzdává se, ale v seznamu částí,
  „se kterými systém jen interaguje", uveden není. Přitom N4 se výslovně „checked ... on
  TemplateDesigner" a celá developer study se opírá o zpětnou vazbu jeho autora. Komise
  tak má hodnotit kritérium závislé na kódu, o kterém spec neříká, že ho nedostane.
- **Návrh:** přidat TemplateDesigner do seznamu neodevzdaných částí. U N4 nechat
  demonstrativní mini-app jako primární (odevzdaný) důkaz a TemplateDesigner jako
  doplňkový; u developer study doplnit, že výstupem je zdokumentovaná zpětná vazba
  (v developer/AI dokumentaci), aby bylo co hodnotit.

#### 8. Licenční požadavky L1, L3, L4 nemají žádný milník ani man-days

- **Místo:** tabulka Time Schedule (spec:522-542), Milestones (spec:488-495), text nad
  tabulkou (spec:518 „Licensing and the update mechanism follow once the foundation is
  stable, partly in parallel").
- **Problém:** text nad tabulkou licensing slibuje, ale žádný milník ho nenese: MS3
  pokrývá jen L2 (module tree). Vynucování licence v shellu (L1), správa sessions (L3)
  a vstup bez platné licence (L4) nemají slot ani man-days. Totéž platí pro M6 (news
  panel), T9 (error reporting), T10 (telemetrie) a T11 (support panel) — v aktivitách
  milníků se nevyskytují. U dokumentu, který se hodnotí proti milníkům, to znamená, že
  část požadavků nemá termín, proti kterému se pozná skluz.
- **Návrh:** doplnit licensing do aktivity MS3 (kam věcně patří — login, licence, strom)
  a M6/T9/T10/T11 jmenovat v aktivitě MS4 nebo MS6, případně přidat řádek. Man-days
  přerozdělit, součet 78 zachovat.

#### 9. Nepřiznaná odchylka od záměru: „all public APIs ... ship with unit tests" → „80 % of non-visual types"

- **Místo:** N3 (spec:283) a Evaluation (spec:461) vs. proposal (project-proposal.tex:65
  „all public APIs in the toolkit and shell ship with unit tests"); sekce Changes
  (spec:563) o tom mlčí.
- **Problém:** kritérium testování se mezi záměrem a specifikací změnilo (jiná metrika,
  věcně slabší — 80 % typů nepokrývá „každé veřejné API"). Pravidlo dokumentu je, že
  odchylky od záměru přiznává sekce Changes; tahle tam chybí.
- **Návrh:** buď do Changes doplnit odstavec (proč je 80 % non-visual typů + povinné test
  třídy na Caché lepší měřitelné kritérium než „all public APIs"), nebo N3 formulovat tak,
  aby závazek ze záměru zůstal zachovaný a 80 % bylo jeho měření.

### Minor

#### 10. L1: „the installation stays read-only" vynucuje neodevzdaný kód

- **Místo:** L1 poslední věta (spec:237).
- **Problém:** read-only režim po expiraci vynucují v legacy agendy a backend — kód, který
  se neodevzdává. Z odevzdaných zdrojáků jde hodnotit jen podíl shellu: vyhodnotit stav
  licence a předat ho (S6 to pro bridge už říká). Věta o „instalaci" dělá z L1 požadavek
  hodnotitelný jen skrz neodevzdanou část.
- **Důkaz:** legacy: `EDISON/Licence/EDLicence/Activation.vb:277-285` (E04 → režim čtení),
  `EDISON/EDISONMAIN/Rozhrani (mainform)/frmNoMDI.vb:1258-1259` (`MLProps.ReadOnlyMode`),
  respektování v agendách (např. `EDISON/DSPODILOVANI/formDSPODILOVANIMain.vb:2086-2090`).
- **Návrh:** přeformulovat na povinnost shellu: shell stav „expirováno = čtení" vyhodnotí,
  zobrazí a předá agendám (bridge i nový stack); vynucení uvnitř legacy agend zůstává
  existující chování.

#### 11. L3: neříká se, kde bude UI správy sessions

- **Místo:** L3 (spec:239).
- **Problém:** v legacy je prohlížení a vyhazování sessions okno agendy USERS
  (`formUSERSActivity`), které shell jen spouští. Pokud nový shell tohle okno pouze
  spustí přes bridge, je L3 realizované neodevzdaným kódem a nejde hodnotit. Pokud má
  vzniknout v novém shellu, má to být vidět (a chybí to v milnících, viz nález 8).
- **Důkaz:** `EDISON/USERS/Users-v4/formUSERSActivity.vb:527-558` (`SetLogout`),
  automatické otevření při plném limitu `frmNoMDI.vb:2573-2580`.
- **Návrh:** jedna věta v L3: dialog je součást nového shellu (odevzdané), nebo se
  přiznaně používá legacy agenda přes bridge a hodnotí se jen napojení.

#### 12. U10: pauzu distribuce dělá neodevzdaný nástroj M-line

- **Místo:** U10 (spec:254).
- **Problém:** „The company can pause the distribution" — akce pauzy se provádí nástrojem
  na straně M-line (flag `BlokovatAktualizace` ve `VERZE.INF`, přepínaný Compilerem),
  který se neodevzdává a o kterém spec říká, že publikace verzí zůstává beze změny.
  Odevzdatelná a hodnotitelná je jen klientská strana: pozastavenou verzi ignorovat.
- **Důkaz:** `update-mechanismus.md` §1 (Typy verzí a VERZE.INF), §5 (nový shell flag
  honoruje v notice kanálu).
- **Návrh:** přeformulovat z pohledu klienta: „When M-line marks a release as paused,
  clients behave as if no newer release existed." Mechanismus pauzy zařadit mezi věci,
  se kterými systém jen interaguje.

#### 13. „The chain has two known failures" — známé vady jsou tři

- **Místo:** úvod sekce Update (spec:243), U7 (spec:251 „fixes the two known failures").
- **Problém:** analýza dokládá tři známé mouchy: (a) HTTP, (b) re-download, (c) vynucená
  aktualizace občas způsobí, že klient vůbec nejde spustit. Spec (c) fakticky řeší — N11
  žádá „a failed update must leave the client startable" — ale prezentuje řetězec jako se
  dvěma vadami a souvislost N11 s třetí vadou nikde není.
- **Důkaz:** `update-mechanismus.md` úvod (mouchy a/b/c) a §4c (sedm mechanismů, jak
  forced update zabrání startu).
- **Návrh:** přiznat třetí vadu v úvodu Update sekce a u ní odkázat N11 (a případně U1:
  nový koordinátor nesmí zdědit `CliUp.ini` mechaniku). U7 pak opravit počet, nebo nechat
  U7 dvěma vadám downloadu a třetí nechat N11.

#### 14. U6: „The eventual goal is to write and serve them from the company web" je aspirace uvnitř požadavku

- **Místo:** U6 (spec:250).
- **Problém:** poslední věta požadavku popisuje cíl, o kterém nejde poznat, zda je součástí
  projektu, a tedy ani co se hodnotí. Zbytek U6 je v pořádku (notes z textových souborů
  release).
- **Návrh:** větu vypustit, nebo přesunout do Out of scope („serving release notes from
  the company web is future work").

#### 15. M7: mini-app obaly legacy panelů nejsou jmenované v odevzdávaném kódu

- **Místo:** M7 (spec:228) vs. seznam Source code (spec:499).
- **Problém:** deliverables jmenují „the demonstrative mini-app", ale mini-app podoby
  legacy přehledových panelů (M7) ne. Nejspíš se rozumí součástí shellu, ale u požadavku,
  který se hodnotí, je to vhodné říct — zvlášť když serverová strana panelů (existující
  kontrakt `ALVA.EDISON.MiniAplikaceAPI`) se správně neodevzdává.
- **Důkaz kontraktu:** `EDISON/EDISONMAIN/EDMiniApplication.vb:806` (LoadData),
  `EDISON/EDISONMAIN/CACHE/ALVA/EDISON/MiniAplikaceAPI.cls:226-325`.
- **Návrh:** v seznamu Source code rozšířit položku na „the demonstrative mini-app and the
  mini-app adapters for the legacy overview panels".

#### 16. Evaluation nemá měřitelné kritérium toolkitu, které si spec sám připravil v poznámkách

- **Místo:** Evaluation — developer study (spec:467); `notes-for-specification.md:17`
  („first end-to-end agenda port using the toolkit should fit within X developer-days").
- **Problém:** poznámky k specifikaci navrhují měřitelné kritérium úspěchu toolkitu;
  ve specifikaci z něj nezbylo nic — developer study je průběžný sběr zpětné vazby bez
  prahu. Není jasné, jestli jde o vědomé rozhodnutí.
- **Návrh:** buď kritérium doplnit (stačí měkčí, měřitelná varianta: mini-app pro další
  agendu postavená nad toolkitem bez zásahu do shellu — což už měří N4), nebo poznámku
  v `notes-for-specification.md` označit za vyřízenou rozhodnutím „nedávat".

#### 17. Man-days MS4 a MS5 jsou proti obsahu těsné

- **Místo:** tabulka Time Schedule (spec:522-542).
- **Problém:** MS4 má 6 MD na celý mini-app subsystém: interface, hosting s dokováním,
  settings okno (M3), demonstrativní mini-app (M5) a převzetí legacy panelů (M7 — kontrakt
  s 15 typy panelů). MS5 má 12 MD na koordinaci updatů plus nový download pro server
  i stanice, pro oba klienty, s HTTPS (U3, U7) — proti složitosti řetězce doložené
  analýzou (EDTRANSFER2, EDISONINS, evidence souborů, koexistence s legacy) je to spodní
  odhad. Rezerva jinde v tabulce není.
- **Návrh:** buď přerozdělit (MS2/MS3 mají po 17 MD a část toolkitu lze zúžit), nebo u MS5
  explicitně říct minimální variantu (např. HTTPS jen na úseku M-line → server) — sekce
  Risks už „each outcome can be reduced" připouští, ale u updatů se hodí říct, co je ta
  menší, stále užitečná verze.

### Nit

#### 18. Kontextový diagram: legacy šipka na M-line services nenese error reports

- **Místo:** C4 context (spec:147 „Updates, news") vs. T9 (spec:268 „the portal's existing
  intake channel that the legacy client already uses").
- **Důkaz:** legacy klient kanál skutečně používá —
  `EDISON/EDLIBRARY/frmMLErrorEx.vb:11-12` (`podpora.m-line.cz/api/telemetry/ingest`).
- **Návrh:** doplnit „error reports" i do legacy šipky (nebo z obou šipek nechat jen
  souhrnné „news, updates, error reports").

#### 19. Diagramy a captions prosakují implementaci

- **Místo:** container diagram — „Spawns [.NET Reflection]" u bridge (spec:429), captions
  mockupů „captured from the development build" (spec:305, 312, 319).
- **Problém:** spec má teoreticky předcházet implementaci; „.NET Reflection" u bridge je
  forward-looking implementační detail (u legacy je „.NET Remoting" doložitelný fakt —
  `EDISON/EDLIBRARY/clsForm.vb:2910-2970`, IpcChannel `:3303-3380`) a „development build"
  přiznává existující implementaci. Věcně je obojí pravdivé, jde o soulad s vlastním
  pravidlem dokumentu — rozhodnutí je na autorovi a supervizorovi.
- **Návrh:** u bridge stačí „Spawns [process]"; u mockupů např. „captured from
  a prototype" nebo ponechat a vědomě akceptovat.

#### 20. N12: práh 2 s nikdo neměří

- **Místo:** N12 (spec:292).
- **Problém:** „Checked during the daily internal use: a freeze noticed by the developers
  counts as a defect" — deklarovaný práh 2 s se reálně nekontroluje, kontroluje se
  „všimnutelný freeze". Jako jediné z NFR má N12 check, který s vlastním číslem nesouvisí.
- **Návrh:** buď práh vypustit (freeze bez indikátoru = defekt), nebo přidat měřitelný
  moment (např. dlouhé operace jdou přes progress overlay z T2, což je kontrolovatelné).

---

## Trasovatelnost požadavků na odevzdávané artefakty (souhrn)

Odevzdává se (spec:499): shell, bridge, toolkit, demonstrativní mini-app, nový download,
Caché kód projektu (server shellu, definice stromu, koordinace updatů), testy, upravené
interní nástroje. Neodevzdává se (spec:505): backend s logikou agend, legacy klient
a agendy, existující apply engine, licenční archiv, release server, podpůrný portál.

- **S1–S19**: realizace v shellu, serverová strana v „the shell's server side" — OK.
  S6 se hodnotí předváděcí instalací (spec:513), závislost na legacy agendách je přiznaná.
- **M1–M11**: shell + mini-apps — OK až na M7 (nález 15). M6/M8/M9 správně čerpají
  z neodevzdaného portálu a server kontraktu, klientská strana je odevzdaná.
- **L1–L4**: L2 odevzdáno (definice stromu + generátor jako interní nástroj). L1 read-only
  (nález 10), L3 UI (nález 11), všechny tři bez milníku (nález 8).
- **U1–U11**: U2/U8/U9/U10 kryje „update coordination" Caché kód + shell; U3 „new
  download". Rozpory a hranice scope: nálezy 1, 2, 3, 12.
- **T1–T13**: toolkit + shell — OK. T9 správně deklaruje „uses the channel, does not build
  it". T10 (nález 4). Sémantika T9 („narrower than the legacy behavior") fakticky sedí —
  legacy přikládá dva screenshoty celých obrazovek všech monitorů
  (`EDISON/EDCORE/clsCore.vb:141-176`, `EDISON/EDLIBRARY/clsSystem.vb:547,559`).
- **N1–N14**: checky jsou proveditelné z odevzdaného + pilot; výjimky: N4 závisí na
  TemplateDesigneru (nález 7), N12 check nesedí na práh (nález 20).

## Ověřená fakta bez nálezu (výběr, pro úplnost)

- S1 tři režimy autentizace: EDISON / LDAP / LDAP-AUTO, per uživatel
  (`EDISON/EDUsers/EDUsers/CACHE/ALVASYSLogin.inc:21-25`, `.../SYS/Login.cls:1129-1181`).
- S10 EDISONini.xml vedle binárek, klíčový soubor updatu, zdroj připojení
  (`EDISON/EDLOADER/clsStartup.vb:18,64-67`, `EDISON/UPDATERCL/clsAPI.vb:1089-1119`).
- S18 inactivity limit na serveru + per-user override, override vyhrává
  (`EDISON/MLRest/MLRest/CACHE/ALWA/SYS/Activity.cls:146-150`,
  `EDISON/USERS/CACHE/ALVAUSERS.inc:87`).
- U9 odpočet ~minuta odpovídá legacy (60 s,
  `EDISON/EDSystem/CACHE/ALVA/SYSTEM/Main.cls:70`).
- L1(a,b,d), L4, „admin práva neobcházejí licenci", L3 chování — vše doloženo
  (`EDISON/EDISONMAIN/frmMain.vb:1079-1091`, `EDISON/EDUsers/EDUsers/EDUsers.vb:283-311`,
  `EDISON/Licence/EDLicence/Activation.vb:277-285`, `frmNoMDI.vb:415-428, 2573-2580`,
  `EDISON/USERS/Users-v4/formUSERSActivity.vb:527-558`).
- L2 sémantika k zachování (hidden, mlineOnly, input, state) ve stromu existuje
  (`EDISON/EDISONMAIN/frmMain.vb:1094-1134`).
- M9 „funkční omezení" existuje a je oddělené od práv
  (`EDISON/USERS/CACHE/ALVA/USERS/API/FO.cls`).
- N1 zámky: drží je session GUID, logout je uvolňuje
  (`EDISON/MLRest/MLRest/CACHE/ALWA/SYS/Lock.cls:166-199`,
  `EDISON/EDUsers/EDUsers/EDUsers.vb:191-206`).
- N13 motivace reálná: agendy = samostatné EDPROCESS.exe bez watchdogu rodiče, po zabití
  shellu drží zámky (`EDISON/EDLIBRARY/clsForm.vb:2910-2970`, `EDISON/EDPROCESS/`).
- Kontrola dokumentu: všechny `\req{...}` odkazy vedou na definované požadavky, číslování
  souvislé; žádné TODO, `\textcolor{red}`, `% MH` ani „nice to have"; soubory
  `img/login.png`, `img/main-window.png`, `img/mini-apps.png` existují; součet man-days
  78 sedí (6+17+17+6+12+5+10+5) a odpovídá ~39 týdnům × 2 dny.
- Pokrytí záměru: až na nález 9 nic ze záměru nechybí; odchylky (licenční tool, pilot,
  update engine, service locator) jsou v Changes přiznané.

## Otevřené otázky pro autora

1. Kde přesně vede hranice mezi „novým downloadem" a tím, co zůstává? Řetězec má tři
   kroky: stažení na server (EDTRANSFER2), instalace na serveru (EDISONINS), stažení
   a výměna na stanici (EDUPDATE). Které z nich nový download nahrazuje? (souvisí
   s nálezy 1–3)
2. Kdo zajistí certifikát a HTTPS binding na release.m-line.cz a jak se u obhajoby
   prokáže „supports secure transport" — stačí, že odevzdaný kód HTTPS umí?
3. MS5 se demonstruje jen self-updatem stanice. Kde se předvedou U8 (odmítnutí loginu při
   instalaci), U9 (vyhození klientů), U10 (pauza) a U11 (oprávnění + běžící uživatelé)?
4. Má nový shell vlastní okno správy sessions (L3), nebo se spustí legacy agenda USERS
   přes bridge?
5. Metrika rychlosti toolkitu z `notes-for-specification.md` („first agenda port ≤ X
   developer-days") — vědomě vypuštěna, nebo se má do Evaluation vrátit?
6. Login obrazovka nového shellu ukazuje panely portálu před přihlášením; legacy je
   ukazuje až po něm (`EDISON/EDISONMAIN/frmMain.vb:954-961` v `InitializeForm`). Záměrná
   změna chování? (Do specifikace nic nechybí, jen ať je to vědomé — a případně zmíněné
   v user dokumentaci.)
7. „Update coordination" Caché kód: přebírá i pending lock při serverové instalaci
   (dnes `ALVA.SETUP.Main.LoadSettings` + EDLOADER), nebo U8 pro nový shell znovu čte
   existující globál? Rozhoduje o tom, co z Caché kódu se odevzdává.
