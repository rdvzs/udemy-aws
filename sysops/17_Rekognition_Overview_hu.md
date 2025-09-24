# 17. fejezet: Monitorozás, auditálás és teljesítmény

## Amazon CloudWatch
A CloudWatch egy monitorozási szolgáltatás az AWS erőforrások és alkalmazások számára.
-   **Metrikák:** Metrikákat biztosít minden AWS szolgáltatáshoz (pl. CPU kihasználtság, hálózati bemenet). A metrikák névterekhez tartoznak, és dimenziókkal rendelkezhetnek. Az EC2 példányok alapértelmezetten 5 percenként küldenek metrikákat; részletes monitorozás (1 perces metrikák) engedélyezhető költség ellenében. A memóriahasználat alapértelmezetten nem kerül továbbításra, és egyéni metrikát igényel.
-   **Egyéni metrikák:** Saját egyéni metrikákat (pl. memóriahasználat, lemezterület, bejelentkezett felhasználók) definiálhat és továbbíthat a `PutMetricData` API hívás segítségével. A metrikák lehetnek standard (1 perces) vagy nagy felbontásúak (1, 5, 10, 30 másodperc). A CloudWatch akár 2 héttel korábbi vagy 2 órával későbbi metrikákat is elfogad.
-   **Irányítópultok:** Kulcsfontosságú metrikák és riasztások vizualizálása több régióban és fiókban. Különböző widgeteket (vonal, halmozott terület, szám, szöveg, naplótábla, riasztási állapot) tartalmazhat. Három irányítópult (akár 50 metrika) ingyenes.
-   **Naplók:** Központosított szolgáltatás az alkalmazásnaplók tárolására.
    -   **Naplócsoportok:** Logikai csoportosításokat definiál a naplókhoz (pl. alkalmazásonként).
    -   **Naplófolyamok:** Példányokat, specifikus naplófájlokat vagy konténereket képviselnek egy naplócsoporton belül.
    -   **Napló megőrzés:** Konfigurálható 1 naptól korlátlan ideig.
    -   **Naplóforrások:** SDK, CloudWatch Unified Agent, Elastic Beanstalk, ECS, Lambda, VPC Flow Logs, API Gateway, CloudTrail, Route 53.
    -   **Logs Insights:** Lekérdezési motor a naplóadatok keresésére és elemzésére egy célzott lekérdezési nyelv segítségével. Támogatja a szűrést, aggregációt, rendezést és több naplócsoport lekérdezését fiókok/régiók között.
    -   **Napló exportálás:** Kötegelt exportálás S3-ba (akár 12 óra is lehet) a `CreateExportTask` segítségével.
    -   **Előfizetési szűrők:** Valós idejű naplóesemény-folyam a Kinesis Data Streams, Kinesis Data Firehose vagy Lambda számára feldolgozás és elemzés céljából. Összegyűjtheti a naplókat különböző fiókokból/régiókból.
    -   **Live Tail:** Valós idejű nézet a bejövő naplóeseményekről hibakeresés céljából.
-   **Riasztások:** Értesítések vagy műveletek kiváltása metrikus küszöbértékek alapján.
    -   **Állapotok:** OK, INSUFFICIENT_DATA, ALARM.
    -   **Célok:** EC2 műveletek (leállítás, leállítás, újraindítás, helyreállítás), Auto Scaling műveletek (skálázás ki/be), SNS értesítések (Lambda függvényeket is kiválthat).
    -   **Összetett riasztások:** Több más riasztás állapotának figyelése ÉS/VAGY feltételekkel a zaj csökkentése érdekében.
    -   **EC2 példány helyreállítása:** Automatikusan helyreállít egy EC2 példányt, ha az állapotellenőrzések sikertelenek, megőrizve az IP-t, a metaadatokat és az elhelyezési csoportot.
    -   **Riasztások tesztelése:** Használja a `set-alarm-state` CLI parancsot a riasztás manuális kiváltására tesztelés céljából.

## AWS Service Quotas
Szolgáltatás az AWS szolgáltatások kvótáinak (korlátainak) megtekintésére és kezelésére.
-   **Monitorozás:** Figyelje a kvótahasználatot, és állítson be CloudWatch riasztásokat, ha a használat megközelíti a küszöbértéket (pl. a limit 80%-a).
-   **Kvóta növelése:** Kérjen kvótanövelést közvetlenül a konzolról.
-   **Alternatíva:** A Trusted Advisor is ellenőrzi a szolgáltatási korlátokat, de a Service Quotas átfogóbb.

## AWS CloudTrail
Kormányzást, megfelelőséget és auditot biztosít az AWS fiókjához az API hívások és a felhasználói tevékenység rögzítésével. Alapértelmezetten engedélyezve van.
-   **Eseménytípusok:**
    -   **Menedzsment események:** AWS erőforrásokon végzett műveletek (pl. `IAM AttachRolePolicy`, `CreateSubnet`). Alapértelmezetten naplózva. Különválaszthatók az olvasási (nem módosító) és írási (módosító) események.
    -   **Adat események:** Nagy volumenű műveletek adatforrásokon (pl. S3 objektum szintű tevékenység, mint a `GetObject`, `PutObject`, Lambda függvény `Invoke`). Alapértelmezetten nem naplózva, költséggel jár.
    -   **Insights események:** Szokatlan tevékenységet észlel (pl. pontatlan erőforrás-kiépítés, szolgáltatási korlátok elérése, IAM műveletek kiugrásai). Engedélyezést és költséget igényel. Az anomáliák megjelennek a konzolon, elküldésre kerülnek az S3-ba, és EventBridge eseményeket váltanak ki.
-   **Esemény megőrzés:** Az események 90 napig tárolódnak a CloudTrail konzolon. Hosszabb megőrzéshez naplózza az S3-ba, és elemezze az Athena segítségével.
-   **Naplófájl integritás ellenőrzése:** Digest fájlokat (SHA-256 hash) generál a naplófájlok integritásának ellenőrzésére, amelyek az S3-ba kerültek, hogy nem manipulálták-e őket.
-   **Integráció az EventBridge-dzsel:** A CloudTrail események elküldésre kerülnek az EventBridge-be, lehetővé téve a szabályok számára, hogy műveleteket váltsanak ki (pl. SNS értesítés a `DeleteTable` API hívásról, `AssumeRole` API hívásról, `AuthorizeSecurityGroupIngress` API hívásról).
-   **Szervezeti nyomvonalak:** A CloudTrail konfigurálása az AWS Organizations szintjén, hogy naplózza az eseményeket az összes tagfiók számára egy központi S3 tárolóba. A tagfiókok nem módosíthatják a szervezeti nyomvonalat.

## AWS Config
Rögzíti a konfigurációs változásokat, és értékeli az AWS erőforrások megfelelőségét a szabályoknak.
-   **Erőforrás rögzítés:** Rögzíti a konfigurációt az összes támogatott erőforrásról egy régióban (beleértve a globális erőforrásokat, mint az IAM). A konfigurációs előzményeket az S3-ban tárolja.
-   **Szabályok:**
    -   **AWS által kezelt szabályok:** Több mint 75 előre definiált szabály (pl. `restricted-ssh`).
    -   **Egyéni szabályok:** Lambda függvények segítségével definiálhatók (pl. EBS lemez típusának, EC2 példány típusának ellenőrzése).
    -   **Kiváltás:** A szabályok konfigurációs változások vagy rendszeres időintervallumok alapján válthatók ki.
-   **Megfelelőség:** Az erőforrások megfelelőnek vagy nem megfelelőnek minősülnek. A Config szabályok nem akadályozzák meg a műveleteket, de auditálást biztosítanak.
-   **Helyreállítás:** Automatikusan vagy manuálisan helyreállítja a nem megfelelő erőforrásokat az SSM Automation Documents segítségével (pl. régi IAM hozzáférési kulcsok deaktiválása). Konfigurálható újrapróbálkozások.
-   **Értesítések:** Konfigurációs változásokról és megfelelőségi értesítésekről küldhet SNS-re vagy EventBridge eseményeket válthat ki.
-   **Aggregátorok:** Központosítja a konfigurációs és megfelelőségi adatokat több fiókból és régióból egyetlen aggregátor fiókba. Ha AWS Organizations-t használ, az engedélyezés automatikus.
-   **CloudFormation StackSets:** A Config szabályok telepítésére szolgál több fiókban és régióban.

## AWS X-Ray
Segít a fejlesztőknek elosztott alkalmazások (mikroszolgáltatások, szerver nélküli, webes alkalmazások) elemzésében és hibakeresésében.
-   **Komponensek:**
    -   **X-Ray SDK:** Az alkalmazáskódba integrálva adatküldés céljából.
    -   **X-Ray Daemon:** Adatokat gyűjt az SDK-ból, és elküldi az X-Ray szolgáltatásnak.
    -   **X-Ray Service:** Tárolja és feldolgozza az adatokat.
-   **Funkciók:**
    -   **Szolgáltatástérkép:** Az alkalmazás komponenseinek és kapcsolataik vizuális megjelenítése.
    -   **Nyomkövetések:** Az egyes kérések részletes nézete, amely a késleltetést és a hibákat mutatja a szolgáltatások között.
    -   **Anomáliák:** Hibák és hibák azonosítása.
-   **Integráció:** Integrálódik az EC2, Lambda, ECS, EKS, Elastic Beanstalk, API Gateway szolgáltatásokkal.
-   **Felhasználási esetek:** Teljesítményelemzés, szűk keresztmetszetek azonosítása, hibaelhárítás.

## CloudWatch Synthetics Canary
Konfigurálható szkriptek (Node.js vagy Python) futtatása a CloudWatchból API-k, URL-ek vagy weboldalak monitorozására.
-   **Funkcionalitás:** Ügyfélműveletek reprodukálása (pl. bejelentkezés, kosárba helyezés, fizetés), végpontok elérhetőségének/késleltetésének ellenőrzése, betöltési idő adatok tárolása, UI képernyőképek készítése.
-   **Tervek:** Heartbeat Monitor, API Canary, Broken Link Checker, Visual Monitoring, Canary Recorder (CloudWatch Synthetics Recorderrel), GUI Workflow Builder.
-   **Felhasználási esetek:** Proaktív monitorozás, problémák felderítése az ügyfelek előtt, felhasználói folyamatok tesztelése.