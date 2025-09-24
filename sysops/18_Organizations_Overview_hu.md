# 18. fejezet: AWS Fiókkezelés

## AWS Health Dashboard
-   **Szolgáltatási előzmények (Service History):** Az összes AWS szolgáltatás általános állapotát mutatja régiók szerint. Történelmi adatokat és RSS-hírcsatornát biztosít.
-   **Fiók állapot-irányítópult (Account Health Dashboard, korábban Personal Health Dashboard - PHD):** Személyre szabott riasztásokat és helyreállítási útmutatót nyújt az AWS-fiókjait és erőforrásait közvetlenül érintő eseményekhez. Releváns és időben érkező információkat, valamint értesítéseket biztosít a tervezett karbantartásokról. Összegyűjti az adatokat a teljes AWS szervezet számára. A konzol harang ikonján keresztül érhető el. Közvetlenül Önt érintő leállásokat mutat, és eseménynaplót biztosít a múltbeli eseményekhez.
-   **Eseményértesítések:** Az EventBridge segítségével reagálhat az AWS Health eseményeire. Példa: e-mail értesítések fogadása EC2-példányfrissítésekről, vagy automatikusan törölheti a felfedett IAM-kulcsokat Lambda segítségével, vagy újraindíthatja a leállításra ütemezett EC2-példányokat.

## AWS Organizations
-   **Globális szolgáltatás:** Több AWS-fiók kezelését teszi lehetővé.
-   **Struktúra:**
    -   **Kezelőfiók (Management Account):** A szervezet fő fiókja.
    -   **Tagfiókok (Member Accounts):** Egyéb fiókok, amelyek csatlakoznak a szervezethez, vagy azon belül jönnek létre. Csak egy szervezet tagjai lehetnek.
    -   **Szervezeti egységek (Organizational Units - OUs):** Fiókok hierarchikus csoportosítása (pl. üzleti egység, környezet vagy projekt szerint).
-   **Előnyök:**
    -   **Konszolidált számlázás:** Egységes fizetési mód az összes fiókhoz.
    -   **Árazási előnyök:** Az összesített használat kedvezményekhez vezet (pl. EC2, S3 esetén).
    -   **Megosztott fenntartott példányok (RIs) és megtakarítási tervek (Savings Plans):** A kedvezmények a teljes szervezetre érvényesek. Fiókonként engedélyezhető/letiltható.
    -   **Automatizált fióklétrehozás:** API az új fiókok egyszerű létrehozásához.
    -   **Fokozott biztonság:** Jobb elkülönítés, mint több VPC egy fiókban.
    -   **Központosított naplózás:** Engedélyezze a CloudTrail/CloudWatch naplókat az összes fiókhoz egy központi S3-gyűjtőbe/naplózási fiókba.
    -   **Fiókközi szerepkörök:** Szerepkörök létrehozása adminisztrációs célokra.
    -   **Címkézési szabványok:** Címkézési szabványok érvényesítése a számlázás és az erőforrás-kategorizálás érdekében.
    -   **`aws:PrincipalOrgID` feltétel:** Használható IAM-szabályzatokban az IAM-principálok hozzáférésének biztosítására a szervezet összes fiókjából.
-   **Szolgáltatásvezérlési szabályzatok (Service Control Policies - SCPs):**
    -   IAM-szabályzatok, amelyeket OU-kra vagy fiókokra alkalmaznak, hogy korlátozzák a felhasználók és szerepkörök tevékenységét.
    -   NEM vonatkoznak a kezelőfiókra (mindig teljes adminisztrációs jogosultsággal rendelkezik).
    -   Az OU gyökerétől a fiókig vezető úton mindenhol explicit `Allow` (engedélyezés) szükséges egy művelet engedélyezéséhez. A `Deny` (tiltás) utasítások felülírják az `Allow` utasításokat.
    -   Példák: Adott szolgáltatások blokkolása (pl. DynamoDB, S3, EC2), vagy csak adott szolgáltatások engedélyezése.
-   **Címkeszabályzatok (Tag Policies):** Címkézési szabványok érvényesítése a fiókok között auditálás, kategorizálás és költségallokáció céljából. A nem megfelelő címkéket a CloudWatch Events segítségével lehet figyelni.

## AWS Control Tower
-   **Egyszerű beállítás és irányítás:** Egyszerűsíti a biztonságos, megfelelő, több fiókos AWS környezet beállítását és irányítását a legjobb gyakorlatok alapján.
-   **Automatizálja:** A környezet beállítását, a folyamatos szabályzatkezelést védőkorlátok (guardrails) segítségével, a szabályzatsértések észlelését és orvoslását.
-   **Interaktív irányítópult:** Figyeli a megfelelőséget.
-   **Organizations-re épül:** Automatikusan beállítja az AWS Organizations-t és implementálja az SCP-ket a védőkorlátokhoz.
-   **Landing Zone:** Létrehoz egy több fiókos környezetet egy Home Region-nel, Security OU-val (naplóarchívum és biztonsági audit fiókokhoz), és egy Sandbox OU-val.
-   **Védőkorlátok (Guardrails):** Megelőző (szabályzatok érvényesítése, pl. naplóarchívumok törlésének, naplók nyilvános olvasási hozzáférésének, CloudTrail változtatásoknak tiltása) és Észlelő (konfigurációs hibák észlelése).
-   **Account Factory:** Új fiókok Control Tower-be való felvételére szolgál.
-   **IAM Identity Center (korábban AWS SSO):** Kezeli a felhasználói hozzáférést a Control Tower összes fiókjához. Felhasználói portált biztosít az audit, naplóarchívum és egyéb fiókokhoz való egyszeri bejelentkezéshez.

## AWS Számlázási és Költségkezelési Eszközök

### Számlázási riasztások (Billing Alarms)
-   **Régió:** A számlázási adatok az `us-east-1` (Észak-Virginia) régióban tárolódnak, de a világméretű költségeket reprezentálják.
-   **Beállítás:** Engedélyezze a számlázási riasztásokat a számlázási preferenciákban.
-   **CloudWatch:** Hozzon létre riasztásokat a CloudWatch-ban (csak az `us-east-1` régióban látható).
-   **Metrikák:** Figyelje a teljes becsült költségeket vagy az adott szolgáltatások költségeit (pl. EC2, S3).
-   **Értesítések:** SNS-témákat indíthat riasztásokhoz (pl. ha a költség meghalad egy küszöböt).

### Költségfelfedező (Cost Explorer)
-   **Vizualizáció és kezelés:** Az AWS költségeinek és használatának vizualizálása, megértése és kezelése az idő múlásával.
-   **Egyéni jelentések:** Költség- és használati adatok elemzése magas szinten (összköltség) vagy részletes szinten (havi, óránkénti, erőforrás szintű).
-   **Költségmegtakarítás:** Segít kiválasztani az optimális megtakarítási terveket (Savings Plans).
-   **Előrejelzés:** Akár 12 hónapra előre jelzi a használatot a korábbi használat alapján a költségtervezéshez.
-   **Részletes bontás:** Óránkénti és erőforrás-szintű költséginformációkat biztosít.

### AWS Költségkeretek (Budgets)
-   **Részletes költségkeret-kezelés:** Költségkeretek létrehozása tényleges vagy becsült költségek alapján, riasztásokkal a küszöbértékek túllépése esetén. Részletesebb és rugalmasabb, mint a számlázási riasztások.
-   **Költségkeret-típusok:** Költség, Használat, Foglalás, Megtakarítási terv.
-   **Foglaláskövetés:** A fenntartott példányok (EC2, ElastiCache, RDS, Redshift) kihasználtságának nyomon követése.
-   **Értesítések:** Költségkeretenként legfeljebb öt értesítés. Szűrhető szolgáltatás, kapcsolt fiók, címke stb. szerint.
-   **Műveletek:** Automatikusan alkalmazhat IAM-szabályzatokat, szolgáltatásvezérlési szabályzatokat, vagy leállíthat EC2/RDS-példányokat, ha a költségkeret küszöbértékei teljesülnek. IAM-szerepkör szükséges a költségkeretek számára a műveletek végrehajtásához.
-   **Költség:** Az első két költségkeret ingyenes, utána napi 0,02 dollár költségkeretenként.

### Költségallokációs címkék és Költség- és Használati Jelentések (CUR)
-   **Költségallokációs címkék (Cost Allocation Tags):** Az AWS költségeinek részletes nyomon követése.
    -   **AWS által generált címkék:** Automatikusan alkalmazódnak (pl. `aws:createdBy`). `aws:` előtaggal kezdődnek.
    -   **Felhasználó által definiált címkék:** Felhasználók által definiáltak (pl. `user:Environment`). `user:` előtaggal kezdődnek.
    -   **Aktiválás:** A címkéket aktiválni kell a számlázási konzolban, hogy megjelenjenek a költségjelentésekben.
-   **Költség- és Használati Jelentések (Cost and Usage Reports - CUR):**
    -   Az AWS költség- és használati adatainak legátfogóbb gyűjteménye.
    -   Tartalmazza a szolgáltatásokra, árazásra és foglalásokra vonatkozó metaadatokat.
    -   Felsorolja az egyes szolgáltatások használatát óránkénti/napi tételsorokkal.
    -   Tartalmazza az aktivált költségallokációs címkéket.
    -   **Exportálás:** Konfigurálható napi exportálásra Amazon S3-ba.
    -   **Elemzés:** Az adatok elemzhetők Athena, Redshift vagy QuickSight segítségével.

## AWS Compute Optimizer
-   **Költségcsökkentés és teljesítményjavítás:** Optimális AWS erőforrásokat ajánl a munkaterhelésekhez.
-   **Gépi tanulás:** Elemzi az erőforrás-konfigurációt és a CloudWatch metrikákat a túl-/alul-ellátott erőforrások azonosítására.
-   **Támogatott erőforrások:** EC2-példányok, Auto Scaling csoportok, EBS-kötetek, Lambda-függvények.
-   **Előnyök:** Akár 25%-kal csökkentheti a költségeket.
-   **Exportálás:** Az ajánlások exportálhatók Amazon S3-ba.