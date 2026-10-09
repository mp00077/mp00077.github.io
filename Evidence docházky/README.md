# Dochazkovy system

Python GUI aplikace (`PySide6/Qt`) pro evidenci zamestnancu a dochazky.
Aplikace je navrzena pro macOS i Windows.



## Funkce
1. Evidence zamestnancu
2. Ukladani dochazky do SQLite databaze
3. Export reportu do XLS
4. Automaticke testy (unit + integration)
5. Samostatne GUI pro editaci uzivatelu a jejich hromadne upravy (menu `Administrace -> Sprava uzivatelu`)
6. Role a opravneni (`admin`, `manager=vedouci`, `employee=zamestnanec`)
7. Automaticke vytvoreni profilu podle OS uzivatele pri prvnim spusteni
8. Viditelnost dochazky podle role (zamestnanec = sebe, vedouci = sebe + podrizeni)
9. Typ dne bez casu, seznam typu je spravovatelny v menu `Administrace -> Nastaveni -> Typ celeho dne`; standardne se zapocita automaticky 8 hodin, vyjimka je `Lékař`, ktery se zapocita jako 5 hodin; pokud je zvolen Typ celeho dne, nelze soucasne ulozit `Prichod`, `Obed od` ani `Odchod`
10. Kontrola nekompletnich dni probiha az nasledujici den; otevre se formular pro doplneni chybejicich udaju
11. V editoru dne lze zvolit `Odchod v prac. dobu` (od/do/duvod). Casove okno, kdy se preruseni neodecita, je nastavitelne v menu `Administrace -> Nastaveni -> Pevna doba` (po dnech Po-Ne). Casy `Prichod`, `Obed odchod` a `Odchod` lze zadavat jen v intervalu 15 minut.
    - vypocet: z intervalu odchodu se odecte pouze cast mimo povolene okno dne
    - Po-Pa: pouzije se nastavene okno `Pevna doba od` az `Pevna doba do`
    - So-Ne: preruseni se odecita cele (bez neodecteneho okna)
    - odecitana hodnota = celkova delka odchodu - prekryv s povolenym oknem, min. 0
12. Pri prvnim prihlaseni se aplikace pokusi nacist jmeno a prijmeni z Active Directory podle `getpass.getuser()` a nastavene OU cesty (`Administrace -> Nastaveni -> Nastaveni AD`, pouze admin)
13. Menu `Vedouci` (pro role `admin` a `manager`) obsahuje polozku `Seznam` s prehledem employee: dnesni den, prichod, obed odchod, odchod, preruseni prac. doby a saldo
14. Menu `Vedouci` obsahuje polozku `Zamestnanci - saldo` se seznamem podrizenych a jejich saldem za cely aktualni pracovni cyklus
15. Menu `Vedouci` obsahuje polozku `Hromadny export XLS`, kde lze pomoci checkboxu vybrat vice podrizenych uzivatelu a exportovat pro ne aktualni nebo vybrany cyklus bud do samostatnych `.xlsx` souboru v jedne cilove slozce, nebo do jednoho `.xlsx` souboru s listy podle uzivatelu
16. Menu `Administrace -> Sprava uzivatelu -> Import z AD` (pouze admin) umi nacist uzivatele z CN skupiny v AD, vybrat je pomoci checkboxu a importovat vybrane uzivatele do DB
17. Menu `Zobrazit` (pro vsechny role) obsahuje volbu `Vybrat aktualni cyklus` se seznamem vsech cyklu
18. Menu `Zobrazit -> Detail dochazky` otevre samostatne okno pro prihlaseneho uzivatele s detailem pracovniho cyklu ve stejnem formatu jako frame `Aktualni pracovni cyklus - detail`; obsahuje combobox pro vyber cyklu, defaultne otevira aktualni pracovni cyklus, navic obsahuje sloupec `Odchod v prac. dobu` ve tvaru `od - do = duvod`, a okno se nacita az pri otevreni (lazy import)
19. Menu `Zobrazit -> Nevyplnene dny` zobrazi jiz existujici okno s nevplnenymi dny pro prave vybraneho uzivatele; stejne okno se automaticky otevre i pri startu aplikace, pokud existuji nevplnene dny
20. Menu `Administrace -> Sprava uzivatelu -> Hierarchie` (pouze admin) zobrazi stromovy diagram vedoucich a jejich podrizenych podle nastavene vazby vedouciho; pokud ma vedouci dalsiho vedouciho, je v diagramu odpovidajicim zpusobem odsazen, a okno umoznuje export aktualni hierarchie do souboru `.rtf`
21. Menu `Administrace -> Databaze` (pouze admin) obsahuje tlacitka `Export` a `Import` pro migraci dat pri zmene struktury DB
    - `Export` ulozi data DB do JSON souboru (zamestnanci, dochazka, typy dne, stavy dne, preruseni prac. doby, app nastaveni)
    - `Import` nacte JSON export a nahradi aktualni data v DB
22. Menu `Administrace -> Sprava uzivatelu -> Editace uzivatelu` umoznuje zvolit role a vedouciho
23. Menu `Administrace -> Sprava uzivatelu -> Hromadne upravy` umoznuje vybrat vice uzivatelu pomoci checkboxu a hromadne zmenit jejich roli a vedouciho
24. Ve frame `Aktualni pracovni cyklus - detail` se `Odpracovano`, denni soucet hodin a `Saldo` zobrazuji ve formatu `hh:mm`; stejny format pouziva i export cyklu do XLS

## Dochazkove cykly
- Cykly jsou pevne 4tydenni bloky po tydnech v roce: 1-4, 5-8, 9-12, ...
- Kazdy cyklus je od pondeli do nedele (28 kalendarnich dnu).
- V ramci roku se cykly radi podle cisla tydne (ISO tydny).
- Pracovni norma: 8 hodin denne, 40 hodin tydne.
- V GUI se zobrazuje cely aktualni cyklus podle aktualniho data, vcetne normy hodin a odpracovanych hodin vybraneho zamestnance.
- Ve frame `Aktualni pracovni cyklus - detail` se norma, `Odpracovano` i `Saldo` pocitaji pres minuty a zobrazuji se jako `hh:mm` misto desetinneho poctu hodin.
- Pres menu `Zobrazit -> Vybrat aktualni cyklus` lze zobrazit i jiny cyklus; pokud nejde o cyklus podle dnesniho data, GUI pouziva oznaceni `Vybrany cyklus`.
- Pres menu `Zobrazit -> Nevyplnene dny` lze znovu otevrit okno pro doplneni chybejicich dnu, pokud pro vybraneho uzivatele existuji.
- Pres menu `Administrace -> Sprava uzivatelu -> Hierarchie` muze admin zobrazit strom vedoucich a podrizenych uzivatelu a vyexportovat jej do `RTF`.
- Pri editaci dne plati, ze `Typ celeho dne` a casove udaje (`Prichod`, `Obed od`, `Odchod`) se navzajem vylucuji a nelze je ulozit soucasne.
Po spusteni otevres formular pro pridani/upravu/smazani uzivatelu v menu `Administrace -> Sprava uzivatelu -> Editace uzivatelu`.

## Role a pristupy
- Identita uzivatele se bere z OS loginu.
- Pokud je nastavena OU cesta v `Administrace -> Nastaveni -> Nastaveni AD`, aplikace se pri vytvareni noveho uzivatele pokusi nacist `jmeno/prijmeni` z AD.
- Kdyz se uzivatel v AD nenajde (nebo AD neni dostupne), pouzije se stavajici dialog pro rucni zadani jmena/prijmeni.
- `Administrace -> Sprava uzivatelu -> Import z AD` slouzi k hromadnemu importu vybranych uzivatelu z CN skupiny v AD (po nacteni seznamu a potvrzeni `Importovat`).
- Pri prvnim spusteni nove DB se prvni uzivatel vytvori jako `admin`.
- `employee` (`zamestnanec`) vidi a upravuje jen svou dochazku.
- `manager` (`vedouci`) vidi a upravuje svou dochazku + podrizene.
- `admin` vidi a spravuje vsechny zamestnance.
- Pro role `admin` a `manager` je dostupne menu `Vedouci -> Seznam` s dennim prehledem employee.
- Pro role `admin` a `manager` je dostupne menu `Vedouci -> Zamestnanci - saldo` se saldem employee za aktualni cyklus.
- Pro role `admin` a `manager` je dostupne menu `Vedouci -> Hromadny export XLS`; manager vidi uzivatele, u kterych je vedoucim, admin vidi vsechny role, a export lze provest bud do samostatnych souboru, nebo do jednoho souboru s listy podle uzivatelu.
- Export CSV respektuje stejna pravidla viditelnosti.
- role `employee` (`zamestnanec`) může editovat:
    dnešní­ den a předchozí­ pracovní­ den (Po-Pá, s přeskočením víkendu)

## Upozorneni k databazi
- Casove udaje dochazky (`prichod`, `obed odchod`, `odchod`, `odchod v prac. dobe`) jsou sifrovane pomoci `cryptography` (Fernet).
- Sifrovaci heslo se pri prvnim spusteni automaticky vygeneruje a ulozi do `db/.dochazka_time_password` (vedle DB).
- Volitelne lze heslo dodat pres prostredi `DOCHAZKA_TIME_PASSWORD`.
- Kvuli kompatibilite umi aplikace cist i drive sifrovana data (legacy klic).
- Stare schema se nemigruje.
- Pred prvnim spustenim nove verze aplikace rucne smaz `db/attendance.db`.
- Nova DB se vytvori automaticky pri startu aplikace.

## Data
- Databaze: `db/attendance.db` (v adresari `db`)
- CSV export: vyberes cestu pri exportu v GUI
- XLS export: vyberes cestu pri exportu v GUI
- Hromadny XLS export: v GUI lze zvolit rezim `Samostatne soubory` nebo `Jeden soubor, listy podle uzivatelu`; v prvnim pripade vyberes cilovou slozku, ve druhem konkretni vystupni `.xlsx` soubor

## Troubleshooting

