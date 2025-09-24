# 19. fejezet: Katasztrófa-helyreállítás

## AWS DataSync
Az AWS DataSync egy adatátviteli szolgáltatás, amely automatizálja az adatok mozgatását a helyszíni tárolók (NFS, SMB, HDFS, saját kezűleg kezelt objektumtároló) és az AWS tárolási szolgáltatások (S3, EFS, FSx) között. Képes az adatok szinkronizálására különböző AWS tárolási szolgáltatások között is.
-   **Ügynök:** DataSync ügynök (VM) telepítése szükséges a helyszínen vagy egy másik felhőben. Az AWS Snowcone eszközök előre telepített DataSync ügynökkel rendelkeznek.
-   **Szinkronizálás:** Ütemezett feladatok (óránként, naponta, hetente), nem folyamatos.
-   **Metaadatok megőrzése:** Megőrzi a fájl engedélyeit és metaadatait (POSIX-kompatibilis NFS esetén, SMB engedélyek). Ez kulcsfontosságú különbség a vizsgák szempontjából.
-   **Sávszélesség:** Akár 10 Gbps is kihasználható ügynökönként, opcionális sávszélesség-korlátozással.
-   **Irány:** Támogatja a kétirányú szinkronizálást (helyszínről AWS-be, és AWS-ből helyszínre).

## AWS Backup
Az AWS Backup egy teljesen menedzselt szolgáltatás, amely központosítja és automatizálja a biztonsági mentéseket különböző AWS szolgáltatások (EC2, EBS, S3, RDS, Aurora, DynamoDB, DocumentDB, Neptune, EFS, FSx, Storage Gateway kötetek) között.
-   **Biztonsági mentési tervek:** Meghatározzák a biztonsági mentés gyakoriságát (pl. naponta, havonta), a biztonsági mentési időablakot, a hideg tárolóba való áthelyezést (pl. Glacier), és a megőrzési időszakot.
-   **Erőforrás-hozzárendelés:** Erőforrások hozzárendelése a biztonsági mentési tervekhez címkék vagy specifikus erőforrástípusok alapján.
-   **Régiók közötti/fiókok közötti biztonsági mentések:** Támogatja a biztonsági mentések más régiókba vagy fiókokba történő másolását katasztrófa-helyreállítás céljából.
-   **Pont-időbeli helyreállítás (PITR):** Támogatott bizonyos szolgáltatásokhoz (pl. Aurora).
-   **Vault Lock:** WORM (Write Once Read Many) házirendet kényszerít ki a biztonsági mentési tárolókon, megakadályozva a biztonsági mentések véletlen vagy rosszindulatú törlését/módosítását, még a gyökérfelhasználó által is.

## AWS Elastic Disaster Recovery (DRS)
Az AWS Elastic Disaster Recovery (DRS) az AWS elsődleges katasztrófa-helyreállítási szolgáltatása, amely minimalizálja az állásidőt és az adatvesztést a fizikai, virtuális és felhőalapú alkalmazások AWS-be történő gyors és megbízható helyreállításával. A CloudEndure Disaster Recovery utódja.
-   **Ügynök:** Könnyűsúlyú ügynök telepítése a forrásszerverekre.
-   **Replikáció:** Blokkszinten replikálja az adatokat egy alacsony költségű, alacsony erőforrás-igényű EC2 staging területre az AWS-ben.
-   **Helyreállítás:** Katasztrófa esetén automatikusan konvertálja a gépeket, hogy natívan fussanak az AWS-en, minimalizálva az állásidőt (percek).
-   **Folyamatos adatvédelem:** Folyamatos adatreplikációt biztosít a megbízhatóság érdekében.

## Katasztrófa-helyreállítási stratégiák
Stratégiák az IT rendszerek katasztrófáira való felkészülésre és azokból való helyreállításra, az állásidő és az adatvesztés minimalizálására összpontosítva.
-   **Helyreállítási idő cél (RTO):** A szolgáltatás megszakadása és a szolgáltatás helyreállítása közötti maximálisan elfogadható késleltetés.
-   **Helyreállítási pont cél (RPO):** Az adatok maximálisan elfogadható mennyisége (időben mérve).
-   **Négy fő stratégia:**
    -   **Biztonsági mentés és visszaállítás:** A legolcsóbb, legkönnyebben megvalósítható. Adatok biztonsági mentése S3-ba, visszaállítás katasztrófa esetén. Magasabb RTO/RPO.
    -   **Pilot Light:** Az alkalmazás minimális verziója fut a katasztrófa-helyreállítási régióban. Gyorsabb RTO/RPO, mint a biztonsági mentés/visszaállítás.
    -   **Warm Standby:** Az alkalmazás skálázott verziója fut a katasztrófa-helyreállítási régióban. Gyorsabb RTO/RPO, mint a pilot light.
    -   **Multi-Site Active/Active:** A legdrágább, legösszetettebb. Az alkalmazás egyidejűleg több régióban fut. Legalacsonyabb RTO/RPO.

## Felhőmigrációs stratégiák (a 7 R)
Stratégiák az alkalmazások és adatok helyszínről a felhőbe való áthelyezésére.
-   **Rehost (Lift and Shift):** Az alkalmazások változatlan áthelyezése a felhőbe.
-   **Replatform (Lift, Tinker, and Shift):** Kisebb módosítások az alkalmazásokon a felhőre való optimalizálás érdekében.
-   **Refactor (Re-architect):** Az alkalmazások újraarchitektúrázása a felhőnatív funkciók teljes kihasználása érdekében.
-   **Repurchase (Drop and Shop):** A meglévő alkalmazások lecserélése SaaS megoldással a felhőben.
-   **Retire:** A már nem szükséges alkalmazások leszerelése.
-   **Retain:** Az alkalmazások helyszínen tartása szabályozási követelmények vagy egyéb okok miatt.
-   **Relocate:** Az alkalmazások áthelyezése a felhőbe új hardver vásárlása vagy kód újraírása nélkül (pl. VMware Cloud on AWS).