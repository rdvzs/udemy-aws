# 14. fejezet: Haladó tárolási szekció

## AWS Snow család
Az AWS Snow család rendkívül biztonságos, hordozható eszközökből áll, amelyek adatgyűjtésre, -feldolgozásra és -migrációra szolgálnak a peremhálózaton, valamint adatok AWS-be és onnan történő átvitelére.
-   **Snowball Edge Storage Optimized:** 210 TB tárhely, elsősorban adatmigrációra.
-   **Snowball Edge Compute Optimized:** 28 TB tárhely, nagyobb számítási teljesítmény, peremhálózati számítási feladatokhoz (pl. EC2 példányok vagy Lambda függvények futtatása korlátozott internet-hozzáféréssel rendelkező helyeken).
-   **Felhasználási esetek:**
    -   **Adatmigráció:** Nagy adathalmazok (petabájt) vagy lassú, költséges, illetve instabil hálózati átvitel esetén (pl. több mint egy hétig tart). Fizikai eszközt kap, feltölti az adatokat, majd visszaküldi az AWS-nek az S3-ba történő importáláshoz.
    -   **Peremhálózati számítás:** Az adatok feldolgozása ott, ahol keletkeznek (pl. távoli helyszínek, járművek), mielőtt elküldik az AWS-nek.

## Amazon FSx
Az Amazon FSx egy teljesen menedzselt szolgáltatás, amely lehetővé teszi népszerű, nagy teljesítményű harmadik féltől származó fájlrendszerek indítását és futtatását az AWS-en.
-   **FSx for Windows File Server:**
    -   Teljesen menedzselt Windows fájlszerver.
    -   Támogatja az SMB protokollt és a Windows NTFS-t.
    -   Integrálódik a Microsoft Active Directory-val a felhasználói hitelesítéshez.
    -   Linux EC2 példányokra is csatlakoztatható.
    -   Skálázható több tíz GB/s átviteli sebességre, millió IOPS-ra, több száz PB adatra.
    -   Tárolási lehetőségek: SSD (alacsony késleltetés) vagy HDD (költséghatékony).
    -   Több-AZ-s telepítés a magas rendelkezésre állás érdekében (szinkron replikáció, automatikus átállás).
    -   Napi biztonsági mentések S3-ba.
-   **FSx for Lustre:**
    -   Nagy teljesítményű fájlrendszer nagyméretű számításokhoz (HPC, gépi tanulás, videófeldolgozás, pénzügyi modellezés).
    -   Skálázható több száz GB/s átviteli sebességre, millió IOPS-ra, milliszekundum alatti késleltetésre.
    -   Tárolási lehetőségek: SSD (alacsony késleltetés, IOPS-intenzív) vagy HDD (átviteli sebesség-intenzív).
    -   Zökkenőmentes integráció az S3-mal (S3 olvasása fájlrendszerként, számítási eredmények visszaírása S3-ba).
    -   Telepítési lehetőségek:
        -   **Scratch fájlrendszer:** Ideiglenes tárolás, nincs adatreplikáció, nagy burst teljesítmény (6x tartós), adatvesztés szerverhiba esetén. Rövid távú feldolgozáshoz.
        -   **Persistent fájlrendszer:** Hosszú távú tárolás, adatok replikálása ugyanazon AZ-n belül, átlátszó csere szerverhiba esetén. Érzékeny adatokhoz.
    -   VPN/Direct Connect-en keresztül is használható helyszíni rendszerekről.
-   **FSx for NetApp ONTAP:**
    -   Menedzselt NetApp ONTAP fájlrendszer.
    -   Kompatibilis az NFS, SMB és iSCSI protokollokkal.
    -   Széles körű kompatibilitás (Linux, Windows, macOS, VMware Cloud, WorkSpaces, AppStream, EC2, ECS, EKS).
    -   Automatikus skálázású tárhely, replikáció, pillanatfelvételek, alacsony költség, adattömörítés, adatduplikáció-mentesítés.
    -   Pont-időbeli azonnali klónozás teszteléshez.
-   **FSx for OpenZFS:**
    -   Menedzselt OpenZFS fájlrendszer.
    -   Kompatibilis az NFS protokollal.
    -   Nagy teljesítmény (1M IOPS, <0,5 ms késleltetés).
    -   Támogatja a pillanatfelvételeket, tömörítést, alacsony költséget (nincs adatduplikáció-mentesítés).
    -   Pont-időbeli azonnali klónozás.

## AWS Storage Gateway
Hibrid felhőalapú tárolási szolgáltatás, amely összeköti a helyszíni adatokat az AWS felhőalapú tárolóval. Virtuális gépként (VM) telepítve a helyszínen.
-   **S3 File Gateway:**
    -   S3 tárolókat NFS vagy SMB fájlmegosztásként tesz elérhetővé a helyszínen.
    -   A legutóbb használt adatok helyben gyorsítótárazva vannak.
    -   Támogatja az S3 Standard, S3 Standard-IA, S3 One Zone-IA, S3 Intelligent-Tiering tárolási osztályokat (közvetlenül nem a Glacier-t, de életciklus-házirendekkel átvihető).
    -   Integrálódik az Active Directory-val az SMB-hez.
    -   POSIX-kompatibilis (metaadatok, engedélyek, időbélyegek az S3 objektum metaadataiban tárolva).
-   **Volume Gateway:**
    -   Blokktároló köteteket biztosít a helyszíni alkalmazásoknak iSCSI protokollon keresztül.
    -   A köteteket EBS pillanatfelvételek támasztják alá az S3-ban.
    -   **Gyorsítótárazott kötetek:** Elsődleges adatok az S3-ban, gyakran hozzáférhető adatok helyben gyorsítótárazva az alacsony késleltetés érdekében.
    -   **Tárolt kötetek:** Teljes adathalmaz helyben tárolva, aszinkron módon biztonsági mentve az S3-ba.
    -   Monitorozás: `CacheHitPercent` (magas érték jó), `CachePercentUsed` (kerüljük a túl magas értéket).
-   **Tape Gateway:**
    -   Fizikai szalagos könyvtárakat helyettesít virtuális szalagokkal az AWS-ben.
    -   iSCSI-VTL protokollt használ.
    -   Virtuális szalagok az S3-ban tárolva, és archiválhatók Glacier/Glacier Deep Archive-ba.
-   **Aktiválás:** A Gateway VM-et aktiválni kell, vagy CLI-n, vagy webes kérésen keresztül (80-as port). Az időszinkronizálás (NTP) kulcsfontosságú az aktiváláshoz.
-   **Újraindítás:** A File Gateway egyszerűen újraindítható. A Volume/Tape Gateway esetén le kell állítani a szolgáltatást, újra kell indítani a VM-et, majd újra el kell indítani a szolgáltatást.

## AWS DataSync
Adatátviteli szolgáltatás az adatok mozgatásának automatizálására a helyszíni tárolók (NFS, SMB, saját kezűleg kezelt objektumtároló, HDFS) és az AWS tárolási szolgáltatások (S3, EFS, FSx for Windows File Server) között.
-   DataSync ügynök (VM) telepítése a helyszínen.
-   Feladat létrehozása, amely meghatározza a forrást, a célt és az átviteli opciókat (pl. adatintegritás ellenőrzése, metaadatok megőrzése, sávszélesség korlátozása, ütemezés).
-   Gyors (akár 10-szer gyorsabb, mint a nyílt forráskódú eszközök), biztonságos (titkosítás átvitel közben), megbízható (végpontok közötti adatintegritás) és költséghatékony (inkrementális átvitelek).

## CloudEndure Migration (most AWS Application Migration Service - MGN)
Szolgáltatás a lift-and-shift (rehosting) felhőmigrációk egyszerűsítésére és felgyorsítására.
-   Könnyűsúlyú ügynök telepítése a forrásszerverekre (fizikai, virtuális vagy felhőalapú).
-   Adatok replikálása blokkszinten egy alacsony költségű, alacsony erőforrás-igényű EC2 staging területre az AWS-ben.
-   A gépek automatikus konvertálása az AWS-en natívan futtatásra az átállás során, minimalizálva az állásidőt (másodpercek).
-   Folyamatos adatreplikációt biztosít a megbízhatóság érdekében.

## AWS Backup
Teljesen menedzselt szolgáltatás, amely központosítja és automatizálja az adatvédelmet a különböző AWS szolgáltatások (EBS, RDS, DynamoDB, EFS, Storage Gateway kötetek, EC2 példányok) között.
-   Biztonsági mentési tervek létrehozása, amelyek meghatározzák a gyakoriságot, az időablakot, a megőrzési időszakot és a biztonsági mentési tárolót (logikai tároló a biztonsági mentésekhez).
-   Erőforrások hozzárendelése a biztonsági mentési tervekhez címkék, erőforrástípus vagy azonosító alapján.
-   Automatikus inkrementális biztonsági mentések létrehozása.
-   Egyszerű, minimális működési többletköltség, költséghatékony és biztonságos (titkosítás nyugalmi állapotban és átvitel közben).

## AWS Elastic Disaster Recovery (DRS)
Az AWS elsődleges katasztrófa-helyreállítási szolgáltatása, amely minimalizálja az állásidőt és az adatvesztést a fizikai, virtuális és felhőalapú alkalmazások AWS-be történő gyors és megbízható helyreállításával. A CloudEndure Disaster Recovery utódja.
-   Könnyűsúlyú ügynök telepítése a forrásszerverekre.
-   Adatok replikálása blokkszinten egy alacsony költségű, alacsony erőforrás-igényű EC2 staging területre az AWS-ben.
-   A gépek automatikus konvertálása az AWS-en natívan futtatásra katasztrófa esetén, minimalizálva az állásidőt (percek).
-   Folyamatos adatreplikációt biztosít a megbízhatóság érdekében.