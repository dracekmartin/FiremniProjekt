# Gap report: kandidáti na požadavky chybějící v project-specification.tex

Vzniklo průchodem ../EDISON2 (dokumenty + kód shellu a toolkitu) a ../EDISON (legacy parita),
třemi nezávislými cestami s křížovou verifikací proti plnému textu specifikace (S1-S11, M1-M6,
L1-L6, U1-U6, T1-T6, N1-N11). Pracovní soubor, po zpracování smazat.

Formát položky: navržené znění (EN, hlas specifikace), evidence, doporučení
(**essential** / **nice to have** / **spíš nezanášet**), případná kolize se stávajícím textem.
Doporučení je můj odhad; výběr je na autorovi. Položky, u kterých z repa nejde poznat záměr,
jsou v sekci „Ověřit s autorem" na konci.

Legenda evidence: `E2/` = c:\MLineRepos\EDISON2, `E1/` = c:\MLineRepos\EDISON.

---

## Shell

**SH-1. První konfigurace stanice** - essential
> At first launch on an unconfigured workstation, the shell asks for the server connection, verifies it, and continues without a restart.

Evidence: `E2/EDISONMAIN2/dotnet/Windows/SetupWindow/`, `docs/archive/PLAN.md` feature #5, legacy `E1/EDISONMAIN/frmSetup.vb`.
Bez toho neexistuje cesta, jak dostat shell na čisté PC; uvidí ho i komise (čistá instalace na demu). Pozor na rozhraničení: konfigurace **stanice** (připojení) je v projektu, inicializace **serveru** (firma, licence, první admin) vědomě zůstává legacy nástroji - i to stojí za větu, viz „Ověřit s autorem".

**SH-2. Změna hesla (dobrovolná i vynucená)** - essential
> Users with backend-managed accounts can change their password in the shell, and the backend can require a password change at the next login.

Evidence: `E2/.../Windows/LoginWindow/ChangePasswordWindow.xaml.cs`, `RESPONSIBILITIES.md` §2.3, legacy `frmChangePassword.vb`.
Ve spec chybí úplně, přitom je to součást přihlašovacího toku (S1) a bezpečnostní hygieny. Podmínka „backend-managed accounts" kryje to, že u doménových/Windows účtů heslo nespravuje EDISON.

**SH-3. Přihlášení jiného uživatele za běhu** - essential
> The user can switch to a different user account at runtime, which closes open agenda windows first.

Evidence: `E2/.../Startup/SessionSwitcher.cs`, legacy `frmNoMDI.vb` (`mnuPrgLogin_Click`).
Zrcadlí S2 (přepnutí firmy), stejná formulace. Sdílené stanice a podpora to používají denně; netriviální kvůli licenčnímu slotu a per-user preferencím.

**SH-4. Plocha se zástupci uvnitř shellu** - essential
> The shell offers a desktop area where the user arranges shortcut tiles for modules and launches them, persisted per user and company.

Evidence: `E2/EDISONMAIN2/docs/DESKTOP_SHORTCUTS.md`, `.../MiniApps/DesktopMiniAppView.xaml.cs`, legacy `frmMain.vb` (`edDesktop`).
Legacy parita, velká viditelná funkce. Nezaměňovat se S10: S10 jsou **Windows** zástupce (.lnk), tohle je plocha **uvnitř** shellu. Datový model je nadmnožina oblíbených a legacy zná i zástupce na tiskové sestavy a pohledy datového skladu - jestli je nový shell má podporovat, viz „Ověřit s autorem".

**SH-5. Přehled běžících agend a single-instance chování** - essential
> The shell lists the running agendas and can bring any of them to the front, and launching an agenda that already runs focuses the existing window instead of starting a second copy.

Evidence: `E2/.../EdisonMainViewModel` (RefreshRunningModules), `EDControls/dotnet/EDViewModel/WindowNavigator.cs`, legacy `frmNoMDI.vb` (`ObnovDropdawnSAktivnimiOkny`).
Spec o práci s více okny/agendami mlčí úplně; tohle je základ orientace v multi-okno provozu.

**SH-6. Doplnění S6: bridge předává i licence a práva** - essential (úprava S6)
> ...through a compatibility bridge that passes the established session, company context, and the user's licensed permissions.

Evidence: `E2/EDISONMAIN2/CLAUDE.md` (ConfigureForm / AccessTable: bez licencovaných elementů agendy gatující obsah nezobrazí data), `RESPONSIBILITIES.md` §7.
Kolize: S6 dnes říká jen „session and company context" - to je fakticky málo, agendy by nefungovaly.

**SH-7. Tiché přihlášení Windows identitou** - nice to have (rozšíření S1)
> Where the backend allows it, the shell signs the user in under their Windows identity without showing the login dialog.

Evidence: `E2/.../Startup/AppBootstrapper.cs` (TryAutoLogin), legacy `frmNoMDI.vb` (`AllowAutoLogin` + server-side vypínač).
S1 režim „Windows identity without a password" zmiňuje, ale ne že login proběhne úplně bez interakce - to je jiná uživatelská zkušenost. Kolize: mockup loginu popisuje jen formulář; stačí doplnit větu, že se může přeskočit.

**SH-8. Zapamatování přihlášení** - nice to have
> The login screen can remember the user name and, where the backend permits it, the password, stored encrypted on the workstation.

Evidence: `E2/.../Startup/PasswordProtector.cs`, legacy server-side přepínač „Povolit pamatování hesla uživatele".
Mockup „a remember-me option" už zmiňuje, ale jako požadavek to nikde není; N11 kryje jen šifrování. Za zmínku stojí i to, že o povolení rozhoduje server.

**SH-9. Výchozí (startovací) modul** - nice to have
> The user can mark one module as their startup module, and the shell opens it automatically after login.

Evidence: `E2/.../EdisonMainViewModel.SetStartupWindow`, legacy `frmMain.vb` (`RunStartupForm`).
Pro uživatele dělající celý den jednu agendu zásadní; per uživatel a firma.

**SH-10. Automatický seznam nejpoužívanějších modulů** - nice to have
> Next to the favorites, the shell keeps an automatically maintained list of the user's most used modules.

Evidence: `E2/.../ModulePreferencesService.cs`, `EDControls/DOCS/05-userdata-scopes.md` (klíč ModuleUsage), `LEGACY_FEATURE_INVENTORY.md` §5.1 (vědomě nad rámec legacy).
Doplňuje S5 (oblíbené). Je to záměrná novinka, ne vedlejší produkt.

**SH-11. Doplnění S8: potvrzení ukončení a zavřít vše** - nice to have (úprava S8)
> Closing the shell warns the user that all open agenda windows will close, and one action can close all open windows at once.

Evidence: `E2/.../EdisonMainViewModel.ConfirmShellClose`, `CloseAllCommand`, legacy `frmNoMDI.vb` (`frmMDI_FormClosing`).
S8 má „orderly shutdown", ale ne varování před ztrátou rozdělané práce v agendách. Programové ukončení (force logout, version gate) potvrzení obchází - to je detail pod rozlišovací úroveň.

**SH-12. Ovladatelnost klávesnicí** - nice to have (spíš do N skupiny)
> The shell and the toolkit components are operable by keyboard, and the keyboard focus stays visible in both themes.

Evidence: `E2/EDISONMAIN2/dotnet/Windows/EdisonMainWindow/` (Enter, Ctrl+F, Esc), `EDControls/CLAUDE.md` (FocusVisualStyle: systémový rámeček je v dark neviditelný), `EDControls/dotnet/EDInputForm/CLAUDE.md` (zkratky formuláře).
Accessibility vlastnost, kterou toolkit reálně řeší; jednotlivé zkratky jsou pod rozlišovací úroveň, princip ne.

**SH-13. Druhá instance shellu** - spíš nezanášet
Menu „Spustit další EDISON" (práce ve dvou firmách současně). Evidence: `E2/.../EdisonMainViewModel.RunEdisonCopy`, legacy `ToolStartEdisonCopy`. V legacy inventuře zároveň označeno jako „neportovat", přesto v novém shellu implementováno - rozpor, viz „Ověřit s autorem". Jako požadavek bych nezanášel, dokud není jasný záměr.

**SH-14. Drobnosti: informační pruh se svátkem, sezónní čekací okna, hyperlink strom** - spíš nezanášet
Evidence: `E2/.../CzechNamedays.cs`, legacy `frmW8Xmas.vb` a spol. Kosmetika pod rozlišovací úroveň specifikace.

---

## Mini-apps

**MA-1. Firemní komunikační kanál v shellu** - essential
> The shell displays company-provided web content (news, wiki, training) as built-in mini-apps, available already on the login screen.

Evidence: `E2/.../MiniApps/CoreMiniApps.cs`, `WidgetUrls.cs`, `LoginWindow.xaml` (tři widgety vedle formuláře), legacy `frmMain.vb` (DashPanel z podpora.m-line.cz).
Explicitně motivovaná byznys funkce (IT dosud nemělo kanál k uživatelům), viditelná při každém demu. Spec o webovém obsahu v shellu nemá ani slovo.

**MA-2. Připnutá (nezavíratelná) mini-app** - essential (doplněk MA-1, úprava M1/M3)
> A mini-app can be marked as permanently present, and the shell keeps it open across layout changes and resets.

Evidence: `E2/.../MiniApps/MiniAppManager.EnsurePinnedOpen`, `MiniAppDescriptor.Pinned`, `MINIAPPS.md`.
Garance, že komunikační kanál z MA-1 uživatel trvale nezavře. Kolize: M3 říká „each user adds and removes them" - připnutá mini-app je vědomá výjimka z tohoto pravidla, spec by ji měla přiznat.

**MA-3. Legacy přehledové panely zůstávají dostupné** - essential
> The at-a-glance panels of the legacy shell remain available as mini-apps through their existing server contract.

Evidence: `E2/EDISONMAIN2.LegacyMiniApps/` (generický driver + 15 přehledů: fluktuace, lékařské prohlídky, údržba vozidel, statistiky faktur...), legacy `EDMiniApplication.vb`.
M5 slibuje „aspoň jednu demonstrativní mini-app" - realita je celý legacy panel, jinak by uživatel přechodem funkce ztratil. Zároveň ukazuje zamýšlený model „mini-apps vlastní satelitní projekt, ne shell" (souvisí s M2 a N4).

**MA-4. Bohatší správa plochy mini-apps** - nice to have (úprava M1)
> A mini-app can be opened several times with different settings and can float in a separate window, still under shell management.

Evidence: `MINIAPPS.md` §1/§4/§5 (multi-instance, floating, auto-hide), `LEGACY_FEATURE_INVENTORY.md` §6 („lepší než legacy").
M1 zná jen „arranged and resized"; floating a multi-instance jsou viditelně víc.

**MA-5. Automatické obnovování mini-apps** - nice to have
> The user can have open mini-apps refresh automatically at a chosen interval.

Evidence: `E2/.../MiniApps/PeriodicRefreshMiniAppViewModel.cs`, `EDControls/dotnet/MiniApps/MiniAppRefresh.cs`. Vědomě nová funkce nad legacy (dispečerské nástěnky).

**MA-6. Lifecycle při přepnutí firmy/uživatele** - nice to have (úprava M6)
> After a user or company switch, mini-apps reopen from the layout of the new context and never show data of the previous session.

Evidence: `MINIAPPS.md` §5/§7, `E2/.../EdisonMainViewModel.ReplaceMiniAppSession`.
M6 má persistenci; korektnostní část (data staré firmy nesmí zůstat viditelná) je nová. Patří k ní i odolnost: poškozený uložený layout nesmí shodit start.

**MA-7. Licencování mini-apps** - kolize s M4, ověřit
Legacy licencuje mini-apps samostatně (vlastní licenční element + filtr přístupovými právy + server-side přepínač funkčního omezení), M4 říká „nabízena, když je licencovaná rodičovská agenda" - to je vědomé prozatímní zjednodušení (`MINIAPPS.md` §10 „odloženo"). Buď M4 rozšířit („only when the user is licensed and permitted to use it"), nebo nechat a vědět o rozdílu. Pozn.: v kódu zatím není ani ta agendová varianta M4 a katalog se nepřefiltrovává při přepnutí firmy.

---

## Licensing

**LI-1. Admin řešení vyčerpaných licencí** - essential
> When the concurrent-login limit is reached, an administrator can view the active sessions, force one out, and sign in, with the authorization checked on the server.

Evidence: `E2/.../Startup/LicenceVerifier.cs`, `RESPONSIBILITIES.md` §2.9 (včetně zdůvodnění deadlocku a server-side autorizace), legacy `formUSERSActivity`.
L2 má jen „presents login rejections" - chybí cesta ven. Souvisí s NF-12 (audit zásahu).

**LI-2. Instalace bez licence se nesmí zamknout** - essential
> An installation without a valid licence still lets an administrator in far enough to import a new one.

Evidence: `E2/.../Startup/LicenceService.cs` (fallback licence), `RESPONSIBILITIES.md` §13.4, `MVP_PLAN_v1.md` (alternativní přístup „shell bez licence, jen unrestricted forms").
Recovery cesta, opak L2. Konkrétní mechanismus (syntetická licence vs. nechráněná minimální množina) je implementační volba, požadavek je „nesmí nastat zamčený stav".

**LI-3. Obnova licence za běhu** - nice to have
> The licence can be refreshed at runtime without restarting the shell, and the installation renews it automatically from the central archive before it expires.

Evidence: `E2/.../EdisonMainViewModel.RefreshLicense`, `RESPONSIBILITIES.md` §13.5-13.12 (import klíče, 15denní auto-refresh), legacy `ToolRefreshLicense`.
L1 kryje delivery do backendu; klientská strana (ruční refresh, automatika před expirací) chybí.

**LI-4. Doplnění L4: migrace stromu zachovává sémantiku a nerozbije konzumenty XML** - essential (úprava L4)
> The migration preserves the semantics of the existing tree definitions (hidden and internal-only entries, input parameters, window defaults), and tools that read the XML today keep working until they are replaced.

Evidence: legacy `frmMain.vb` (`BuildTreeXML`: atributy hidden, mlineOnly, color, state, input), `docs/archive/PLAN.md` „Vědomý dluh: dva zdroje navigačního stromu" (na XML visí licenční nástroje: USERS tiskové sestavy, GetLicensedForms, Licence, LicenceInfo; genuinely nový Caché-only modul nejde dnes olicencovat).
Kolize: L4 mluví jen o přesunu definic; ekvivalence (obdoba L5) a externí konzumenti tam nejsou, přitom je to hlavní riziko té migrace.

**LI-5. Admin fallback nikdy neobchází licenci** - nice to have
> An administrator always keeps access to the administration modules, and administrator rights never bypass the licence.

Evidence: `RESPONSIBILITIES.md` §5.2 (klíčový invariant: fallback doplňuje jen accessTable, ne licenseRights).
Anti-lockout invariant na ose práv, doplněk LI-2 na ose licence.

**LI-6. Licenční limity nad rámec souběžných přihlášení** - částečně pokryto, jen připomínka
Úvod sekce Licensing říká „numeric limits (most importantly concurrent logins)" a L2 „warnings for approaching limits" - to plurál limitů kryje. Jen pro migraci (L5) je dobré vědět, že limity zahrnují i celkové uživatele, vozidla, zaměstnance a databázová připojení (`RESPONSIBILITIES.md` §13.1, §13.8) a že překročení vozidel/zaměstnanců legacy hlásí v InfoPanelu. Nezanášet nový požadavek, jen případně rozšířit závorku v úvodu sekce.

---

## Update

**UP-1. Novinky per agenda při spuštění** - essential (úprava/rozšíření U6)
> Before opening an agenda whose package changed since the user last saw it, the shell shows what changed, and the user can dismiss the notes until they change again.

Evidence: `E2/.../Windows/ChangelogWindow/ChangelogPromptService.cs` (per-balík _DIFF.INF, snooze hashem obsahu, max jednou za session), legacy `frmInformation.vb` (server-side opt-out per uživatel + verze + formulář).
Kolize: U6 mluví jen o zobrazení „after an update" - implementovaný mechanismus je bohatší (per agenda, odložení) a je to nejviditelnější část U6. Doporučuji rozšířit U6 nebo přidat sousední požadavek.

**UP-2. Zákaz přihlášení během serverové instalace** - essential
> While the server side of an installation is being updated, new logins are refused and running clients are asked to close.

Evidence: `E2/EDISONMAIN2/docs/UPDATE_FLOW.md` §3/§5 (pending lock, TerminateClients, noční automatika), legacy `EDLOADER/clsStartup.vb`, `EDTRAY/clsMain.vb` (LoginAllowed).
U3 „modernizovaná distribuce" tenhle provozní semafor nezmiňuje, přitom je uživatelsky nejviditelnější částí update flow. Druhá půlka (broadcast běžícím uživatelům „uložte a zavřete") je v repu vědomě odložená - viz „Ověřit s autorem".

**UP-3. Srozumitelné selhání updatu** - nice to have (doplněk U1 a N11)
> When the mandatory update cannot proceed, the user learns why in plain terms and the client exits in an orderly way.

Evidence: `E2/.../Startup/Updater.cs` (rozlišené stavy: chybí VERZE.INF, nedostupný zdroj, neshoda verze programů a Caché objektů, nespustitelný updater), legacy `clsUpdate.vb` (kontroly práv adresářů, hlášky o firewallu/antiviru).
N11 řeší jen „klient zůstane spustitelný"; co uživatel uvidí a udělá, neřeší nikdo.

**UP-4. Granularita version guardu** - nice to have (úprava U2)
> The version guard tolerates server-only changes, so running clients are forced out only by releases that change the client.

Evidence: `UPDATE_FLOW.md` §1 (patch segment verze tolerován, klienti se dotáhnou při dalším startu).
Kolize: U2 je formulován jednosměrně („rejects requests from stale clients") - bez protipólu by doslovné čtení vyžadovalo vyhazovat uživatele i při čistě serverové opravě, což je přesně to, co rychlé Caché nasazení (sekce 1.1 spec) nesmí dělat.

**UP-5. Pozastavení distribuce release** - nice to have (doplněk U5)
> The company can pause the distribution of a release, and clients then behave as if no newer release existed.

Evidence: `E2/.../Startup/Updater.cs` (BlokovatAktualizace), legacy `clsUpdate.vb`, `UPDATE_FLOW.md`.
Nástroj na zastavení rolloutu vadné verze; malé, ale provozně důležité.

**UP-6. Kdo smí spustit update serverové instalace** - nice to have
> On a server installation, only an authorized user can start the update, and the update does not start while other users work with the system.

Evidence: legacy `frmNoMDI.vb` (`AllowUpdate`: admin nebo právo UpdateEdison; `EdisonaNikdoNepouziva` otevře Aktivitu uživatelů), `RESPONSIBILITIES.md` §13.6.
Rozlišení server vs. klientská instalace (klient = vždy vynucený update, server = řízený) spec nezná.

**UP-7. Detaily enginu (rollback, diferenciální kopie, předstažení, TEST kanál, dev build)** - spíš nezanášet
Evidence: `UPDATE_FLOW.md` §4, legacy `clsUpdate.vb`. U1 explicitně připouští ponechání legacy apply enginu, takže jeho vnitřní vlastnosti jsou pod rozlišovací úroveň. Výjimka: pokud by se engine přece jen nahrazoval, rollback („failed update leaves the client startable" z N11) by měl dostat vlastní větu.

---

## Toolkit

**TK-1. Server-driven formuláře** - essential (úprava T2)
> Input forms are defined and validated by the backend, so a form change does not require redistributing clients.

Evidence: `E2/EDControls/dotnet/EDInputForm/` + `EDControls/CACHE/.../EDInputForm/Form.cls` (schéma, viditelnosti, validace Init/Change/BeforeSave na serveru).
T2 říká jen „input forms". Server-driven charakter je kvalitativně jiný požadavek a přímý případ N5 (konfigurovatelnost bez redistribuce) - u listu to spec zdůrazňuje („server-driven data list"), u formulářů ne.

**TK-2. Persistence geometrie oken + uživatelský reset** - nice to have
> Windows remember their position and size per user and company, and the user can reset all window layouts to their defaults.

Evidence: `E2/EDControls/dotnet/WindowStatePersistence.cs`, `E2/.../EdisonMainViewModel.ResetForms`, legacy `ToolResetForms`.
S7/T5 mluví o preferencích obecně; reset je záchrana po odpojení monitoru („okno se otevírá mimo obrazovku") a bez něj je persistence dvousečná.

**TK-3. Potvrzení neuložených změn zdarma** - nice to have
> A window with unsaved changes asks for confirmation before it closes, provided by the toolkit rather than by each window.

Evidence: `E2/EDControls/dotnet/EDFooter/CLAUDE.md`.
Developer-facing schopnost přesně v duchu T skupiny (agenda ji dostane bez vlastního kódu).

**TK-4. Spustitelný testovací harness** - nice to have (úprava T6/N3)
> The toolkit ships test doubles of the backend access layer and a test runner for the Caché side, so agenda logic is testable on both stacks without a live installation.

Evidence: `E2/EDTesting/dotnet/MLRestMocks.cs`, `E2/EDTesting/CACHE/.../MLTestCase.cls`, `tools/run-cache-tests.ps1`.
T6 slibuje „test patterns" jako dokumentaci; realita je spustitelná knihovna a vlastní běhové prostředí testů pro Caché (deliverable MS1). Silnější a měřitelnější formulace.

**TK-5. Spouštěcí kontrakt pro nové aplikace** - nice to have
> A defined launch contract lets any application, not only bridged legacy agendas, start with the session and company context established by the shell.

Evidence: `E2/EDControls/dotnet/EdisonLaunchArgs.cs`, `WindowNavigator.cs`.
S6 kryje legacy bridge; obecný kontrakt je mechanismus, kterým TemplateDesigner přejde na sdílenou session (spec to v Architecture slibuje, ale žádný požadavek to nenese).

**TK-6. Export a tisk dat** - ověřit s autorem, potenciálně nice to have
> Data lists can export their contents, and printing is available where the legacy client offers it.

Evidence: `.claude/reviews/arch-backlog.md` R13a (API seam existuje, implementace ne), legacy `EDMiniApplication.vb` (tisk mini-apps), `LEGACY_FEATURE_INVENTORY.md` §6.
V ERP jde o jednu z nejviditelnějších funkcí a spec o exportu/tisku nemá ani slovo. Z repa ale nejde poznat, jestli je to v plánu projektu, nebo až po něm - proto primárně do sekce k ověření.

---

## Non-functional

**NF-1. Responzivita UI** - essential
> No backend call blocks the UI thread: long operations run under a progress overlay and the window stays responsive.

Evidence: `E2/EDControls/DOCS/00-overview.md` pravidlo R1, `arch-backlog.md` R2 (zamrzlé UI na VPN, měření 956 ms paralelně vs. 5041 ms sekvenčně).
N6 měří jen start shellu a spuštění agendy; responzivita při běžné práci (pomalá linka, VPN) chybí, přitom je to závazné pravidlo celého toolkitu a měřitelná kvalita.

**NF-2. Automatické hlášení chyb podpoře** - essential
> Client errors are reported automatically to the company's support backend, and the user can add a description of what they were doing.

Evidence: `E2/EDControls/dotnet/ErrorReporting/` (deduplikace, throttling, snímek vlastních oken, fatální chyby synchronně), vlastní serverový kontrakt.
Samostatný netriviální subsystém, ve spec zcela chybí. Pozor: součástí je snímek obrazovky vlastních oken - má GDPR rozměr, hodí se zmínit v N11 nebo u tohoto požadavku.

**NF-3. Pád jedné části nesmí zabít klienta** - essential
> An unhandled error in one window does not terminate the client, and agendas launched by the shell never outlive it.

Evidence: `E2/EDISONMAIN2/dotnet/App.xaml.cs` (globální handler), `EDControls/dotnet/EDViewModel/KillOnExit.cs` (Job Object: pád shellu ukončí agendy, žádné osiřelé procesy), legacy `ApplicationEvents.vb`.
Dvě strany téže robustnosti; druhá půlka navíc garantuje uvolnění licenčního slotu (legacy bolest).

**NF-4. Byznys zámky napříč klienty** - essential (konkretizace N1)
> Record locks taken by one client are honored by the other, and all locks of a session are released at logout.

Evidence: `MVP_PLAN_v1.md` DoD kritérium 3, `RESPONSIBILITIES.md` §3.9-3.10 (dnes se při náhlém odhlášení neuvolní).
N1 říká „no breaking changes" abstraktně; tohle je testovatelný scénář, na kterém se koexistence reálně pozná.

**NF-5. Telemetrie používání** - essential, ale ověřit záměr
> The client records which windows and controls are used and sends the aggregate to the backend, and the collection can be switched off.

Evidence: `E2/EDControls/dotnet/EDViewModel/EDUsageTelemetry.cs`, serverový kontrakt s testem (`TelemetryContractTest.cls`).
Přímý přínos pro projekt (data „co se používá" řídí, co portovat dál) a plošně zapnutá schopnost. Má privacy rozměr (sledování práce uživatele) - pokud je to záměr, patří do spec explicitně i s vypínačem; viz „Ověřit s autorem".

**NF-6. Panel podpory na každém okně** - nice to have
> Every window offers a support panel with the company's support channels and version information that identifies the window's context.

Evidence: `E2/EDControls/dotnet/EDSupportPanel/` (feedback nese název formuláře, verzi, namespace).
Nahrazuje legacy menu Helpdesk/TeamViewer/Nápověda jedním mechanismem; viditelné při demu.

**NF-7. Kompatibilní evoluce uloženého stavu** - nice to have
> A client upgrade never loses or corrupts stored user preferences: unknown or damaged stored state falls back to defaults.

Evidence: `E2/EDControls/DOCS/05-userdata-scopes.md` (pravidla evoluce, registr scope, reakce na schema mismatch).
Měřitelná kvalita persistence z S7/T5, kterou spec nevyslovuje.

**NF-8. Konfigurovatelný limit nečinnosti** - nice to have (případ N5)
> The inactivity limit is configured on the server, with an optional per-user override.

Evidence: `RESPONSIBILITIES.md` §3.5/§3.9 (kiosk, dispečink 24/7), legacy Nastavení („Systém ukončit po nečinnosti").
S8 má inactivity logout; kdo a kde limit nastavuje, chybí - a je to přesně případ pro N5.

**NF-9. Vzhled a písmo jako sdílené uživatelské nastavení** - nice to have (úprava T3/N8)
> The user can switch the theme and adjust the application font at runtime, the whole UI scales with the font, and the choice is shared by the m-line applications on the workstation.

Evidence: `E2/EDControls/dotnet/UserFonts/`, `EDTheme/ThemeService.cs` (cross-process přes %APPDATA%\m-line\common), `LEGACY_FEATURE_INVENTORY.md` §11 (legacy standard 9,75 pt - accessibility důvod).
T3/N8 kryjí existenci a konzistenci témat, ne runtime přepínání ani škálování písma. Škálování písma je jediná explicitně accessibility funkce projektu.

**NF-10. Rozlišitelnost klientů v administraci** - nice to have (doplněk N2)
> An administrator can tell new-client sessions from legacy ones in the administration views.

Evidence: `RESPONSIBILITIES.md` §18.2 (dnes nelze; doporučení ClientKind), v MVP vědomě škrtnuto.
Bez toho je diagnostika pilotu (N2, MS7) slepá: „komu se to děje - starým, nebo novým klientem?"

**NF-11. Audit administrátorských zásahů** - nice to have
> Administrative interventions (forced logouts, licence and permission changes) leave an audit record.

Evidence: `RESPONSIBILITIES.md` §2.9 (kickedBy, timestamp, reason), §7.
Váže se k LI-1; bez auditu je kick-and-login u zákazníka těžko obhajitelný.

**NF-12. Diagnostika u zákazníka** - ověřit s autorem
> The user can send diagnostic data to support directly from the shell.

Evidence: legacy `formEDISONMAINLogList.vb` (výběr logů, odeslání e-mailem), v EDISON2 jen bridge na legacy okno; `docs/archive/PLAN.md` vědomě „žádný logging framework".
Spec nemá o log/diagnostice nic. Rozhodnutí „bez logů, hlášení chyb stačí" může být v pořádku, ale mělo by být vyslovené (NF-2 je pak jediný diagnostický kanál).

---

## Kolize a zpětná zjištění (spec vs. realita)

Nejsou to kandidáti na nové požadavky, ale při zanášení výše uvedeného na ně autor narazí:

1. **N7 vs. lokalizační infrastruktura.** N7: „the UI is Czech. Multi-language support is not a goal." Repo ale plánuje lokalizační infrastrukturu se serverovým slovníkem (`docs/archive/PLAN.md` feature #4: „obsahově dodat jen CZ", „každá iterace prohlubuje vázanost na češtinu") a legacy má lokalizaci jako licencované právo (`AllowLocalization`) s daty u zákazníků. Doporučení: v N7 rozlišit jazyk dodávky (čeština) od připravenosti architektury, a říct, co bude s existujícími zákaznickými překlady.
2. **N8/T3 vs. per-firma barevné schéma.** Legacy přebarvuje shell podle firmy (ochrana proti záměně produkce/test, `frmNoMDI.RefreshMainformScheme`), data tečou serverem dodnes. Jednotné design tokeny to nepředpokládají - buď to vědomě opustit, nebo do theming požadavku doplnit.
3. **M4 není v kódu.** Launcher nabízí všechny registrované mini-apps, licenční gate chybí (vědomě odloženo, `MINIAPPS.md` §10). Spec je forward-looking, takže M4 může zůstat, jen ať autor ví, že to realita zatím neplní - stejně jako S11 (status bar dnes expiraci licence neukazuje).
4. **U2 jednosměrnost.** Viz UP-4 - doslovné čtení U2 koliduje s principem rychlého server-side nasazení ze sekce 1.1.

---

## Ověřit s autorem

Z repa nejde poznat, jestli jde o záměr projektu, vedlejší produkt vývoje, nebo vědomě opouštěnou legacy funkci:

1. **Tray aplikace a systém zpráv mezi uživateli** (EDTRAY: schránky, adresáti, reakce s timeoutem, odmítnutí vynuceného vypnutí). Integrace je v legacy z velké části zakomentovaná (polomrtvý kód), nový shell tray nemá. Pokud se opouští, stálo by za explicitní zmínku v out of scope - hlavně náhrada scénáře „správce potřebuje před updatem vyhnat uživatele" (dnes UP-2 + odpočet bez možnosti odmítnout).
2. **Správce připojení a výběr databáze/prostředí** (legacy: pojmenovaná připojení, přepnutí za běhu, admin zámek výchozího připojení, výběr namespace). V EDISON2 je „Připojit k databázi" zatím stub. Je runtime přepnutí prostředí cíl projektu, nebo stačí konfigurace při prvním spuštění (SH-1)?
3. **Inicializace serveru (greenfield setup: firma, licence, první admin)** zůstává legacy nástrojům - zanést do out of scope?
4. **Telemetrie používání (NF-5) a snímky oken v hlášení chyb (NF-2)**: je plošný sběr záměr projektu (a je právně ošetřen), nebo interní experiment? Ovlivňuje, jestli patří do spec a s jakými pojistkami.
5. **Export a tisk (TK-6)**: API seam existuje, implementace ne. V plánu projektu, nebo až po něm?
6. **Read-only režim při expirované licenci a DEMO licence** (legacy `MLProps.ReadOnlyMode`, DEMO gate na splashi). L2 zná jen varování. Přebírá se degradace do read-only, nebo expirace jen blokuje další login? Souvisí s otevřenou otázkou expirace za běhu session (`RESPONSIBILITIES.md` §23).
7. **Správa přístupových práv ve shellu** (legacy AccessTree + export práv do CSV). Zůstane legacy agendou přes bridge, nebo je v plánu nová? Spec práva jen konzumuje (S3).
8. **Úlohy na pozadí** (legacy formEDISONMAINTaskList: přehled serverových úloh napříč agendami). Portovat, bridge, nebo opustit?
9. **Uložení rozdělané práce před vynuceným odhlášením** - v repu explicitně zrušeno (`docs/archive/PLAN.md`, „aspirativní bez validovaného user story"). Uživatelsky je to ale nejcitelnější rozdíl proti očekávání; zvážit uvedení mezi vědomými omezeními S8.
10. **Broadcast běžícím uživatelům před updatem a vynucený re-sync bez změny verze** (`RESPONSIBILITIES.md` §12.3, `UPDATE_FLOW.md` §7) - vědomě odloženo pro pilot; patří do spec jako požadavek, nebo jako vědomé omezení?
11. **Vzdálená správa klientských stanic a plánované úlohy updatu** (legacy Nastavení: seznam PC, auto-download/auto-install, e-mail notifikace, TEST kanál) - kolik z toho má „modernizovaná distribuce" (U3) pokrýt?
12. **Testovací/ukázková okna komponent v produkčním buildu** („Test funkcí" menu) - záměrný showcase toolkitu (hodil by se k obhajobě), nebo dočasnost?
13. **Druhá instance shellu (SH-13)** - legacy inventura říká „neportovat", kód nového shellu to má. Co platí?
14. **Filtrace dat mini-apps funkčním omezením uživatele** (legacy UseFO; nový klient příznak neposílá, uživatel vidí data za všechny - `LEGACY_FEATURE_INVENTORY.md` §6). Potenciálně bezpečnostní díra, ne kosmetika.
15. **MCP žurnál a další AI nástroje v tools/** - jsou součást AI dokumentace/deliverable (N10), nebo vedlejší produkt?
16. **Nástěnky WebInfo na ploše legacy** (novinky M-line, daňový kalendář, upozornění e-mailem) - nahrazeno MA-1 widgety, nebo se část (daňový kalendář, e-mail notifikace) opouští?
