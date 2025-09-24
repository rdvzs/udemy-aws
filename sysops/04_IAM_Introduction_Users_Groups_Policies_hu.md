
# 4. Fejezet: AMI (Amazon Machine Image)

Ez a fejezet az Amazon Machine Image-ekre (AMI) összpontosít, bemutatva azok létrehozását, kezelését és éles környezetben való használatát. Részletezi továbbá az EC2 Image Builder automatizálási szolgáltatást, valamint a példányok migrálására és az AMI-k megosztására vonatkozó stratégiákat.

## 4.1: AMI Áttekintés
- **AMI (Amazon Machine Image):** Egy sablon, amely tartalmazza a példány indításához szükséges szoftverkonfigurációt (operációs rendszer, alkalmazásszerver és alkalmazások).
- **Előnyök:** Egyéni AMI használata gyorsabb indítási és konfigurációs időt eredményez, mivel az összes szükséges szoftver előre telepítve van.
- **AMI Források:**
  - **Nyilvános AMI-k:** Az AWS által biztosított és karbantartott (pl. Amazon Linux 2).
  - **Saját AMI-k:** Ön által létrehozott és karbantartott egyéni képek.
  - **AWS Marketplace AMI-k:** Harmadik féltől származó, előre konfigurált képek, amelyek kereskedelmi szoftvereket is tartalmazhatnak.
- **Létrehozási Folyamat:** Jellemzően egy nyilvános AMI-ból indít egy példányt, testreszabja (szoftvereket telepít, frissítéseket alkalmaz), majd ebből a példányból hoz létre egy új, egyéni AMI-t. Ez a folyamat a háttérben EBS pillanatképeket (Snapshot) is létrehoz.

## 4.2: AMI Gyakorlat
- **Folyamat:**
  1.  Indítson egy szabványos EC2 példányt (pl. Amazon Linux 2).
  2.  Egy User Data szkript segítségével telepítsen és indítson el egy webszervert (pl. Apache HTTPD), de a végső `index.html` fájl létrehozása nélkül. Ez a közös szoftverek előtelepítését szimulálja.
  3.  Miután a példány fut és a szoftver települt, hozzon létre egy képet belőle az AWS Konzol segítségével (`Actions` > `Image and templates` > `Create image`).
  4.  Indítson egy új példányt *az Ön egyéni AMI-jából*. Ez az új példány sokkal gyorsabban fog elindulni a már előre telepített webszerverrel.
  5.  Adjon meg egy új User Data szkriptet a második példánynak a végső konfigurációk elvégzéséhez, például az `index.html` fájl létrehozásához.
- **Eredmény:** Ez bemutatja, hogyan gyorsíthatják fel jelentősen az AMI-k az új, előre konfigurált példányok telepítését.

## 4.3: AMI Újraindítás Nélküli (No-Reboot) Opció
- **Alapértelmezett Viselkedés:** Alapértelmezetten, amikor egy futó példányból AMI-t hoz létre, az AWS először **újraindítja** a példányt a fájlrendszer integritásának biztosítása érdekében, mielőtt pillanatképeket készítene.
- **No-Reboot Opció:** Választhatja, hogy az AMI-t a példány újraindítása *nélkül* hozza létre. Ez gyorsabb és kevésbé zavaró, de kockázattal jár.
  - **Kockázat:** A létrehozott kép fájlrendszerének integritása nem garantált, mivel a memóriában lévő adatok és az OS pufferei esetleg nem íródnak ki a lemezre.
- **AWS Backup:** Az AWS Backup szolgáltatás alapértelmezetten a `No-Reboot` opciót használja, amikor AMI-kat hoz létre egy mentési terv részeként. Ha a fájlrendszer konzisztenciája kritikus, egyéni megoldásra lehet szükség (pl. egy ütemezett Lambda függvény, amely újraindítja a példányt az AMI készítése előtt).

## 4.4: EC2 Példány Migráció és Fiókok Közötti Megosztás
- **Példány Migrálása Másik AZ-be:** A szabványos módszer az, hogy létrehoz egy AMI-t a példányból, majd abból az AMI-ból indít egy új példányt a kívánt rendelkezésre állási zónában (Availability Zone).
- **Fiókok Közötti AMI Megosztás:** Megoszthatja egyéni AMI-jait más AWS-fiókokkal.
  - **Titkosítatlan AMI-k:** Könnyen megoszthatók más fiókokkal, vagy akár nyilvánossá is tehetők.
  - **Titkosított AMI-k:** Ha az AMI gyökérkötete egy **ügyfél által kezelt KMS kulccsal (CMK)** van titkosítva, akkor a KMS kulcsot is meg kell osztania a célfiókkal, hogy az el tudja indítani a példányokat.
- **Fiókok Közötti AMI Másolás:** Egy másik fiókban lévő felhasználó *lemásolhat* egy vele megosztott AMI-t. Ez egy új, független AMI-t hoz létre a saját fiókjában, amelynek ő lesz a tulajdonosa. Ez hasznos a függőségek izolálására és a titkosítás saját kulcsokkal való kezelésére.

## 4.5: EC2 Image Builder
- **EC2 Image Builder:** Teljesen menedzselt AWS szolgáltatás, amely automatizálja a Linux és Windows AMI-k létrehozását, kezelését, validálását és tesztelését.
- **Munkafolyamat:**
  1.  **Építés (Build):** Elindít egy ideiglenes EC2 példányt, hogy telepítse és konfigurálja a szoftvereket egy meghatározott „recept” és komponensek alapján.
  2.  **Tesztelés (Test):** Elindít egy második példányt az újonnan létrehozott AMI-ból, hogy futtassa az Ön által definiált teszteket, biztosítva a kép biztonságosságát és működőképességét.
  3.  **Terjesztés (Distribute):** Elosztja a validált AMI-t a megadott AWS-régiókba.
- **Előnyök:**
  - **Automatizálás:** Leegyszerűsíti az „arany AMI-k” (golden AMI) létrehozását.
  - **Ütemezés:** Futtatható ütemezés szerint (pl. hetente) vagy események által kiváltva, hogy a képek naprakészek legyenek a legújabb frissítésekkel.
  - **Költség:** Maga a szolgáltatás ingyenes; csak az építési és tesztelési folyamat során használt alapvető erőforrásokért kell fizetni (pl. EC2 példányok, S3 tárolás).

## 4.6: AMI-k Használata Éles Környezetben
- **Jóváhagyott AMI-k Kikényszerítése:** IAM házirendekkel korlátozhatja a felhasználókat, hogy csak meghatározott, előre jóváhagyott AMI-kból indíthassanak EC2 példányokat. Ez általában a következőképpen történik:
  1.  A jóváhagyott AMI-k megcímkézése egyedi címkével (pl. `Environment=Prod`).
  2.  Olyan IAM házirend létrehozása, amely egy `Condition` feltétellel ellenőrzi ezt a címkét az `ec2:RunInstances` művelet során.
- **Auditálás AWS Config-gal:** Az AWS Config szabályokkal folyamatosan monitorozhatja a környezetét, és azonosíthatja azokat a futó EC2 példányokat, amelyeket nem jóváhagyott AMI-kból indítottak, nem megfelelőnek (non-compliant) jelölve őket.

## 4.7: AMI Szekció Tisztítása
- A költségek elkerülése érdekében a gyakorlati laborok befejezése után fontos a felhasznált erőforrások eltávolítása:
  1.  **Szüntesse meg (Terminate)** az összes futó EC2 példányt.
  2.  **Törölje a regisztrációt (Deregister)** minden létrehozott egyéni AMI-ról.
  3.  **Törölje (Delete)** a regisztrációból törölt AMI-khoz tartozó EBS pillanatképeket.
