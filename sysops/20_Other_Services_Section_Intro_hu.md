# 20. fejezet: Biztonság és megfelelőség SysOps számára

## Megosztott felelősségi modell
Az AWS felelőssége "a felhő biztonsága" (infrastruktúra, menedzselt szolgáltatások). Az Ön felelőssége "a felhőben lévő biztonság" (OS javítás, tűzfal konfiguráció, IAM, alkalmazásadatok titkosítása). Néhány vezérlő megosztott (pl. patch menedzsment).

## DDoS védelem (Shield, WAF, CloudFront, Route 53)
-   **DDoS támadás:** Elosztott szolgáltatásmegtagadási támadás, amely túlterheli a szervert kérésekkel.
-   **AWS Shield Standard:** Ingyenes, minden ügyfél számára engedélyezett, védelmet nyújt a gyakori DDoS támadások ellen (SYN/UDP áradatok, reflexiós támadások).
-   **AWS Shield Advanced:** Opcionális (3000 dollár/hó/szervezet), prémium DDoS védelem a kifinomult támadások ellen, 24/7 DDoS válaszcsapat, költségvédelem a támadások során felmerülő díjakra.
-   **AWS WAF (Web Application Firewall):** Védi a webes alkalmazásokat a gyakori webes kihasználásoktól (7. réteg). ALB-n, API Gateway-n, CloudFronton telepítve. Web ACL-eket használ IP-címek, fejlécek, törzs, karakterláncok, SQL injekció, cross-site scripting alapján. Támogatja a sebesség alapú szabályokat.
-   **CloudFront és Route 53:** DDoS védelmet biztosítanak a globális peremhálózati hálózat kihasználásával és a forgalom elosztásával.
-   **Skálázás:** Az Auto Scaling segíthet az alkalmazásoknak a nagyobb igények kezelésében a támadások során.

## Penetrációs tesztelés az AWS-en
Az ügyfelek általában engedélyezettek biztonsági felmérések és penetrációs tesztelés végrehajtására saját AWS infrastruktúrájukon bizonyos szolgáltatások (EC2 példányok, NAT Gateway-ek, ELB, RDS, CloudFront, Aurora, API Gateway, Lambda, Lightsail, Elastic Beanstalk) ellen előzetes jóváhagyás nélkül.
-   **Tiltott tevékenységek:** DNS zóna bejárás (Route 53), DDoS támadások (DoS/DDoS/Szimulált DoS/DDoS), port/protokoll/kérés áradat.
-   Egyéb tevékenységek esetén vegye fel a kapcsolatot az AWS biztonsági csapatával jóváhagyásért.

## Amazon Inspector
Automatizált biztonsági felmérési szolgáltatás EC2 példányok, ECR konténerképek és Lambda függvények számára.
-   **Funkcionalitás:** Elemzi a nem szándékos hálózati elérhetőséget és az ismert sebezhetőségeket (CVE-k) a futó operációs rendszerben és a csomagfüggőségekben. Folyamatos szkennelés.
-   **Jelentéskészítés:** Jelentéseket küld az AWS Security Hub-nak, és eseményeket küld az Amazon EventBridge-nek automatizálás céljából.
-   **Kockázati pontszám:** Kockázati pontszámot rendel a sebezhetőségekhez a prioritás meghatározásához.

## AWS Artifact
Egy portál, amely igény szerinti hozzáférést biztosít az AWS megfelelőségi jelentésekhez (pl. ISO, PCI, SOC) és megállapodásokhoz (pl. BAA, HIPAA). Belső auditok és megfelelőségi igények támogatására szolgál.

## AWS Certificate Manager (ACM)
TLS/SSL tanúsítványokat kezel az átvitel közbeni titkosításhoz.
-   **Nyilvános tanúsítványok:** Ingyenes, automatikusan megújul (ha ACM által generált és DNS által ellenőrzött). Nem exportálható.
-   **Privát tanúsítványok:** Belső használatra.
-   **Érvényesítés:** DNS érvényesítés (preferált automatizáláshoz a Route 53-mal) vagy e-mail érvényesítés.
-   **Integráció:** Integrálódik az ELB-vel (Classic, ALB, NLB), CloudFronttal, API Gateway-vel. Nem használható közvetlenül EC2 példányokkal.
-   **Megújítás:** Az ACM által generált tanúsítványok automatikusan megújulnak 60 nappal a lejárat előtt. Az importált tanúsítványok manuális megújítást igényelnek.
-   **Lejárati események:** Az ACM napi lejárati eseményeket küld az EventBridge-nek 45 nappal a lejárat előtt. A Config szabályok (pl. `ACM-certificate-expiration-check`) is ellenőrizhetik a lejáró tanúsítványokat.

## AWS Secrets Manager
Titkokat (pl. adatbázis hitelesítő adatok, API kulcsok) tárol és kezel.
-   **Főbb jellemzők:**
    -   **Automatikus rotáció:** Kényszeríti a titkok rotációját X naponta (pl. 30 naponta) Lambda függvények segítségével (AWS által biztosított RDS, Redshift, DocumentDB esetén; egyéni mások esetén).
    -   **Integráció:** Jól integrálódik az RDS, Redshift, DocumentDB szolgáltatásokkal.
    -   **Titkosítás:** A titkok KMS-sel titkosítva vannak (kötelező).
    -   **Több régiós titkok:** Titkok replikálása több AWS régió között katasztrófa-helyreállítás és több régiós alkalmazások céljából.
-   **Költség:** 0,40 dollár/titok/hó + 0,05 dollár/10 000 API hívás (első 30 nap ingyenes).

## AWS KMS (Key Management Service)
Titkosítási kulcsokat kezel.
-   **Kulcstípusok:**
    -   **Szimmetrikus:** Egyetlen kulcs titkosításhoz/visszafejtéshez. A legtöbb AWS szolgáltatás használja.
    -   **Aszimmetrikus:** Nyilvános kulcs (titkosítás), privát kulcs (visszafejtés). Titkosítási/visszafejtési vagy aláírási/ellenőrzési műveletekhez használják. A nyilvános kulcs letölthető.
-   **Kulcstulajdon:**
    -   **AWS tulajdonú kulcsok:** Ingyenes, AWS által kezelt (pl. SSE-S3).
    -   **AWS által kezelt kulcsok:** Ingyenes, AWS által kezelt, `aws/service-name` alias alapján felismerhető (pl. `aws/rds`).
    -   **Ügyfél által kezelt kulcsok (CMK):** Felhasználó által kezelt, 1 dollár/hó. Importálható.
-   **Kulcsházirendek:** Hozzáférés-vezérlés a KMS kulcsokhoz (hasonló az S3 tárolóházirendekhez). Szükséges a fiókok közötti hozzáféréshez.
-   **Kulcsrotáció:**
    -   **Automatikus:** AWS által kezelt kulcsok (évente). Ügyfél által kezelt szimmetrikus kulcsok (opcionális, konfigurálható 90-2560 nap, alapértelmezett 1 év).
    -   **Igény szerinti:** Ügyfél által kezelt szimmetrikus kulcsokhoz.
    -   **Manuális:** Importált KMS kulcsokhoz.
-   **Régiók közötti másolás:** Titkosított EBS pillanatfelvétel más régióba másolásához újra kell titkosítani egy KMS kulccsal a célrégióban.

## AWS Organizations
Több AWS fiók kezelése.
-   **Menedzsment fiók:** Központi fiók a számlázáshoz és a kezeléshez.
-   **Tagfiókok:** A szervezet többi fiókja.
-   **Konszolidált számlázás:** Egyetlen fizetési mód, összesített felhasználás az árkedvezményekhez, RI/Savings Plan megosztás.
-   **Szervezeti egységek (OU):** Fiókok hierarchikus csoportosítása kezelés és házirend alkalmazás céljából.
-   **Szolgáltatásvezérlési házirendek (SCP):** IAM házirendek, amelyeket OU-kra vagy fiókokra alkalmaznak az engedélyek korlátozására. Az OU/fiók összes felhasználójára/szerepkörére vonatkoznak (kivéve a menedzsment fiókot). Explicit engedélyezést igényelnek az útvonal mentén.

## AWS Control Tower
Egyszerűsíti a biztonságos, több fiókos AWS környezetek beállítását és irányítását a legjobb gyakorlatok alapján.
-   **Landing Zone:** Automatizálja a környezet beállítását (OU-kat, megosztott fiókokat hoz létre naplókhoz/auditokhoz).
-   **Guardrails:** Automatizálja a házirend-kezelést, észleli a jogsértéseket (megelőző és detektív).
-   **Integráció:** Az AWS Organizations tetején fut, SCP-ket implementál.

## AWS Service Catalog
Önkiszolgáló portál a felhasználók számára engedélyezett AWS termékek (CloudFormation sablonok) indítására.
-   **Termékek:** CloudFormation sablonok.
-   **Portfóliók:** Termékek gyűjteményei.
-   **Hozzáférési vezérlés:** Az IAM engedélyek határozzák meg, hogy ki férhet hozzá a portfóliókhoz.
-   **TagOptions:** A kiépített termékek címkéinek kezelése a megfelelő erőforrás-címkézés érdekében.

## AWS Compute Optimizer
Optimális AWS erőforrásokat (EC2, ASG, EBS, Lambda) ajánl a költségek csökkentésére és a teljesítmény javítására gépi tanulás segítségével.

## AWS Trusted Advisor
Magas szintű fiókfelmérést és tanácsokat nyújt a költségoptimalizálás, a teljesítmény, a biztonság, a hibatűrés, a szolgáltatási korlátok és a működési kiválóság terén. Néhány ellenőrzés üzleti/vállalati támogatási terveket igényel.

## AWS Cost Explorer
Vizualizálja, megérti és kezeli az AWS költségeit és felhasználását az idő múlásával.
-   **Jelentések:** Egyéni jelentések a költség- és felhasználási adatokról (havi, óránkénti, erőforrás szintű).
-   **Savings Plan ajánlások:** Segít kiválasztani az optimális Savings Plan-eket.
-   **Előrejelzés:** Akár 12 hónapra előrejelzi a felhasználást.

## AWS Budgets
Költség- és felhasználási riasztásokat hoz létre.
-   **Költségvetés típusok:** Költség, felhasználás, foglalás, megtakarítási terv.
-   **Riasztások:** Értesítéseket küld (e-mail, SNS, Chatbotok), ha a tényleges vagy előre jelzett költségek/felhasználás meghaladja a meghatározott küszöbértékeket.
-   **Műveletek:** Automatikus műveleteket indíthat (pl. IAM házirend csatolása, EC2/RDS példányok leállítása), ha a költségvetési küszöbértékeket túllépik.

## AWS Cost Allocation Tags
Felhasználó által definiált címkék (pl. `Environment:Production`) és AWS által generált címkék (pl. `aws:createdBy`), amelyeket az AWS költségeinek részletes nyomon követésére használnak a költség- és felhasználási jelentésekben.

## AWS Cost and Usage Reports
Átfogó jelentések az AWS költségeiről és felhasználásáról, beleértve a részletes metaadatokat és a költségallokációs címkéket. S3-ba kézbesítve.

## CloudWatch Metrics
Teljesítmény metrikákat biztosít az AWS erőforrások és alkalmazások számára.

## CloudWatch Logs
Központosított szolgáltatás az alkalmazásnaplók tárolására.

## CloudWatch Alarms
Riasztásokat vagy műveleteket vált ki metrikus küszöbértékek alapján.

## CloudWatch Dashboards
Kulcsfontosságú metrikák és riasztások vizualizálása.

## CloudWatch Synthetics Canary
Konfigurálható szkriptek API-k, URL-ek vagy weboldalak proaktív monitorozására.

## Amazon EventBridge
Eseményvezérelt architektúra szolgáltatás az események AWS szolgáltatások és az alkalmazások közötti útválasztására.

## AWS CloudTrail
API hívások és felhasználói tevékenység auditálása.

## AWS DataSync
Adatátviteli szolgáltatás az adatok helyszínről AWS tárolóba történő mozgatásának automatizálására.

## AWS Elastic Disaster Recovery (DRS)
Az AWS elsődleges katasztrófa-helyreállítási szolgáltatása.

## Katasztrófa-helyreállítási stratégiák
Stratégiák az IT rendszerek katasztrófáira való felkészülésre és azokból való helyreállításra.

## Felhőmigrációs stratégiák
Stratégiák az alkalmazások és adatok felhőbe való áthelyezésére.