# 24. fejezet: Egyéb szolgáltatások

## AWS X-Ray
Az AWS X-Ray egy szolgáltatás az elosztott alkalmazások nyomon követésére és vizuális elemzésére. Segít a teljesítményproblémák hibakeresésében, a mikroszolgáltatások függőségeinek megértésében, a szolgáltatási problémák azonosításában, a kérés viselkedésének áttekintésében és a hibák/kivételek azonosításában.
-   **Felhasználási esetek:** Elosztott nyomkövetés, hibaelhárítás, szolgáltatásgráf vizualizáció.

## AWS Amplify
Webes és mobil alkalmazásfejlesztő eszköz, amely különböző AWS szolgáltatásokat integrál egyetlen platformba.
-   **Amplify CLI:** Amplify back-endek létrehozására szolgál (S3, Cognito, AppSync, API Gateway, SageMaker, Lex, Lambda, DynamoDB stb. felhasználásával).
-   **Frontend könyvtárak:** Webes és mobil alkalmazások csatlakoztatása az Amplify back-endhez.
-   **Amplify Console:** Telepítésre szolgál az Amplify-ra és a CloudFrontra.
-   **Analógia:** Gondoljon rá úgy, mint az Elastic Beanstalkra webes és mobil alkalmazásokhoz.

## AWS AppSync
Teljesen menedzselt szolgáltatás GraphQL API-k fejlesztéséhez.
-   **GraphQL:** Lekérdezési nyelv API-khoz és futásidejű környezet a lekérdezések meglévő adatokkal való teljesítéséhez. Alternatíva a REST API-khoz.
-   **Előnyök:** Rugalmas (egyetlen végpont több adatforráshoz), hatékony (csak a szükséges adatokat kéri le), skálázható, biztonságos (több hitelesítési mód).
-   **Komponensek:**
    -   **Séma:** Meghatározza az adattípusokat és a műveleteket.
    -   **Feloldók:** Összekötik a sémát az adatforrásokkal (DynamoDB, Lambda, RDS, Elasticsearch, HTTP végpontok).

## AWS IoT Core
Teljesen menedzselt szolgáltatás milliárdnyi IoT eszköz AWS Cloudhoz való csatlakoztatására.
-   **Jellemzők:** Skálázható, biztonságos (kölcsönös hitelesítés, titkosítás), rugalmas (több protokoll).
-   **Funkcionalitás:** Eszközök csatlakoztatása, adatok továbbítása más AWS szolgáltatásokhoz (Lambda, Kinesis, S3, DynamoDB, CloudWatch), eszközök kezelése (regisztrálás, monitorozás, frissítés).

## AWS Device Farm
Teljesen menedzselt szolgáltatás mobil alkalmazások tesztelésére valós eszközökön az AWS Cloudban.
-   **Jellemzők:** Skálázható, rugalmas (több keretrendszer), biztonságos (izolált tesztkörnyezetek).
-   **Funkcionalitás:** Alkalmazás és tesztek feltöltése, valós vagy virtuális eszközök kiválasztása, tesztek futtatása, eredmények megtekintése (képernyőképek, naplók, teljesítményadatok).

## AWS WorkSpaces
Teljesen menedzselt szolgáltatás felhőalapú virtuális asztalok biztosítására a felhasználók számára.
-   **Jellemzők:** Skálázható, rugalmas (több operációs rendszer), biztonságos (izolált virtuális asztalok).
-   **Funkcionalitás:** WorkSpace-ek kiépítése, felhasználók bármilyen eszközről csatlakozhatnak, rendszeres asztalként használhatják.

## AWS AppStream 2.0
Teljesen menedzselt szolgáltatás asztali alkalmazások streamelésére bármilyen eszközre.
-   **Jellemzők:** Skálázható, rugalmas (több operációs rendszer), biztonságos (izolált alkalmazásfolyamok).
-   **Funkcionalitás:** Alkalmazás feltöltése, streamelés a felhasználókhoz, felhasználók bármilyen eszközről hozzáférhetnek, rendszeres asztali alkalmazásként használhatják.

## AWS Elastic Transcoder
Teljesen menedzselt szolgáltatás médiafájlok egyik formátumból a másikba való konvertálására.
-   **Jellemzők:** Skálázható, rugalmas (több formátum), költséghatékony (pay-as-you-go).
-   **Funkcionalitás:** Média feltöltése S3-ba, Transcoder feladat létrehozása, fájlok konvertálása, kézbesítés a felhasználóknak.

## AWS WorkMail
Teljesen menedzselt szolgáltatás e-mail és naptár hosztolására a felhasználók számára.
-   **Jellemzők:** Skálázható, rugalmas (több kliens), biztonságos (titkosítás nyugalmi állapotban és átvitel közben).
-   **Funkcionalitás:** WorkMail szervezet létrehozása, felhasználók létrehozása, felhasználók hozzáférhetnek e-mailjeikhez és naptárukhoz bármilyen kliensről.

## AWS WorkDocs
Teljesen menedzselt szolgáltatás dokumentumok és fájlok tárolására, megosztására és együttműködésére.
-   **Jellemzők:** Skálázható, rugalmas (több fájltípus), biztonságos (titkosítás nyugalmi állapotban és átvitel közben).
-   **Funkcionalitás:** WorkDocs oldal létrehozása, felhasználók létrehozása, felhasználók bármilyen eszközről hozzáférhetnek dokumentumaikhoz és fájljaikhoz.

## AWS WorkLink
Teljesen menedzselt szolgáltatás belső weboldalakhoz és webes alkalmazásokhoz való biztonságos hozzáférés biztosítására.
-   **Jellemzők:** Skálázható, rugalmas (több böngésző), biztonságos (izolált webes munkamenetek).
-   **Funkcionalitás:** WorkLink konfigurálása belső weboldalakhoz, felhasználók bármilyen eszközről hozzáférhetnek, rendszeres webböngészőként használhatják.

## AWS Cost Explorer
Az AWS költségeinek és felhasználásának vizualizálására, megértésére és kezelésére szolgál az idő múlásával.
-   **Jelentések:** Egyéni jelentések a költség- és felhasználási adatokról (havi, óránkénti, erőforrás szintű).
-   **Előrejelzés:** Akár 12 hónapra előrejelzi a felhasználást.

## AWS Budgets
Költség- és felhasználási riasztásokat hoz létre.
-   **Költségvetés típusok:** Költség, felhasználás, foglalás, megtakarítási terv.
-   **Riasztások:** Értesítéseket küld, ha a tényleges vagy előre jelzett költségek/felhasználás meghaladja a meghatározott küszöbértékeket.
-   **Műveletek:** Automatikus műveleteket indíthat (pl. IAM házirend csatolása, EC2/RDS példányok leállítása), ha a költségvetési küszöbértékeket túllépik.

## AWS Cost Allocation Tags
Felhasználó által definiált címkék (pl. `Environment:Production`) és AWS által generált címkék (pl. `aws:createdBy`), amelyeket az AWS költségeinek részletes nyomon követésére használnak a költség- és felhasználási jelentésekben.

## AWS Cost and Usage Reports
Átfogó jelentések az AWS költségeiről és felhasználásáról, beleértve a részletes metaadatokat és a költségallokációs címkéket. S3-ba kézbesítve.