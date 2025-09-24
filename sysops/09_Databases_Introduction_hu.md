
# 9. Fejezet: Lambda SysOps Szemszögből

Ez a fejezet az AWS Lambda szolgáltatást vizsgálja SysOps nézőpontból, az alapkoncepciókra, az eseményvezérelt architektúrára, a jogosultságokra, a teljesítményre és a monitorozásra összpontosítva.

## 9.1: Lambda Áttekintés
- **EC2 vs. Lambda:**
  - **EC2:** Virtuális szerverek, amelyeket Önnek kell kiépítenie és kezelnie. Folyamatosan futnak, és a skálázásért Ön a felelős.
  - **Lambda:** Szerver nélküli (serverless) számítási szolgáltatás. A kódot „függvényekként” tölti fel, amelyek igény szerint futnak. Nem kell szervereket kezelnie.
- **Főbb Lambda Jellemzők:**
  - **Eseményvezérelt (Event-Driven):** A függvényeket más AWS-szolgáltatásokból származó események hívják meg.
  - **Rövid Végrehajtások:** A függvényeknek időkorlátjuk van, maximum 15 perc.
  - **Automatikus Skálázás:** A Lambda automatikusan skálázódik néhány kéréstől akár másodpercenkénti több ezerig.
  - **Árazás:** Használat alapú fizetés, a kérések száma és a számítási időtartam (GB-másodpercben) alapján.
- **Támogatott Futtatási Környezetek:** Támogatja a népszerű nyelveket, mint a Node.js, Python, Java, Go és Ruby. Egyéni futtatókörnyezetek és konténerképek is támogatottak.
- **Gyakori Integrációk (Eseményforrások):** API Gateway, S3, DynamoDB, SQS, SNS, CloudWatch Events (EventBridge), Kinesis és még sok más.

## 9.2: Lambda Eseményindítók (Triggers)
- **CloudWatch Events / EventBridge:** A Lambda függvények indításának gyakori módja.
  - **Ütemezett Események:** Egy függvényt ismétlődő ütemezés szerint indít el (pl. óránként), létrehozva egy szerver nélküli CRON feladatot.
  - **Eseményalapú Szabályok:** Egy függvényt az AWS-fiókjában bekövetkező eseményekre válaszul indít el (pl. egy EC2 példány állapotváltozása, egy CodePipeline fázis változása).
- **S3 Eseményértesítések:** Egy Lambda függvényt indít el S3 vödör eseményekre válaszul (pl. `s3:ObjectCreated:*`). Ez egy klasszikus minta olyan felhasználási esetekre, mint a kép-bélyegképek automatikus létrehozása.
- **Meghívási Típusok:**
  - **Aszinkron Meghívás:** Az olyan szolgáltatások, mint az S3 és az EventBridge, aszinkron módon hívják meg a Lambdát. Ha a meghívás korlátozás (throttling) vagy rendszerhibák miatt meghiúsul, a Lambda automatikusan újrapróbálkozik akár hat órán keresztül, mielőtt az eseményt egy Dead-Letter Queue-ba (DLQ) küldené, ha az konfigurálva van.

## 9.3: Lambda Jogosultságok (IAM Szerepkörök és Erőforrás Házirendek)
- **Végrehajtási Szerepkör (Execution Role):** Minden Lambda függvénynek van egy IAM szerepköre, amely jogosultságokat ad neki más AWS-szolgáltatások és erőforrások eléréséhez. Például a naplók írásához jogosultságra van szüksége a CloudWatch Logs-hoz (`AWSLambdaBasicExecutionRole`). Egy SQS üzenetsorból való olvasáshoz SQS jogosultságokra van szüksége.
- **Erőforrás-alapú Házirend (Resource-Based Policy):** Egy házirend, amely közvetlenül magához a Lambda függvényhez van csatolva. Ez más AWS-szolgáltatásoknak vagy fiókoknak ad engedélyt a Lambda függvény *meghívására*. Így kap engedélyt például az S3 és az EventBridge a függvény indítására. A konzol gyakran automatikusan beállítja ezt, amikor létrehoz egy eseményindítót.
- **Fő Különbség:**
  - A **Végrehajtási Szerepkör** azt határozza meg, hogy a Lambda függvény mit *tehet meg*.
  - Az **Erőforrás Házirend** azt határozza meg, hogy ki *hívhatja meg* a Lambda függvényt.

## 9.4: Lambda Monitorozás és Nyomkövetés
- **CloudWatch Logs:** A Lambda függvény minden kimenete (pl. `print` vagy `console.log` utasítások) és végrehajtási összefoglalója automatikusan egy dedikált naplócsoportba kerül a CloudWatch Logs-ban.
- **CloudWatch Metrikák:** A Lambda automatikusan közzétesz számos kulcsfontosságú metrikát, többek között:
  - `Invocations`: A függvény meghívásainak száma.
  - `Duration`: A függvény végrehajtási ideje.
  - `Errors`: A sikertelen meghívások száma.
  - `Throttles`: A párhuzamossági korlátok miatt visszautasított meghívások száma.
  - `ConcurrentExecutions`: Az eseményeket egyidejűleg feldolgozó függvény-példányok száma.
- **AWS X-Ray:** Elosztott alkalmazásokban a kérések nyomkövetésére és elemzésére szolgáló szolgáltatás. Engedélyezheti az **Aktív Nyomkövetést (Active Tracing)** a Lambda konfigurációjában a függvényhívások és a további AWS-szolgáltatások felé irányuló hívások nyomon követéséhez. Ehhez hozzá kell adni az `AWSXRayDaemonWriteAccess` házirendet a függvény végrehajtási szerepköréhez.

## 9.5: Lambda Teljesítmény
- **RAM:** A függvény memóriáját 128 MB-tól 10 GB-ig konfigurálhatja. **A RAM növelése arányosan növeli a CPU és a hálózati teljesítményt is.** Ez az elsődleges módja egy CPU-igényes függvény teljesítményének javítására.
- **Időkorlát (Timeout):** A maximális idő (legfeljebb 15 perc), ameddig egy függvény futhat, mielőtt leállítanák.
- **Végrehajtási Környezet (Execution Context):** Egy ideiglenes futtatókörnyezet, amelyet a Lambda inicializál a függvény számára. Ez a környezet, beleértve az adatbázis-kapcsolatokat és az SDK klienseket, amelyeket a fő handler függvényen *kívül* inicializáltak, újra felhasználható a későbbi meghívásokhoz. Ez egy kritikus teljesítményoptimalizálás, hogy elkerüljük az erőforrások minden egyes hívásnál történő újra-inicializálását.
- **/tmp Terület:** A Lambda egy ideiglenes, 512 MB-os fájlrendszer-könyvtárat (`/tmp`) biztosít, amely „piszkozatként” használható. Ez a terület megmarad a végrehajtási környezet élettartama alatt, lehetővé téve az adatok gyorsítótárazását a meghívások között.

## 9.6: Lambda Párhuzamosság (Concurrency)
- **Párhuzamosság:** Azon kérések száma, amelyeket a függvény egy adott időpontban kiszolgál. Alapértelmezetten az AWS-fiókjának van egy teljes párhuzamossági korlátja (gyakran 1000), amely megoszlik az adott régió összes függvénye között.
- **Korlátozás (Throttling):** Ha a meghívások száma meghaladja a párhuzamossági korlátot, a kérések korlátozva lesznek.
  - **Szinkron Meghívások (pl. API Gateway-ről):** `429`-es throttling hibát adnak vissza a kliensnek.
  - **Aszinkron Meghívások (pl. S3-ról):** A Lambda automatikusan újrapróbálkozik.
- **Foglalt Párhuzamosság (Reserved Concurrency):** Beállíthat egy specifikus párhuzamossági korlátot egy adott függvényre. Ez garantálja, hogy a függvény mindig képes lesz fel-skálázódni erre a szintre, de egyben korlátozza is a maximális párhuzamosságát, megakadályozva, hogy feleméssze a teljes fiókszintű keretet és más függvényeket korlátozzon.
- **Előre Kiépített Párhuzamosság (Provisioned Concurrency):** Egy funkció, amely lehetővé teszi, hogy meghatározott számú végrehajtási környezetet inicializálva és azonnali válaszra készen tartson. Ezzel kiküszöbölhetők a **hidegindítások (cold starts)** (az új végrehajtási környezet inicializálásával járó késleltetés) a késleltetésre érzékeny alkalmazások esetében. Ennek a funkciónak költsége van.
