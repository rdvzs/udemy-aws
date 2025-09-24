# 12. fejezet: Haladó Amazon S3 és Athena

## S3 életciklus-szabályok S3 Analytics-szel
Az objektumok különböző S3 tárolási osztályok (Standard, Standard-IA, Intelligent-Tiering, One-Zone IA, Glacier, Deep Archive) között mozgathatók az hozzáférési minták alapján. Az életciklus-szabályok automatizálják ezeket az átmeneteket, és beállíthatnak lejárati műveleteket is az objektumokhoz (pl. törlés 365 nap után, régi verziók törlése, befejezetlen többrészes feltöltések törlése). A szabályok alkalmazhatók teljes tárolókra, specifikus előtagokra vagy objektumcímkékre. Az S3 Analytics javaslatokat ad a Standard és Standard-IA közötti optimális átmeneti napokra, napi CSV jelentéseket generálva (24-48 óra után elérhető).

## S3 eseményértesítések
Az S3 események (objektum létrehozása, eltávolítása, visszaállítása, replikálása) értesítéseket válthatnak ki. Ezek az értesítések szűrhetők (pl. fájlkiterjesztés alapján, mint a .JPEG), és elküldhetők SNS témaköröknek, SQS üzenetsoroknak vagy Lambda függvényeknek. IAM engedélyek (erőforrás-hozzáférési házirendek) szükségesek ahhoz, hogy az S3 adatokat küldhessen ezekre a célhelyekre. Minden S3 esemény az Amazon EventBridge-be is elküldésre kerül, amely fejlett szűrési lehetőségeket (metaadatok, objektumméret, név) kínál, és több mint 18 AWS szolgáltatást támogat célhelyként, megbízhatóbb kézbesítést és olyan funkciókat biztosítva, mint az archiválás és az események visszajátszása.

## S3 teljesítmény
Az Amazon S3 automatikusan skálázódik a magas kérésszámokhoz (3500 PUT/COPY/POST/DELETE másodpercenként előtagonként; 5500 GET/HEAD másodpercenként előtagonként) alacsony késleltetéssel (100-200 ms az első bájtig). Az "előtag" a tárolón belüli útvonalra utal (pl. /folder1/subfolder1/). A teljesítményoptimalizálási technikák a következők:
-   **Többrészes feltöltés:** Ajánlott 100 MB-nál nagyobb fájlokhoz, kötelező 5 GB-nál nagyobb fájlokhoz. Párhuzamosítja a feltöltéseket a sávszélesség maximalizálása érdekében, és lehetővé teszi a sikertelen részek újrapróbálását. Az életciklus-házirendek törölhetik a befejezetlen többrészes feltöltéseket.
-   **S3 Transfer Acceleration:** Növeli az átviteli sebességet az adatok AWS peremhálózati helyeken keresztül történő irányításával, amelyek ezután a privát AWS hálózatot használják az S3 tároló eléréséhez a célrégióban. Kompatibilis a többrészes feltöltéssel.
-   **S3 Byte-Range Fetches:** Párhuzamosítja a GET kéréseket egy fájl specifikus bájt tartományainak lekérésével, felgyorsítva a letöltéseket és javítva a hibákkal szembeni ellenállást. Hasznos részleges adatok (pl. fájlfejlécek) lekérésére is.

## S3 kötegelt műveletek
Lehetővé teszi tömeges műveletek végrehajtását meglévő S3 objektumokon egyetlen kéréssel. Használati esetek: metaadatok módosítása, objektumok másolása, titkosítatlan objektumok titkosítása, ACL-ek/címkék módosítása, objektumok visszaállítása a Glacierből, vagy Lambda függvények meghívása objektumokon. Az S3 Batch kezeli az újrapróbálkozásokat, nyomon követi a folyamatot, értesítéseket küld és jelentéseket generál. Az S3 Inventory jelentés felhasználható az Athena-val együtt a kötegelt műveletekhez szükséges szűrt objektumlista generálására.

## S3 leltár
Listát biztosít az S3 tárolóban lévő összes objektumról és metaadatairól, jobb alternatívát kínálva az S3 list API-hoz az auditáláshoz és jelentéskészítéshez. Használati esetek: replikáció/titkosítás állapotának auditálása, objektumok számlálása, titkosítatlan objektumok azonosítása, korábbi verziók teljes tárolási méretének nyomon követése. A kimeneti fájlok CSV, ORC vagy Apache Parquet formátumban készülnek, naponta vagy hetente generálva. Az adatok lekérdezhetők Athena, Redshift, Presto, Hive és Spark segítségével. A leltárkonfigurációkhoz azonos régióban lévő cél tároló szükséges, és tartalmazhatnak további mezőket, mint a méret, utolsó módosítás, tárolási osztály, titkosítási állapot.

## S3 Glacier áttekintés
Alacsony költségű objektumtároló szolgáltatás hosszú távú archiváláshoz és biztonsági mentéshez (több tíz év), alternatívája a helyszíni mágnesszalagos tárolásnak. Magas tartósságot (11 kilences) kínál. Az elemeket "archívumoknak" (akár 40 TB) nevezik, amelyek "tárolókban" vannak tárolva. Az adatok alapértelmezetten titkosítva vannak nyugalmi állapotban AES-256-tal (AWS által kezelt kulcsok). Az életciklus-házirendek átvihetik az S3 objektumokat a Glacierbe.
-   **Tároló műveletek:** Tárolók létrehozása/törlése (csak üres állapotban), tároló metaadatainak lekérése (létrehozási dátum, archívumok száma, teljes méret), tároló leltárának letöltése.
-   **Archívum műveletek:** Feltöltés (közvetlenül vagy többrészes), letöltés (visszaállítási feladat után), törlés.
-   **Visszaállítási lehetőségek:**
    *   **Expedited:** 1-5 perc (legdrágább).
    *   **Standard:** 3-5 óra.
    *   **Bulk:** 5-12 óra (legköltséghatékonyabb).
-   **Tároló házirendek:**
    *   **Tároló hozzáférési házirend:** Hasonló az S3 tároló házirendjéhez, korlátozza a felhasználói/fiók engedélyeket.
    *   **Tároló zárolási házirend:** Változtathatatlan házirend szabályozási/megfelelőségi célokra (WORM - Write Once, Read Many). A zárolás után nem módosítható.
-   **Értesítések:** A Glacier aszinkron. A tároló értesítések üzeneteket küldhetnek SNS témaköröknek a feladat befejezésekor. Az S3 eseményértesítések nyomon követhetik az S3 Object Restore Post (kezdeményezett) és az S3 Object Restore Completed eseményeket az S3 Glacier tárolási osztályból visszaállított objektumokhoz.

## S3 Glacier gyakorlat
Bemutatja egy S3 objektum tárolási osztályának Glacierre történő módosítását. Egy Glacier objektum visszaállításának kezdeményezése visszaállítási lehetőségek (Bulk, Standard, Expedited) kiválasztását és az elérhetőségi napok megadását igényli. Az eseményértesítések konfigurálhatók a visszaállítás kezdeményezésekor és befejezésekor történő riasztásra.

## Glacier Vault Lock gyakorlat
Bemutatja egy Glacier tároló létrehozását és egy Vault Lock házirend alkalmazását. A házirend, miután kezdeményezték és befejezték, megváltoztathatatlanná válik, érvényesítve a megfelelőségi szabályokat (pl. megtiltja a 365 napnál fiatalabb archívumok törlését). A fájlok Glacier tárolókba történő feltöltéséhez általában az SDK vagy a CLI szükséges, mivel a felhasználói felület korlátozott.

## S3 többrészes feltöltés mélyrehatóan
Lehetővé teszi nagy objektumok feltöltését részekben (akár 10 000 rész) bármilyen sorrendben. Ajánlott 100 MB-nál nagyobb fájlokhoz, kötelező 5 GB-nál nagyobb fájlokhoz. Párhuzamosítja a feltöltéseket, felgyorsítja az átvitelt, és lehetővé teszi a sikertelen részek újrapróbálását. Miután az összes részt feltöltötték, egy "teljesítési kérés" összefűzi őket a végső objektummá. Az életciklus-házirendek törölhetik a befejezetlen többrészes feltöltéseket egy megadott számú nap után.

## S3 többrészes feltöltés gyakorlat
Bemutatja az AWS CLI (CloudShell) használatát többrészes feltöltés végrehajtására:
1.  S3 tároló létrehozása.
2.  Nagy fájl (pl. 100 MB) generálása `dd` paranccsal.
3.  A nagy fájl kisebb részekre osztása `split` paranccsal.
4.  Többrészes feltöltés kezdeményezése `aws s3api create-multipart-upload` paranccsal az Upload ID lekéréséhez.
5.  Minden rész feltöltése `aws s3api upload-part` paranccsal, megadva a rész számát és az Upload ID-t.
6.  A részek listázása `aws s3api list-parts` paranccsal az ellenőrzéshez.
7.  A többrészes feltöltés befejezése `aws s3api complete-multipart-upload` paranccsal, megadva az Upload ID-t és az összes rész ETag-jét.
A konzol felhasználói felülete nem mutatja az egyes részeket, amíg a feltöltés be nem fejeződik.

## Amazon Athena
Szerver nélküli lekérdezési szolgáltatás az Amazon S3-ban lévő adatok elemzésére szabványos SQL használatával. A Presto motorra épül. A felhasználók adatokat töltenek be az S3-ba, és az Athena közvetlenül lekérdezi azokat anélkül, hogy áthelyezné. Támogatja a CSV, JSON, ORC, Avro, Parquet formátumokat. Az árképzés a terabátonként beolvasott adatokon alapul. Gyakran használják az Amazon QuickSight-tal együtt jelentésekhez/irányítópultokhoz.
-   **Használati esetek:** Ad-hoc lekérdezések, BI, analitika, jelentéskészítés, AWS szolgáltatásnaplók (VPC Flow Logs, Load Balancer Logs, CloudTrail) elemzése.
-   **Teljesítményjavítások:**
    *   Oszlopos adatformátumok (Apache Parquet, ORC) használata kevesebb adat beolvasásához és a költségek csökkentéséhez.
    *   Adatok tömörítése kisebb lekérésekhez.
    *   Adatkészletek particionálása (pl. év/hónap/nap szerint az S3 útvonalon) csak a releváns adatok beolvasásához.
    *   Nagyobb fájlok (pl. >128 MB) használata a többletköltségek minimalizálása érdekében.
-   **Federated Query:** Az Athena az S3-on kívüli adatokat is lekérdezheti (relációs/nem-relációs adatbázisok, egyedi adatforrások, helyszíni) Data Source Connectorok (Lambda függvények) segítségével. Az eredmények visszatárolhatók az S3-ba.

## Amazon Athena gyakorlat
Bemutatja az Athena használatát:
1.  Lekérdezési eredmények helyének beállítása egy S3 tárolóban.
2.  Athena adatbázis létrehozása.
3.  Tábla létrehozása az Athena-ban az S3 hozzáférési naplók reprezentálására, megadva az S3 tároló helyét.
4.  A tábla előnézetének megtekintése az adatok megtekintéséhez.
5.  Haladó SQL lekérdezések futtatása (pl. kérések számlálása HTTP állapot szerint, jogosulatlan hozzáférés szűrése) az S3 adatok elemzéséhez. Az Athena egyszerűsíti az adatelemzést szerverkezelés nélkül.

## S3 Select és Glacier Select
Lehetővé teszi az adatok egy részhalmazának lekérését egy objektumból (S3 Select) vagy egy archívumból (Glacier Select) SQL lekérdezések segítségével. Javítja a teljesítményt és csökkenti az adatlekérdezési költségeket az adatok szűrésével az átvitel előtt. Hasznos nagy naplófájlokhoz (pl. "error" szót tartalmazó sorok lekérése). Működik CSV, JSON, Apache Parquet és tömörített fájlokkal (Gzip, Bzip2, Snappy).
-   **Kulcsfontosságú különbség:** Az Athena teljes tárolókat kérdez le, míg az S3/Glacier Select egyedi objektumokat/archívumokat.